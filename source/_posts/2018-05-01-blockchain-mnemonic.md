---
layout: post
title: 区块链 - 助记词
date: 2018-05-01 00:00
tags: [blockchain, mnemonic]
---


#### 什么是区块链？

区块链是分布式数据存储、点对点传输、共识机制、加密算法等计算机技术的新型应用模式。
区块链（Blockchain），是比特币的一个重要概念，它本质上是一个去中心化的数据库，同时作为比特币的底层技术，是一串使用密码学方法相关联产生的数据块，每一个数据块中包含了一批次比特币网络交易的信息，用于验证其信息的有效性（防伪）和生成下一个区块。

#### 什么是助记词？

助记词由12-24个单词组成，一个助记词可以生成无穷个私钥，可以理解为助记词是个树的根，这个根上可以长很多分支，每个叶子是一个私钥。这也是为何一个助记词可以管理HD账户下所有的钱包地址。


#### 什么是 Bip39？

将 seed 用方便记忆和书写的单字表示。一般由 12 个单字组成，称为 mnemonic code(phrase)，中文称为助记词或助记码。

#### 如何去生成助记词？

```java

import org.bitcoinj.crypto.MnemonicCode;
import org.bitcoinj.crypto.MnemonicException;

private final static MnemonicCode MNEMONIC_CODE = MnemonicCode.INSTANCE;

public static String generateMnemonic() {
    byte[] random = new byte[16];
    new SecureRandom().nextBytes(random);

    String words = null;
    try {
        words = String.join(" ", MNEMONIC_CODE.toMnemonic(random));
    } catch (Throwable throwable) {
        log.error(throwable.getMessage());
    }

    return words;
}

public enum Bip44Path {
    DEFAULT("M/44H/60H/0H/0/0"),
    ETH("M/44H/60H/0H/0/0"),
    AUTO_DEFAINE("M/44H/60H/1H/0/0"),
    BTC("M/44H/0H/0H/0/0"),
    BTC_TEST_NET("M/44H/1H/0H/0/0"),
    LEDGER("M/44H/60H/0H/0");

    private String Path;

    private Bip44Path(String path) {
        this.Path = path;
    }

    public String getPath() {
        return this.Path;
    }
}

public class WalletManager {

    public WalletFile createByMnemonic(WalletMode mode, String mnemonic, String password, String path) {
        if (Strings.isNullOrEmpty(mnemonic)) {
            throw new EthException("Mnemonic is required, cannot be null or empty", ETH_MNEMONIC_ERROR);
        }

        String bipPath = Strings.isNullOrEmpty(path) ? Bip44Path.DEFAULT.getPath() : path;
        DeterministicKey deterministicKeyFromPath = getDeterministicKeyFromPath(bipPath, mnemonic);

        return createWallet(ECKeyPair.create(deterministicKeyFromPath.getPrivKey()), password, mode);
    }

    public WalletFile createByPrivateKey(WalletMode mode, String privateKey, String password) {
        byte[] privateKeyInBytes = Numeric.hexStringToByteArray(privateKey);
        ECKeyPair ecKeyPair = ECKeyPair.create(privateKeyInBytes);
        return createWallet(ecKeyPair, password, mode);
    }

    public WalletFile createByKeyStore(String keystore, String password) {
        try {
            WalletFile walletFile = Jsonable.getInstance().readValue(keystore, WalletFile.class);
            Wallet.decrypt(password, walletFile);
            return walletFile;
        } catch (Throwable e) {
            throw new EthException("Unable to create wallet: " + e.getMessage(), KEYSTORE_PASSWORD_ERROR);
        }
    }

    private WalletFile createWallet(ECKeyPair ecKeyPair, String password, WalletMode walletMode) {
        WalletFile walletFile = null;

        try {
            if (WalletMode.STANDARD.equals(walletMode)) {
                walletFile = Wallet.createStandard(password, ecKeyPair);
            }

            if (WalletMode.LIGHT.equals(walletMode)) {
                walletFile = Wallet.createLight(password, ecKeyPair);
            }

            return walletFile;
        } catch (Throwable e) {
            log.error(e.getMessage());
            throw new EthException("Unable to create wallet: " + e.getMessage(), ETH_CREATE_WALLET_ERROR);
        }
    }

    /**
     * m/44H/60H/0H/0/0
     */
    private DeterministicKey getDeterministicKeyFromPath(String path, String mnemonic) {
        List<ChildNumber> childNumbers = HDUtils.parsePath(path);

        byte[] seed = generateSeed(mnemonic);
        DeterministicKey masterPrivateKey = HDKeyDerivation.createMasterPrivateKey(seed);

        DeterministicKey tempDeterMinisticKey = masterPrivateKey;
        for (ChildNumber childNumber : childNumbers) {
            tempDeterMinisticKey = HDKeyDerivation.deriveChildKey(tempDeterMinisticKey, childNumber);
        }

        return tempDeterMinisticKey;
    }

    private byte[] generateSeed(String mnemonic) {
        return org.web3j.crypto.MnemonicUtils.generateSeed(mnemonic, StringUtil.EMPTY);
    }
}
```
