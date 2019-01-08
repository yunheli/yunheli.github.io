---
layout: post
title: 数据库敏感信息加密在ruby中的应用
date: 2017-05-7 00:00
tags: [secure, ruby, AES, RSA]
---

## 数据库敏感信息加密研究

### 数据库敏感信息加密原因
> 鉴于之前有用户使用阿里云数据库遭到数据泄露问题，库中一部分用户的敏感信息是采用明文存储的，例如项目私钥、Git token、手机号、邮箱等

### 加密方案：
- 简单加密（对称加密）：
  对加密的信息采用对称加密，秘钥保存在本地
- 高级加密（非对称加密）：
  对敏感信息对称加密，对加密秘钥进行非对称加密，因为非对称加密对敏感信息大小有限制一般在1k以下，加密速度也和对称加密不是一个量级
  
### AES（高级加密）概念：
  進階加密标准（英语：Advanced Encryption Standard，缩写：AES），在密码学中又称Rijndael加密法，是美国联邦政府采用的一種區塊加密标准。这个标准用来替代原先的DES，已經被多方分析且廣為全世界所使用。經過五年的甄選流程，進階加密標準由美國國家標準與技術研究院（NIST）於2001年11月26日發佈於FIPS PUB 197，並在2002年5月26日成為有效的標準。2006年，進階加密标准已然成為对称密钥加密中最流行的演算法之一

### RSA（非对称加密）概念：
  RSA加密演算法是一种非对称加密演算法。在公开密钥加密和电子商业中RSA被广泛使用。RSA是1977年由罗纳德·李维斯特（Ron Rivest）、阿迪·萨莫尔（Adi Shamir）和伦纳德·阿德曼（Leonard Adleman）一起提出的。当时他们三人都在麻省理工学院工作。RSA就是他们三人姓氏开头字母拼在一起组成的。
  
### 针对两种加密方式的解决方案：
  大数据加密采用非对称加密和高级加密并行的方式，AES加密的key采用RSA非对称加密之后再对数据加密，这样保证了数据的安全
  
### 数据加密在ruby中的应用

```
# 重新复制方法，对保存到数据库的数据加密，读取的数据解密
module SensitiveWordEncrypt

  def self.prepended(base)
    base.extend ClassMethods
  end
  
  module ClassMethods
    def sensitive_words_register(*args)
      instance_eval do 
        args.each do |key|
          # 重写key方法
          send(:define_method, "#{key}") do
            @decrypted_data = super()
            DatabaseCipher.decrypt(@decrypted_data)
          end

          # 添加 key_without_decrypt 方法获取加密数据
          send(:define_method, "#{key}_without_decrypt") do
            send(key)
            @decrypted_data
          end

          # 重写 key= 方法
          send(:define_method, "#{key}=".to_sym) do |arg|
            super(DatabaseCipher.encrypt(arg))
          end
        end
      end
    end
  end
end


# 数据库加密模块 
# 采用的AES  CBC 加密算法
# RSA加密算法
class DatabaseCipher
  include Singleton 

  attr_accessor :encrypted_aes_key, :aes_iv, :public_key, :private_key

  # AES加密 CBC算法更安全
  class AES
    attr_accessor :iv, :key
    def initialize(iv, key)
      self.iv = iv
      self.key = key
    end

    def encrypt(content)
      return content if content.blank?
      cipher = OpenSSL::Cipher::AES.new(128, :CBC)
      cipher.encrypt
      cipher.key = key
      cipher.iv  = iv
      encrypted = cipher.update(content) + cipher.final
      Base64.strict_encode64(encrypted)
    end

    def decrypt(content)
      return content if content.blank?
      encrypted_string = Base64.decode64(content)
      cipher = OpenSSL::Cipher::AES.new(128, :CBC)
      cipher.decrypt
      cipher.key = key
      cipher.iv  = iv
      result = cipher.update(encrypted_string) + cipher.final      
    end
  end

  # RSA非对称加密
  class RSA
    attr_accessor :public_key, :private_key, :pub_rsa, :pri_rsa
    def initialize(public_key, private_key)
      self.public_key = public_key
      self.private_key = private_key
      self.pub_rsa = OpenSSL::PKey::RSA.new(public_key)
      self.pri_rsa = OpenSSL::PKey::RSA.new(private_key)
    end

    def encrypt(content)
      return content if content.blank?
      encrypted = pub_rsa.public_encrypt(content)
      Base64.strict_encode64(encrypted)
    end

    def decrypt(content)
      return content if content.blank?
      encrypted_string = Base64.decode64(content)
      pri_rsa.private_decrypt(encrypted_string)
    end
    class << self
    end
  end

  def configure 
    yield(self) if block_given?
    @rsa = RSA.new(public_key, private_key)
  end

  def aes 
    aes_key = decrypt_aes_key
    @aes ||= AES.new(aes_key, aes_iv)
  end

  def rsa
    @rsa ||= RSA.new(public_key, private_key)
  end

  def encrypt(content)
    return content if content.blank?
    encrypted = aes.encrypt(content)
  rescue Exception => e
    Rails.logger.info("DatabaseCipher.encrypt error: #{e}")
    content
  end

  def decrypt(content)
    return content if content.blank?
    decrypted = aes.decrypt(content)
  rescue Exception => e 
    Rails.logger.info("DatabaseCipher.decrypt error: #{e}")
    content    
  end

  def encrypt_aes_key
    @encrypted_aes_key ||= @rsa.encrypt(aes_key)
  end

  def decrypt_aes_key
    @decrypted_aes_key ||= @rsa.decrypt(encrypted_aes_key)
  end

  class << self
    def method_missing(name, *args, &block)
      instance.send(name, *args, &block)
    end
  end
end

# ActiveModel中引入加密模块
class User
  prepend SensitiveWordEncrypt
  # 要加密的数据
  sensitive_words_register(:access_token) 
end
```
