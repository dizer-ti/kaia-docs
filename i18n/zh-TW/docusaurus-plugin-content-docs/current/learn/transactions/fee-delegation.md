# 收費代表團

## TxTypeFeeDelegatedValueTransfer <a id="txtypefeedelegatedvaluetransfer"></a>

TxTypeFeeDelegatedValueTransfer 用於用戶發送 KAIA。 由於 Kaia 提供了多種交易類型，使每種交易類型只服務於一個目的，因此 TxTypeFeeDelegatedValueTransfer 只限於將 KAIA 發送到外部擁有的賬戶。 因此，TxTypeFeeDelegatedValueTransfer 只有在 `to` 是外部擁有的賬戶時才會被接受。 要將 KAIA 轉入智能合約賬戶，請使用 [TxTypeFeeDelegatedSmartContractExecution](#txtypefeedelegatedsmartcontractexecution)。 該交易類型將進行以下更改。

1. 付費者的餘額會減少交易費的金額。
2. 發送方的 nonce 增加一個。
3. `value` KAIA 由發送方傳送給接收方。

### 屬性<a id="attributes"></a>

| 屬性                 | 類型                                                                                                                                                                          | 說明                                                                                                                                                                  |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| type               | uint8\(Go\)                                                                                                                                            | TxTypeFeeDelegatedValueTransfer 的類型。 必須為 0x09。                                                                                                                      |
| nonce              | uint64 \(Go\)                                                                                                                                          | 用於唯一標識發件人交易的值。 如果一個發送方生成了兩個具有相同 nonce 的交易，則只執行其中一個。                                                                                                                 |
| gasPrice           | \*big.Int （ Go\)                                                                                                                                           | 以 `kei` 為單位的氣體單價，發件人將支付交易費。 交易費的計算公式為 `gas`\* `gasPrice`。 例如，如果交易消耗了 10 單位天然氣，而 gasPrice 為 10^18，則交易費為 10 KAIA。 參見[KAIA 單位](../kaia-native-token.md#units-of-klay)。 |
| gas                | uint64 \(Go\)                                                                                                                                          | 交易允許使用的最大燃氣量。                                                                                                                                                       |
| to                 | common.Address\(Go\)                                                                                                                   | 接收轉賬金額的賬戶地址。                                                                                                                                                        |
| value              | \*big.Int （ Go\)                                                                                                                                           | 以 `kei` 為單位的 KAIA 轉賬金額。                                                                                                                                             |
| from               | common.Address\(Go\)                                                                                                                   | 發件人地址。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |
| txSignatures       | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 發件人簽名。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |
| feePayer           | common.Address\(Go\)                                                                                                                   | 繳費人地址。                                                                                                                                                              |
| feePayerSignatures | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 繳費人簽名。                                                                                                                                                              |

### 寄件人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-sender"></a>

要對發送方進行簽名，RLP 序列化的方法如下：

```javascript
SigRLP = encode([encode([type, nonce, gasPrice, gas, to, value, from]), chainid, 0, 0])
SigHash = keccak256(SigRLP)
Signature = sign(SigHash, <the sender's private key>)
```

### 繳費人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-fee-payer"></a>

要製作繳費人簽名，RLP 序列化工作應如下進行：

```javascript
SigFeePayerRLP = encode([ encode([type, nonce, gasPrice, gas, to, value, from]), feePayer, chainid, 0, 0 ])
SigFeePayerHash = keccak256(SigFeePayerRLP)
SignatureFeePayer = sign(SigFeePayerHash, <the fee payer's private key>)
```

### SenderTxHash 的 RLP 編碼<a id="rlp-encoding-for-sendertxhash"></a>

要製作 SenderTxHash，RLP 序列化過程如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
SenderTxHashRLP = type + encode([nonce, gasPrice, gas, to, value, from, txSignatures])
SenderTxHash = keccak256(SenderTxHashRLP)
```

### 交易哈希的 RLP 編碼<a id="rlp-encoding-for-transaction-hash"></a>

要製作事務哈希值，RLP 序列化的步驟如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
feePayerSignatures (a single signature) = [[v, r, s]]
feePayerSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
TxHashRLP = type + encode([nonce, gasPrice, gas, to, value, from, txSignatures, feePayer, feePayerSignatures])`
TxHash = keccak256(TxHashRLP)
```

### RLP 編碼 （示例）<a id="rlp-encoding-example"></a>

下面顯示的是 RLP 序列化的結果和事務對象：

```javascript
ChainID 0x1
PrivateKey 0x45a915e4d060149eb4365960e6a7a45f334393093061116b197e3240065ff2d8
PublicKey.X 0x3a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33d
PublicKey.Y 0x8072e77939dc03ba44790779b7a1025baf3003f6732430e20cd9b76d953391b3
SigRLP 0xf839b5f4098204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0b018080
SigHash 0xb86e4cc0955f7c2cda1b36038c9d43a2724fc956c11e09c37625379b7eb2bd21
Signature f845f84325a09f8e49e2ad84b0732984398749956e807e4b526c786af3c5f7416b293e638956a06bf88342092f6ff9fabe31739b2ebfa1409707ce54a54693e91a6b9bb77df0e7
FeePayerPrivateKey 0xb9d5558443585bca6f225b935950e3f6e69f9da8a5809a83f51c3365dff53936
FeePayerPublicKey.X 0x327434d4cfc66ef8857d431419e9deebdc53a3e415edcc55382e2d417b8dd102
FeePayerPublicKey.Y 0x65fc97045707faf7b8f81ac65089d4cc71f69ad0bf1bc8559bc24f13fc284ced
SigRLPFeePayer 0xf84eb5f4098204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0b945a0043070275d9f6054307ee7348bd660849d90f018080
SigHashFeePayer 0x3e7c5f40e826d1d22493be59bf62928dc397de5c972bd9bfa3fe5206c24a5f82
SignatureFeePayer f845f84326a0f45cf8d7f88c08e6b6ec0b3b562f34ca94283e4689021987abb6b0772ddfd80aa0298fe2c5aeabb6a518f4cbb5ff39631a5d88be505d3923374f65fdcf63c2955b
TxHashRLP 0x09f8d68204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0bf845f84325a09f8e49e2ad84b0732984398749956e807e4b526c786af3c5f7416b293e638956a06bf88342092f6ff9fabe31739b2ebfa1409707ce54a54693e91a6b9bb77df0e7945a0043070275d9f6054307ee7348bd660849d90ff845f84326a0f45cf8d7f88c08e6b6ec0b3b562f34ca94283e4689021987abb6b0772ddfd80aa0298fe2c5aeabb6a518f4cbb5ff39631a5d88be505d3923374f65fdcf63c2955b
TxHash e1e07f9971153499fc8c7bafcdaf7abc20b37aa4c18fb1e53a9bfcc259e3644c
SenderTxHashRLP 0x09f87a8204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0bf845f84325a09f8e49e2ad84b0732984398749956e807e4b526c786af3c5f7416b293e638956a06bf88342092f6ff9fabe31739b2ebfa1409707ce54a54693e91a6b9bb77df0e7
SenderTxHash 40f8c94e01e07eb5353f6cd4cd3eabd5893215dd53a50ba4b8ff9a447ac51731

    TX(e1e07f9971153499fc8c7bafcdaf7abc20b37aa4c18fb1e53a9bfcc259e3644c)
    Type:          TxTypeFeeDelegatedValueTransfer
    From:          0xa94f5374Fce5edBC8E2a8697C15331677e6EbF0B
    To:            0x7b65B75d204aBed71587c9E519a89277766EE1d0
    Nonce:         1234
    GasPrice:      0x19
    GasLimit:      0xf4240
    Value:         0xa
    Signature:     [{"V":"0x25","R":"0x9f8e49e2ad84b0732984398749956e807e4b526c786af3c5f7416b293e638956","S":"0x6bf88342092f6ff9fabe31739b2ebfa1409707ce54a54693e91a6b9bb77df0e7"}]
    FeePayer:      0x5A0043070275d9f6054307Ee7348bD660849D90f
    FeePayerSig:   [{"V":"0x26","R":"0xf45cf8d7f88c08e6b6ec0b3b562f34ca94283e4689021987abb6b0772ddfd80a","S":"0x298fe2c5aeabb6a518f4cbb5ff39631a5d88be505d3923374f65fdcf63c2955b"}]
    Hex:           09f8d68204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0bf845f84325a09f8e49e2ad84b0732984398749956e807e4b526c786af3c5f7416b293e638956a06bf88342092f6ff9fabe31739b2ebfa1409707ce54a54693e91a6b9bb77df0e7945a0043070275d9f6054307ee7348bd660849d90ff845f84326a0f45cf8d7f88c08e6b6ec0b3b562f34ca94283e4689021987abb6b0772ddfd80aa0298fe2c5aeabb6a518f4cbb5ff39631a5d88be505d3923374f65fdcf63c2955b
```

### RPC 輸出 （示例）<a id="rpc-output-example"></a>

下面顯示的是通過 JSON RPC 返回的事務對象。

```javascript
{
  "blockHash": "0x7ad6ed1f9955be00db8fb5452125f0e9a3c0856abb5b4cc4aed91ffc134321da",
  "blockNumber": "0x1",
  "contractAddress": null,
  "feePayer": "0x029fdce0457db02f05c4be9f67b7115cb8ea15ca",
  "feePayerSignatures": [
    {
      "V": "0x26",
      "R": "0x984e9d43c496ef39ef2d496c8e1aee695f871e4f6cfae7f205ddda1589ca5c9e",
      "S": "0x46647d1ce8755cd664f5fb4eba3082dd1a13817488029f3869662986b7b1a5ae"
    }
  ],
  "from": "0x0fcda0f2efbe1b4e61b487701ce4f2f8abc3723d",
  "gas": "0x174876e800",
  "gasPrice": "0x5d21dba00",
  "gasUsed": "0x7918",
  "logs": [],
  "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "nonce": "0x2",
  "senderTxHash": "0x6a8cf9a2f6d16561303445309d4f210c8be862f0d0c0e6f4998775fef9b4f957",
  "signatures": [
    {
      "V": "0x25",
      "R": "0x368b3324b37831b51711a2eba2a7608438a2bd5956ccecbcdb07d9163ff8bc87",
      "S": "0x7ee2e86ad6f01c867b2ced9d69e614ba22e539726451400fccdd56acbbc7a6f7"
    }
  ],
  "status": "0x1",
  "to": "0x75c3098be5e4b63fbac05838daaee378dd48098d",
  "transactionHash": "0xea4341b5c95fd5a0c3a8a15a4177ab6394725c24f722a9e31f53474a6dcf086a",
  "transactionIndex": "0x2",
  "type": "TxTypeFeeDelegatedValueTransfer",
  "typeInt": 9,
  "value": "0x21e19e0c9bab2400000"
}
```

## TxTypeFeeDelegatedValueTransferMemo<a id="txtypefeedelegatedvaluetransfermemo"></a>

TxTypeFeeDelegatedValueTransferMemo 用於用戶發送帶有特定信息的 KAIA。 TxTypeFeeDelegatedValueTransferMemo 僅在 `to` 為外部所有賬戶時才被接受。 要將 KAIA 轉入智能合約賬戶，請使用 [TxTypeFeeDelegatedSmartContractExecution](#txtypefeedelegatedsmartcontractexecution)。 該交易類型將進行以下更改。

1. 付費者的餘額會減少交易費的金額。
2. 發送方的 nonce 增加一個。
3. `value` KAIA 由發送方傳送給接收方。

### 屬性<a id="attributes"></a>

| 屬性                 | 說明                                                                                                                                                                          | 類型                                                                                                                                                                  | 示例值 |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :-- |
| type               | uint8\(Go\)                                                                                                                                            | TxTypeFeeDelegatedValueTransferMemo 的類型。 必須為 0x11。                                                                                                                  |     |
| nonce              | uint64 \(Go\)                                                                                                                                          | 用於唯一標識發件人交易的值。 如果一個發送方生成了兩個具有相同 nonce 的交易，則只執行其中一個。                                                                                                                 |     |
| gasPrice           | \*big.Int （ Go\)                                                                                                                                           | 以 `kei` 為單位的氣體單價，發件人將支付交易費。 交易費的計算公式為 `gas`\* `gasPrice`。 例如，如果交易消耗了 10 單位天然氣，而 gasPrice 為 10^18，則交易費為 10 KAIA。 參見[KAIA 單位](../kaia-native-token.md#units-of-klay)。 |     |
| gas                | uint64 \(Go\)                                                                                                                                          | 交易允許使用的最大燃氣量。                                                                                                                                                       |     |
| to                 | common.Address\(Go\)                                                                                                                   | 接收轉賬金額的賬戶地址。                                                                                                                                                        |     |
| value              | \*big.Int （ Go\)                                                                                                                                           | 以 `kei` 為單位的 KAIA 轉賬金額。                                                                                                                                             |     |
| from               | common.Address\(Go\)                                                                                                                   | 發件人地址。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |     |
| input              | \[\]byte \(Go\)                                                                                  | 交易附帶的數據。 信息應傳遞給該屬性。                                                                                                                                                 |     |
| txSignatures       | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 發件人簽名。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |     |
| feePayer           | common.Address\(Go\)                                                                                                                   | 繳費人地址。                                                                                                                                                              |     |
| feePayerSignatures | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 繳費人簽名。                                                                                                                                                              |     |

### 寄件人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-sender"></a>

要對發送方進行簽名，RLP 序列化的方法如下：

```javascript
SigRLP = encode([encode([type, nonce, gasPrice, gas, to, value, from, input]), chainid, 0, 0])
SigHash = keccak256(SigRLP)
Signature = sign(SigHash, <the sender's private key>)
```

### 繳費人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-fee-payer"></a>

要製作繳費人簽名，RLP 序列化工作應如下進行：

```javascript
SigFeePayerRLP = encode([encode([type, nonce, gasPrice, gas, to, value, from, input]), feePayer, chainid, 0, 0])
SigFeePayerHash = keccak256(SigFeePayerRLP)
SignatureFeePayer = sign(SigFeePayerHash, <the fee payer's private key>)
```

### SenderTxHash 的 RLP 編碼<a id="rlp-encoding-for-sendertxhash"></a>

要製作 SenderTxHash，RLP 序列化過程如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
SenderTxHashRLP = type + encode([nonce, gasPrice, gas, to, value, from, input, txSignatures])
SenderTxHash = keccak256(SenderTxHashRLP)
```

### 交易哈希的 RLP 編碼<a id="rlp-encoding-for-transaction-hash"></a>

要製作事務哈希值，RLP 序列化的步驟如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
feePayerSignatures (a single signature) = [[v, r, s]]
feePayerSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
TxHashRLP = type + encode([nonce, gasPrice, gas, to, value, from, input, txSignatures, feePayer, feePayerSignatures])
TxHash = keccak256(TxHashRLP)
```

### RLP 編碼 （示例）<a id="rlp-encoding-example"></a>

下面顯示的是 RLP 序列化的結果和事務對象：

```javascript
ChainID 0x1
PrivateKey 0x45a915e4d060149eb4365960e6a7a45f334393093061116b197e3240065ff2d8
PublicKey.X 0x3a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33d
PublicKey.Y 0x8072e77939dc03ba44790779b7a1025baf3003f6732430e20cd9b76d953391b3
SigRLP 0xf841b83cf83a118204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0b8568656c6c6f018080
SigHash 0x3333b9336d431ffa53b795fedcf03cc2217cea3f26825ea5cbf7d69f0b99fde9
Signature f845f84326a064e213aef0167fbd853f8f9989ef5d8b912a77457395ccf13d7f37009edd5c5ba05d0c2e55e4d8734fe2516ed56ac628b74c0eb02aa3b6eda51e1e25a1396093e1
FeePayerPrivateKey 0xb9d5558443585bca6f225b935950e3f6e69f9da8a5809a83f51c3365dff53936
FeePayerPublicKey.X 0x327434d4cfc66ef8857d431419e9deebdc53a3e415edcc55382e2d417b8dd102
FeePayerPublicKey.Y 0x65fc97045707faf7b8f81ac65089d4cc71f69ad0bf1bc8559bc24f13fc284ced
SigRLPFeePayer 0xf856b83cf83a118204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0b8568656c6c6f945a0043070275d9f6054307ee7348bd660849d90f018080
SigHashFeePayer 0xed015096fb27764f576415e23576228cbf7c4fdad464ea7ffc3a1856dfe391c9
SignatureFeePayer f845f84326a087390ac14d3c34440b6ddb7b190d3ebde1a07d9a556e5a82ce7e501f24a060f9a037badbcb12cda1ed67b12b1831683a08a3adadee2ea760a07a46bdbb856fea44
TxHashRLP 0x11f8dc8204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0b8568656c6c6ff845f84326a064e213aef0167fbd853f8f9989ef5d8b912a77457395ccf13d7f37009edd5c5ba05d0c2e55e4d8734fe2516ed56ac628b74c0eb02aa3b6eda51e1e25a1396093e1945a0043070275d9f6054307ee7348bd660849d90ff845f84326a087390ac14d3c34440b6ddb7b190d3ebde1a07d9a556e5a82ce7e501f24a060f9a037badbcb12cda1ed67b12b1831683a08a3adadee2ea760a07a46bdbb856fea44
TxHash 8f68882f6192a53ba470aeca1e83ed9b9e519906a91256724b284dee778b21c9
SenderTxHashRLP 0x11f8808204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0b8568656c6c6ff845f84326a064e213aef0167fbd853f8f9989ef5d8b912a77457395ccf13d7f37009edd5c5ba05d0c2e55e4d8734fe2516ed56ac628b74c0eb02aa3b6eda51e1e25a1396093e1
SenderTxHash fffaa2b38d4e684ea70a89c78fc7b2659000d130c76ad721d68175cbfc77c550

    TX(8f68882f6192a53ba470aeca1e83ed9b9e519906a91256724b284dee778b21c9)
    Type:          TxTypeFeeDelegatedValueTransferMemo
    From:          0xa94f5374Fce5edBC8E2a8697C15331677e6EbF0B
    To:            0x7b65B75d204aBed71587c9E519a89277766EE1d0
    Nonce:         1234
    GasPrice:      0x19
    GasLimit:      0xf4240
    Value:         0xa
    Signature:     [{"V":"0x26","R":"0x64e213aef0167fbd853f8f9989ef5d8b912a77457395ccf13d7f37009edd5c5b","S":"0x5d0c2e55e4d8734fe2516ed56ac628b74c0eb02aa3b6eda51e1e25a1396093e1"}]
    FeePayer:      0x5A0043070275d9f6054307Ee7348bD660849D90f
    FeePayerSig:   [{"V":"0x26","R":"0x87390ac14d3c34440b6ddb7b190d3ebde1a07d9a556e5a82ce7e501f24a060f9","S":"0x37badbcb12cda1ed67b12b1831683a08a3adadee2ea760a07a46bdbb856fea44"}]
    Data:          36383635366336633666
    Hex:           11f8dc8204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0b8568656c6c6ff845f84326a064e213aef0167fbd853f8f9989ef5d8b912a77457395ccf13d7f37009edd5c5ba05d0c2e55e4d8734fe2516ed56ac628b74c0eb02aa3b6eda51e1e25a1396093e1945a0043070275d9f6054307ee7348bd660849d90ff845f84326a087390ac14d3c34440b6ddb7b190d3ebde1a07d9a556e5a82ce7e501f24a060f9a037badbcb12cda1ed67b12b1831683a08a3adadee2ea760a07a46bdbb856fea44
```

### RPC 輸出 （示例）<a id="rpc-output-example"></a>

下面顯示的是通過 JSON RPC 返回的事務對象。

```javascript
{
  "blockHash": "0x7ad6ed1f9955be00db8fb5452125f0e9a3c0856abb5b4cc4aed91ffc134321da",
  "blockNumber": "0x1",
  "contractAddress": null,
  "feePayer": "0x029fdce0457db02f05c4be9f67b7115cb8ea15ca",
  "feePayerSignatures": [
    {
      "V": "0x25",
      "R": "0xb5d80dc924c51f58eb674a142ebfd8ca1c0bc722bc85b001a5a6905ba8226b1",
      "S": "0x79852418faacd4407aee4a461a08602fcf6a3a3cb63b9ba69d70ffe2f5fe3cd"
    }
  ],
  "from": "0x0fcda0f2efbe1b4e61b487701ce4f2f8abc3723d",
  "gas": "0x174876e800",
  "gasPrice": "0x5d21dba00",
  "gasUsed": "0x7b0c",
  "input": "0x68656c6c6f",
  "logs": [],
  "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "nonce": "0x5",
  "senderTxHash": "0x5a4e42bac0b2bc8dda4ee82bfafc83e7f156f74d81d367a3db430abd40b2cd47",
  "signatures": [
    {
      "V": "0x26",
      "R": "0xe8f5484b057b542c80f16c5bb8707e040619c3dc9ac5628d2797aa3d8a2fc0d0",
      "S": "0x5d598f2f10283ded6f6e6a216f4278b27fdf4d431272fa090064ac0fd3fc8102"
    }
  ],
  "status": "0x1",
  "to": "0x75c3098be5e4b63fbac05838daaee378dd48098d",
  "transactionHash": "0x66fe4d1abdf15a250f9391646e0242c8e4c3310250ca316d8fd00856aac16172",
  "transactionIndex": "0x5",
  "type": "TxTypeFeeDelegatedValueTransferMemo",
  "typeInt": 17,
  "value": "0x989680"
}
```

## TxTypeFeeDelegatedSmartContractDeploy<a id="txtypefeedelegatedsmartcontractdeploy"></a>

TxTypeFeeDelegatedSmartContractDeploy 部署具有費用委託功能的智能合約。 該交易類型將進行以下更改。

1. 付費者的餘額會減少交易費的金額。
2. 發送方的 nonce 增加一個。
3. 智能合約與 `input` 中的代碼一起部署。 部署地址將通過收據中的 `contractAddress` 返回。
4. `value` KAIA 由發送方傳送給接收方。

### 屬性<a id="attributes"></a>

| 屬性                 | 類型                                                                                                                                                                          | 說明                                                                                                                                                                  |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| type               | uint8\(Go\)                                                                                                                                            | TxTypeFeeDelegatedSmartContractDeploy 的類型。 必須為 0x29。                                                                                                                |
| nonce              | uint64 \(Go\)                                                                                                                                          | 用於唯一標識發件人交易的值。 如果一個發送方生成了兩個具有相同 nonce 的交易，則只執行其中一個。                                                                                                                 |
| gasPrice           | \*big.Int （ Go\)                                                                                                                                           | 以 `kei` 為單位的氣體單價，發件人將支付交易費。 交易費的計算公式為 `gas`\* `gasPrice`。 例如，如果交易消耗了 10 單位天然氣，而 gasPrice 為 10^18，則交易費為 10 KAIA。 參見[KAIA 單位](../kaia-native-token.md#units-of-klay)。 |
| gas                | uint64 \(Go\)                                                                                                                                          | 交易允許使用的最大燃氣量。                                                                                                                                                       |
| to                 | \*common.Address（Go\）                                                                                                                                       | 接收轉賬金額的賬戶地址。 目前，該值必須為零。 今後將支持指定地址。                                                                                                                                  |
| value              | \*big.Int （ Go\)                                                                                                                                           | 以 `kei` 為單位的 KAIA 轉賬金額。                                                                                                                                             |
| from               | common.Address\(Go\)                                                                                                                   | 發件人地址。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |
| input              | \[\]byte \(Go\)                                                                                  | 附屬於事務的數據，用於執行事務。                                                                                                                                                    |
| humanReadable      | bool \(Go\)                                                                                                                                            | 必須為假，因為目前還不支持人類可讀地址。 如果為 "true"，交易將被拒絕。                                                                                                                             |
| codeFormat         | uint8\(Go\)                                                                                                                                            | 智能合約代碼的代碼格式。 目前僅支持 EVM\(0x00\) 值。                                                                                                              |
| txSignatures       | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 發件人簽名。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |
| feePayer           | common.Address\(Go\)                                                                                                                   | 繳費人地址。                                                                                                                                                              |
| feePayerSignatures | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 繳費人簽名。                                                                                                                                                              |

### 寄件人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-sender"></a>

要對發送方進行簽名，RLP 序列化的方法如下：

```javascript
SigRLP = encode([encode([type, nonce, gasPrice, gas, to, value, from, input, humanReadable, codeFormat]), chainid, 0, 0])
SigHash = keccak256(SigRLP)
Signature = sign(SigHash, <the sender's private key>)
```

### 繳費人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-fee-payer"></a>

要製作繳費人簽名，RLP 序列化工作應如下進行：

```javascript
SigFeePayerRLP = encode([encode([type, nonce, gasPrice, gas, to, value, from, input, humanReadable, codeFormat]), feePayer, chainid, 0, 0])
SigFeePayerHash = keccak256(SigFeePayerRLP)
SignatureFeePayer = sign(SigFeePayerHash, <the fee payer's private key>)
```

### SenderTxHash 的 RLP 編碼<a id="rlp-encoding-for-sendertxhash"></a>

要製作 SenderTxHash，RLP 序列化過程如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
SenderTxHashRLP = type + encode([nonce, gasPrice, gas, to, value, from, input,humanReadable, codeFormat, txSignatures])
SenderTxHash = keccak256(SenderTxHashRLP)
```

### 交易哈希的 RLP 編碼<a id="rlp-encoding-for-transaction-hash"></a>

要製作事務哈希值，RLP 序列化的步驟如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
feePayerSignatures (a single signature) = [[v, r, s]]
feePayerSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
TxHashRLP = type + encode([nonce, gasPrice, gas, to, value, from, input, humanReadable, codeFormat, txSignatures, feePayer, feePayerSignatures])
TxHash = keccak256(TxHashRLP)
```

### RLP 編碼 （示例）<a id="rlp-encoding-example"></a>

下面顯示的是 RLP 序列化的結果和事務對象：

```javascript
ChainID 0x1
PrivateKey 0x45a915e4d060149eb4365960e6a7a45f334393093061116b197e3240065ff2d8
PublicKey.X 0x3a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33d
PublicKey.Y 0x8072e77939dc03ba44790779b7a1025baf3003f6732430e20cd9b76d953391b3
SigRLP 0xf90240b9023af90237298204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0bb901fe608060405234801561001057600080fd5b506101de806100206000396000f3006080604052600436106100615763ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416631a39d8ef81146100805780636353586b146100a757806370a08231146100ca578063fd6b7ef8146100f8575b3360009081526001602052604081208054349081019091558154019055005b34801561008c57600080fd5b5061009561010d565b60408051918252519081900360200190f35b6100c873ffffffffffffffffffffffffffffffffffffffff60043516610113565b005b3480156100d657600080fd5b5061009573ffffffffffffffffffffffffffffffffffffffff60043516610147565b34801561010457600080fd5b506100c8610159565b60005481565b73ffffffffffffffffffffffffffffffffffffffff1660009081526001602052604081208054349081019091558154019055565b60016020526000908152604090205481565b336000908152600160205260408120805490829055908111156101af57604051339082156108fc029083906000818181858888f193505050501561019c576101af565b3360009081526001602052604090208190555b505600a165627a7a72305820627ca46bb09478a015762806cc00c431230501118c7c26c30ac58c4e09e51c4f00290180018080
SigHash 0xfd5e0726c763d117d07e5e688889ab7e4d0d1164d1bbca26a9d4ee629cbd875b
Signature f845f84325a04ea37b8ecfed93795a9f99b1e4d554df6fb05a361965a7655abd4e4c4422a9e5a00b05e3fffe5a3c0892eaff31466f6c47b7edad80703d395d65bbfc1a2c6a2570
FeePayerPrivateKey 0xb9d5558443585bca6f225b935950e3f6e69f9da8a5809a83f51c3365dff53936
FeePayerPublicKey.X 0x327434d4cfc66ef8857d431419e9deebdc53a3e415edcc55382e2d417b8dd102
FeePayerPublicKey.Y 0x65fc97045707faf7b8f81ac65089d4cc71f69ad0bf1bc8559bc24f13fc284ced
SigRLPFeePayer 0xf90255b9023af90237298204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0bb901fe608060405234801561001057600080fd5b506101de806100206000396000f3006080604052600436106100615763ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416631a39d8ef81146100805780636353586b146100a757806370a08231146100ca578063fd6b7ef8146100f8575b3360009081526001602052604081208054349081019091558154019055005b34801561008c57600080fd5b5061009561010d565b60408051918252519081900360200190f35b6100c873ffffffffffffffffffffffffffffffffffffffff60043516610113565b005b3480156100d657600080fd5b5061009573ffffffffffffffffffffffffffffffffffffffff60043516610147565b34801561010457600080fd5b506100c8610159565b60005481565b73ffffffffffffffffffffffffffffffffffffffff1660009081526001602052604081208054349081019091558154019055565b60016020526000908152604090205481565b336000908152600160205260408120805490829055908111156101af57604051339082156108fc029083906000818181858888f193505050501561019c576101af565b3360009081526001602052604090208190555b505600a165627a7a72305820627ca46bb09478a015762806cc00c431230501118c7c26c30ac58c4e09e51c4f00290180945a0043070275d9f6054307ee7348bd660849d90f018080
SigHashFeePayer 0xb7ea4d9d8c4d20ac6fd6cfffcaf89ae7d217d7450820b3b40d9ea29a0f01a1b2
SignatureFeePayer f845f84326a0c6738376304dfb32c77649bddd4ade925b947876cfe6b1fd2c06a2e4394504cca023817ba66a6b7c92fcf23f2d5506ea2a673aae5f1a1e4d742367971ae58a1576
TxHashRLP 0x29f902d98204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0bb901fe608060405234801561001057600080fd5b506101de806100206000396000f3006080604052600436106100615763ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416631a39d8ef81146100805780636353586b146100a757806370a08231146100ca578063fd6b7ef8146100f8575b3360009081526001602052604081208054349081019091558154019055005b34801561008c57600080fd5b5061009561010d565b60408051918252519081900360200190f35b6100c873ffffffffffffffffffffffffffffffffffffffff60043516610113565b005b3480156100d657600080fd5b5061009573ffffffffffffffffffffffffffffffffffffffff60043516610147565b34801561010457600080fd5b506100c8610159565b60005481565b73ffffffffffffffffffffffffffffffffffffffff1660009081526001602052604081208054349081019091558154019055565b60016020526000908152604090205481565b336000908152600160205260408120805490829055908111156101af57604051339082156108fc029083906000818181858888f193505050501561019c576101af565b3360009081526001602052604090208190555b505600a165627a7a72305820627ca46bb09478a015762806cc00c431230501118c7c26c30ac58c4e09e51c4f00290180f845f84325a04ea37b8ecfed93795a9f99b1e4d554df6fb05a361965a7655abd4e4c4422a9e5a00b05e3fffe5a3c0892eaff31466f6c47b7edad80703d395d65bbfc1a2c6a2570945a0043070275d9f6054307ee7348bd660849d90ff845f84326a0c6738376304dfb32c77649bddd4ade925b947876cfe6b1fd2c06a2e4394504cca023817ba66a6b7c92fcf23f2d5506ea2a673aae5f1a1e4d742367971ae58a1576
TxHash a457cc54b5cfd35eb61baa5ad61398fdcecab4c83693815addf00ca7166cb87e
SenderTxHashRLP 0x29f9027d8204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0bb901fe608060405234801561001057600080fd5b506101de806100206000396000f3006080604052600436106100615763ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416631a39d8ef81146100805780636353586b146100a757806370a08231146100ca578063fd6b7ef8146100f8575b3360009081526001602052604081208054349081019091558154019055005b34801561008c57600080fd5b5061009561010d565b60408051918252519081900360200190f35b6100c873ffffffffffffffffffffffffffffffffffffffff60043516610113565b005b3480156100d657600080fd5b5061009573ffffffffffffffffffffffffffffffffffffffff60043516610147565b34801561010457600080fd5b506100c8610159565b60005481565b73ffffffffffffffffffffffffffffffffffffffff1660009081526001602052604081208054349081019091558154019055565b60016020526000908152604090205481565b336000908152600160205260408120805490829055908111156101af57604051339082156108fc029083906000818181858888f193505050501561019c576101af565b3360009081526001602052604090208190555b505600a165627a7a72305820627ca46bb09478a015762806cc00c431230501118c7c26c30ac58c4e09e51c4f00290180f845f84325a04ea37b8ecfed93795a9f99b1e4d554df6fb05a361965a7655abd4e4c4422a9e5a00b05e3fffe5a3c0892eaff31466f6c47b7edad80703d395d65bbfc1a2c6a2570
SenderTxHash f3bca26fc8b50bfbcc1e94bc792ee6489cff14056e7e9aa2b074abb385f2139f

    TX(a457cc54b5cfd35eb61baa5ad61398fdcecab4c83693815addf00ca7166cb87e)
    Type:          TxTypeFeeDelegatedSmartContractDeploy
    From:          0xa94f5374Fce5edBC8E2a8697C15331677e6EbF0B
    To:            0x7b65B75d204aBed71587c9E519a89277766EE1d0
    Nonce:         1234
    GasPrice:      0x19
    GasLimit:      0xf4240
    Value:         0xa
    Data:          363038303630343035323334383031353631303031303537363030303830666435623530363130316465383036313030323036303030333936303030663330303630383036303430353236303034333631303631303036313537363366666666666666663763303130303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303630303033353034313636333161333964386566383131343631303038303537383036333633353335383662313436313030613735373830363337306130383233313134363130306361353738303633666436623765663831343631303066383537356233333630303039303831353236303031363032303532363034303831323038303534333439303831303139303931353538313534303139303535303035623334383031353631303038633537363030303830666435623530363130303935363130313064353635623630343038303531393138323532353139303831393030333630323030313930663335623631303063383733666666666666666666666666666666666666666666666666666666666666666666666666666666663630303433353136363130313133353635623030356233343830313536313030643635373630303038306664356235303631303039353733666666666666666666666666666666666666666666666666666666666666666666666666666666663630303433353136363130313437353635623334383031353631303130343537363030303830666435623530363130306338363130313539353635623630303035343831353635623733666666666666666666666666666666666666666666666666666666666666666666666666666666663136363030303930383135323630303136303230353236303430383132303830353433343930383130313930393135353831353430313930353535363562363030313630323035323630303039303831353236303430393032303534383135363562333336303030393038313532363030313630323035323630343038313230383035343930383239303535393038313131313536313031616635373630343035313333393038323135363130386663303239303833393036303030383138313831383538383838663139333530353035303530313536313031396335373631303161663536356233333630303039303831353236303031363032303532363034303930323038313930353535623530353630306131363536323761376137323330353832303632376361343662623039343738613031353736323830366363303063343331323330353031313138633763323663333061633538633465303965353163346630303239
    HumanReadable: true
    CodeFormat:    CodeFormatEVM
    Signature:     [{"V":"0x25","R":"0x4ea37b8ecfed93795a9f99b1e4d554df6fb05a361965a7655abd4e4c4422a9e5","S":"0xb05e3fffe5a3c0892eaff31466f6c47b7edad80703d395d65bbfc1a2c6a2570"}]
    FeePayer:      0x5A0043070275d9f6054307Ee7348bD660849D90f
    FeePayerSig:   [{"V":"0x26","R":"0xc6738376304dfb32c77649bddd4ade925b947876cfe6b1fd2c06a2e4394504cc","S":"0x23817ba66a6b7c92fcf23f2d5506ea2a673aae5f1a1e4d742367971ae58a1576"}]
    Hex:           29f902d98204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0bb901fe608060405234801561001057600080fd5b506101de806100206000396000f3006080604052600436106100615763ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416631a39d8ef81146100805780636353586b146100a757806370a08231146100ca578063fd6b7ef8146100f8575b3360009081526001602052604081208054349081019091558154019055005b34801561008c57600080fd5b5061009561010d565b60408051918252519081900360200190f35b6100c873ffffffffffffffffffffffffffffffffffffffff60043516610113565b005b3480156100d657600080fd5b5061009573ffffffffffffffffffffffffffffffffffffffff60043516610147565b34801561010457600080fd5b506100c8610159565b60005481565b73ffffffffffffffffffffffffffffffffffffffff1660009081526001602052604081208054349081019091558154019055565b60016020526000908152604090205481565b336000908152600160205260408120805490829055908111156101af57604051339082156108fc029083906000818181858888f193505050501561019c576101af565b3360009081526001602052604090208190555b505600a165627a7a72305820627ca46bb09478a015762806cc00c431230501118c7c26c30ac58c4e09e51c4f00290180f845f84325a04ea37b8ecfed93795a9f99b1e4d554df6fb05a361965a7655abd4e4c4422a9e5a00b05e3fffe5a3c0892eaff31466f6c47b7edad80703d395d65bbfc1a2c6a2570945a0043070275d9f6054307ee7348bd660849d90ff845f84326a0c6738376304dfb32c77649bddd4ade925b947876cfe6b1fd2c06a2e4394504cca023817ba66a6b7c92fcf23f2d5506ea2a673aae5f1a1e4d742367971ae58a1576
```

### RPC 輸出 （示例）<a id="rpc-output-example"></a>

下面顯示的是通過 JSON RPC 返回的事務對象。

```javascript
{
  "blockHash": "0x82983fe294d286e76486760e6904369285554e1744af16786c2393a956fb4ec4",
  "blockNumber": "0x2",
  "codeFormat": "0x0",
  "contractAddress": "0x636f6e7472616374322e6b6c6179746e00000000",
  "feePayer": "0x029fdce0457db02f05c4be9f67b7115cb8ea15ca",
  "feePayerSignatures": [
    {
      "V": "0x25",
      "R": "0x614fd887f4702627156132c9d56584207d1eaff529ee2967431eeaba924678f9",
      "S": "0x6b883a4467ca95a0ee75567062cb6d35629e9a22faeb8a711896488ce2cc4ed9"
    }
  ],
  "from": "0x0fcda0f2efbe1b4e61b487701ce4f2f8abc3723d",
  "gas": "0x174876e800",
  "gasPrice": "0x0",
  "gasUsed": "0xee6e5b4d",
  "humanReadable": true,
  "input": "0x608060405234801561001057600080fd5b506101de806100206000396000f3006080604052600436106100615763ffffffff7c01000000000000000000000000000000000000000000000000000000006000350416631a39d8ef81146100805780636353586b146100a757806370a08231146100ca578063fd6b7ef8146100f8575b3360009081526001602052604081208054349081019091558154019055005b34801561008c57600080fd5b5061009561010d565b60408051918252519081900360200190f35b6100c873ffffffffffffffffffffffffffffffffffffffff60043516610113565b005b3480156100d657600080fd5b5061009573ffffffffffffffffffffffffffffffffffffffff60043516610147565b34801561010457600080fd5b506100c8610159565b60005481565b73ffffffffffffffffffffffffffffffffffffffff1660009081526001602052604081208054349081019091558154019055565b60016020526000908152604090205481565b336000908152600160205260408120805490829055908111156101af57604051339082156108fc029083906000818181858888f193505050501561019c576101af565b3360009081526001602052604090208190555b505600a165627a7a72305820627ca46bb09478a015762806cc00c431230501118c7c26c30ac58c4e09e51c4f0029",
  "logs": [],
  "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "nonce": "0xb",
  "senderTxHash": "0xf8f83c7a4a334430f403b20d84db492fac43ebabbd9676d731e11460d01a2160",
  "signatures": [
    {
      "V": "0x26",
      "R": "0xf3c521fea307b39bfa914b4835112bad18f89a627d639ddabe70c20af99d29a5",
      "S": "0x5179048cf993049b380f8cf7017c6e83b23da7883d2728208fe6161808594f44"
    }
  ],
  "status": "0x1",
  "to": "0x636f6e7472616374322e6b6c6179746e00000000",
  "transactionHash": "0x39b8a31f0c02a951615e3497d68a6534b8c8cc565e514ceafec53ee7ff50b8d9",
  "transactionIndex": "0x4",
  "type": "TxTypeFeeDelegatedSmartContractDeploy",
  "typeInt": 41,
  "value": "0x0"
}
```

## TxTypeFeeDelegatedSmartContractExecution<a id="txtypefeedelegatedsmartcontractexecution"></a>

TxTypeFeeDelegatedSmartContractExecution 使用 "輸入 "中的給定數據執行智能合約。 費用由指定的繳費人支付。 只有當 `to` 是智能合約賬戶時，才接受 TxTypeFeeDelegatedSmartContractExecution。 要將 KAIA 轉移到外部所有的賬戶，請使用 [TxTypeFeeDelegatedValueTransfer](#txtypefeedelegatedvaluetransfer)。 該交易類型將進行以下更改。

1. 如果 `to` 是智能合約賬戶，則根據 `input` 執行代碼。 否則，此交易將被拒絕。
2. 付費者的餘額會減少交易費的金額。
3. 發送方的 nonce 增加一個。
4. 如果提供了 "value"，"value "KAIA 將從發送方轉移到 "to "智能合約。 合同應具有接收 KAIA 的可支付後備功能。

### 屬性<a id="attributes"></a>

| 屬性                 | 類型                                                                                                                                                                          | 說明                                                                                                                                                                  |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| type               | uint8\(Go\)                                                                                                                                            | TxTypeFeeDelegatedSmartContractExecution 的類型。 必須為 0x31。                                                                                                             |
| nonce              | uint64 \(Go\)                                                                                                                                          | 用於唯一標識發件人交易的值。 如果一個發送方生成了兩個具有相同 nonce 的交易，則只執行其中一個。                                                                                                                 |
| gasPrice           | \*big.Int （ Go\)                                                                                                                                           | 以 `kei` 為單位的氣體單價，發件人將支付交易費。 交易費的計算公式為 `gas`\* `gasPrice`。 例如，如果交易消耗了 10 單位天然氣，而 gasPrice 為 10^18，則交易費為 10 KAIA。 參見[KAIA 單位](../kaia-native-token.md#units-of-klay)。 |
| gas                | uint64 \(Go\)                                                                                                                                          | 交易允許使用的最大燃氣量。                                                                                                                                                       |
| to                 | common.Address\(Go\)                                                                                                                   | 要執行的智能合約賬戶的地址。                                                                                                                                                      |
| value              | \*big.Int （ Go\)                                                                                                                                           | 以 `kei` 為單位的 KAIA 轉賬金額。                                                                                                                                             |
| from               | common.Address\(Go\)                                                                                                                   | 發件人地址。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |
| input              | \[\]byte \(Go\)                                                                                  | 附屬於事務的數據，用於執行事務。                                                                                                                                                    |
| txSignatures       | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 發件人簽名。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |
| feePayer           | common.Address\(Go\)                                                                                                                   | 繳費人地址。                                                                                                                                                              |
| feePayerSignatures | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 繳費人簽名。                                                                                                                                                              |

### 寄件人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-sender"></a>

要對發送方進行簽名，RLP 序列化的方法如下：

```javascript
SigRLP = encode([encode([type, nonce, gasPrice, gas, to, value, from, input]), chainid, 0, 0])
SigHash = keccak256(SigRLP)
Signature = sign(SigHash, <the sender's private key>)
```

### 繳費人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-fee-payer"></a>

要製作繳費人簽名，RLP 序列化工作應如下進行：

```javascript
SigFeePayerRLP = encode([encode([type, nonce, gasPrice, gas, to, value, from, input]), feePayer, chainid, 0, 0])
SigFeePayerHash = keccak256(SigFeePayerRLP)
SignatureFeePayer = sign(SigFeePayerHash, <the fee payer's private key>)
```

### SenderTxHash 的 RLP 編碼<a id="rlp-encoding-for-sendertxhash"></a>

要製作 SenderTxHash，RLP 序列化過程如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
SenderTxHashRLP = type + encode([nonce, gasPrice, gas, to, value, from, input, txSignatures])
SenderTxHash = keccak256(SenderTxHashRLP)
```

### 交易哈希的 RLP 編碼<a id="rlp-encoding-for-transaction-hash"></a>

要製作事務哈希值，RLP 序列化的步驟如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
feePayerSignatures (a single signature) = [[v, r, s]]
feePayerSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
TxHashRLP = type + encode([nonce, gasPrice, gas, to, value, from, input, txSignatures, feePayer, feePayerSignatures])
TxHash = keccak256(TxHashRLP)
```

### RLP 編碼 （示例）<a id="rlp-encoding-example"></a>

下面顯示的是 RLP 序列化的結果和事務對象：

```javascript
ChainID 0x1
PrivateKey 0x45a915e4d060149eb4365960e6a7a45f334393093061116b197e3240065ff2d8
PublicKey.X 0x3a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33d
PublicKey.Y 0x8072e77939dc03ba44790779b7a1025baf3003f6732430e20cd9b76d953391b3
SigRLP 0xf860b85bf859318204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0ba46353586b000000000000000000000000bc5951f055a85f41a3b62fd6f68ab7de76d299b2018080
SigHash 0xa5dd93af9f96fa316f0ddd84f10acb2e6eb41baaec3b42f9068c38aa1618f7e1
Signature f845f84325a0253aea7d2c37160da45e84afbb45f6b3341cf1e8fc2df4ecc78f14adb512dc4fa022465b74015c2a8f8501186bb5e200e6ce44be52e9374615a7e7e21c41bc27b5
FeePayerPrivateKey 0xb9d5558443585bca6f225b935950e3f6e69f9da8a5809a83f51c3365dff53936
FeePayerPublicKey.X 0x327434d4cfc66ef8857d431419e9deebdc53a3e415edcc55382e2d417b8dd102
FeePayerPublicKey.Y 0x65fc97045707faf7b8f81ac65089d4cc71f69ad0bf1bc8559bc24f13fc284ced
SigRLPFeePayer 0xf875b85bf859318204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0ba46353586b000000000000000000000000bc5951f055a85f41a3b62fd6f68ab7de76d299b2945a0043070275d9f6054307ee7348bd660849d90f018080
SigHashFeePayer 0xf547d9d0041912e0daa2db2b65170a9e833877cd8482f405a11b03429fcbd554
SignatureFeePayer f845f84326a0e7c51db7b922c6fa2a941c9687884c593b1b13076bdf0c473538d826bf7b9d1aa05b0de2aabb84b66db8bf52d62f3d3b71b592e3748455630f1504c20073624d80
TxHashRLP 0x31f8fb8204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0ba46353586b000000000000000000000000bc5951f055a85f41a3b62fd6f68ab7de76d299b2f845f84325a0253aea7d2c37160da45e84afbb45f6b3341cf1e8fc2df4ecc78f14adb512dc4fa022465b74015c2a8f8501186bb5e200e6ce44be52e9374615a7e7e21c41bc27b5945a0043070275d9f6054307ee7348bd660849d90ff845f84326a0e7c51db7b922c6fa2a941c9687884c593b1b13076bdf0c473538d826bf7b9d1aa05b0de2aabb84b66db8bf52d62f3d3b71b592e3748455630f1504c20073624d80
TxHash ef46f28c54b3d90a183e26f406ca1d5cc2b6e9fbb6cfa7c85a10330ffadf54b0
SenderTxHashRLP 0x31f89f8204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0ba46353586b000000000000000000000000bc5951f055a85f41a3b62fd6f68ab7de76d299b2f845f84325a0253aea7d2c37160da45e84afbb45f6b3341cf1e8fc2df4ecc78f14adb512dc4fa022465b74015c2a8f8501186bb5e200e6ce44be52e9374615a7e7e21c41bc27b5
SenderTxHash 3cd3380f4206943422d5d5b218dd66d03d60d19a109f9929ea12b52a230257cb

    TX(ef46f28c54b3d90a183e26f406ca1d5cc2b6e9fbb6cfa7c85a10330ffadf54b0)
    Type:          TxTypeFeeDelegatedSmartContractExecution
    From:          0xa94f5374Fce5edBC8E2a8697C15331677e6EbF0B
    To:            0x7b65B75d204aBed71587c9E519a89277766EE1d0
    Nonce:         1234
    GasPrice:      0x19
    GasLimit:      0xf4240
    Value:         0xa
    Data:          363335333538366230303030303030303030303030303030303030303030303062633539353166303535613835663431613362363266643666363861623764653736643239396232
    Signature:     [{"V":"0x25","R":"0x253aea7d2c37160da45e84afbb45f6b3341cf1e8fc2df4ecc78f14adb512dc4f","S":"0x22465b74015c2a8f8501186bb5e200e6ce44be52e9374615a7e7e21c41bc27b5"}]
    FeePayer:      0x5A0043070275d9f6054307Ee7348bD660849D90f
    FeePayerSig:   [{"V":"0x26","R":"0xe7c51db7b922c6fa2a941c9687884c593b1b13076bdf0c473538d826bf7b9d1a","S":"0x5b0de2aabb84b66db8bf52d62f3d3b71b592e3748455630f1504c20073624d80"}]
    Hex:           31f8fb8204d219830f4240947b65b75d204abed71587c9e519a89277766ee1d00a94a94f5374fce5edbc8e2a8697c15331677e6ebf0ba46353586b000000000000000000000000bc5951f055a85f41a3b62fd6f68ab7de76d299b2f845f84325a0253aea7d2c37160da45e84afbb45f6b3341cf1e8fc2df4ecc78f14adb512dc4fa022465b74015c2a8f8501186bb5e200e6ce44be52e9374615a7e7e21c41bc27b5945a0043070275d9f6054307ee7348bd660849d90ff845f84326a0e7c51db7b922c6fa2a941c9687884c593b1b13076bdf0c473538d826bf7b9d1aa05b0de2aabb84b66db8bf52d62f3d3b71b592e3748455630f1504c20073624d80
```

### RPC 輸出 （示例）<a id="rpc-output-example"></a>

下面顯示的是通過 JSON RPC 返回的事務對象。

```javascript
{
  "blockHash": "0x82983fe294d286e76486760e6904369285554e1744af16786c2393a956fb4ec4",
  "blockNumber": "0x2",
  "contractAddress": null,
  "feePayer": "0x029fdce0457db02f05c4be9f67b7115cb8ea15ca",
  "feePayerSignatures": [
    {
      "V": "0x25",
      "R": "0x1c7de2c83542b623ba47722f310c0e5893486eef4eed70b634d456262fb430a7",
      "S": "0x177929c52669c4b9433565a76e53723b702bae8142debe1981062f59f25062ab"
    }
  ],
  "from": "0x0fcda0f2efbe1b4e61b487701ce4f2f8abc3723d",
  "gas": "0x174876e800",
  "gasPrice": "0x0",
  "gasUsed": "0xb0bc",
  "input": "0x6353586b0000000000000000000000000fcda0f2efbe1b4e61b487701ce4f2f8abc3723d",
  "logs": [],
  "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "nonce": "0xe",
  "senderTxHash": "0xffd354e4e271ff94a7459c2f1bc0df20dc112a83f5625ff7e31d196444f72710",
  "signatures": [
    {
      "V": "0x25",
      "R": "0xefc6fec3dae47a08941712f637c95dbc46ef2afd3d16e68da602a878c0bba047",
      "S": "0x938a5374edcea0503df8e7af906a7642f7e935eab7c489b7ca8b976a8e5ab7e"
    }
  ],
  "status": "0x1",
  "to": "0x636f6e74726163742e6b6c6179746e0000000000",
  "transactionHash": "0x658a118112ffb0c06adecd59b0f11b58cf7d8afd7ec5e5d323cfca021c3dcb37",
  "transactionIndex": "0x7",
  "type": "TxTypeFeeDelegatedSmartContractExecution",
  "typeInt": 49,
  "value": "0xa"
}
```

## TxTypeFeeDelegatedAccountUpdate <a id="txtypefeedelegatedaccountupdate"></a>

TxTypeFeeDelegatedAccountUpdate 更新給定賬戶的密鑰。 交易費由繳費人支付。 該交易類型將進行以下更改。

1. 付費者的餘額會減少交易費的金額。
2. 發送方的 nonce 增加一個。
3. 賬戶密鑰用 `key` 更新。
4. 執行此類交易後，從賬戶發送的交易將使用新的 "密鑰 "進行驗證。
5. 交易費由繳費人支付。

### 屬性<a id="attributes"></a>

| 屬性                 | 類型                                                                                                                                                                          | 說明                                                                                                                                                                    |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| type               | uint8\(Go\)                                                                                                                                            | TxTypeAccountUpdate 的類型。 必須為 0x21。                                                                                                                                    |
| nonce              | uint64 \(Go\)                                                                                                                                          | 用於唯一標識發件人交易的值。 如果一個發送方生成了兩個具有相同 nonce 的交易，則只執行其中一個。                                                                                                                   |
| gasPrice           | \*big.Int （ Go\)                                                                                                                                           | 一個乘數，用於計算發件人將支付多少代幣。 發送方將支付的代幣數量通過 `gas` \* `gasPrice` 計算得出。 例如，如果 gas 為 10，gasPrice 為 10^18，發件人將支付 10 KAIA 的交易費。 參見[KAIA 單位](../kaia-native-token.md#units-of-klay)。 |
| gas                | uint64 \(Go\)                                                                                                                                          | 交易允許使用的最高交易費金額。                                                                                                                                                       |
| from               | common.Address\(Go\)                                                                                                                   | 發件人地址。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                    |
| key                | AccountKey\(Go\)                                                                                                                                       | [AccountKey](../accounts.md#account-key)將更新到賬戶。                                                                                                                       |
| txSignatures       | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 發件人簽名。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                    |
| feePayer           | common.Address\(Go\)                                                                                                                   | 繳費人地址。                                                                                                                                                                |
| feePayerSignatures | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 繳費人簽名。                                                                                                                                                                |

### 寄件人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-sender"></a>

要對發送方進行簽名，RLP 序列化的方法如下：

```javascript
SigRLP = encode([encode([type, nonce, gasPrice, gas, from, rlpEncodedKey]), chainid, 0, 0])`
SigHash = keccak256(SigRLP)
Signature = sign(SigHash, <the sender's private key>)
```

### 繳費人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-fee-payer"></a>

要製作繳費人簽名，RLP 序列化工作應如下進行：

```javascript
SigFeePayerRLP = encode([encode([type, nonce, gasPrice, gas, from, rlpEncodedKey]), feePayer, chainid, 0, 0])
SigFeePayerHash = keccak256(SigFeePayerRLP)
SignatureFeePayer = sign(SigFeePayerHash, <the fee payer's private key>)
```

### SenderTxHash 的 RLP 編碼<a id="rlp-encoding-for-sendertxhash"></a>

要製作 SenderTxHash，RLP 序列化過程如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
SenderTxHashRLP = type + encode([nonce, gasPrice, gas, from, rlpEncodedKey, txSignatures])
SenderTxHash = keccak256(SenderTxHashRLP)
```

### 交易哈希的 RLP 編碼<a id="rlp-encoding-for-transaction-hash"></a>

要製作事務哈希值，RLP 序列化的步驟如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
feePayerSignatures (a single signature) = [[v, r, s]]
feePayerSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
TxHashRLP = type + encode([nonce, gasPrice, gas, from, rlpEncodedKey, txSignatures, feePayer, feePayerSignatures])
TxHash = keccak256(TxHashRLP)
```

### RLP 編碼 （示例）<a id="rlp-encoding-example"></a>

下面顯示的是 RLP 序列化的結果和事務對象：

```javascript
ChainID 0x1
PrivateKey 0x45a915e4d060149eb4365960e6a7a45f334393093061116b197e3240065ff2d8
PublicKey.X 0x3a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33d
PublicKey.Y 0x8072e77939dc03ba44790779b7a1025baf3003f6732430e20cd9b76d953391b3
SigRLP 0xf849b844f842218204d219830f424094a94f5374fce5edbc8e2a8697c15331677e6ebf0ba302a1033a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33d018080
SigHash 0x78437953e6beb985ea3ccbee8d6a648a09d11249389477a32c7094fc7b8765ef
Signature f845f84326a0ab69d9adca15d9763c4ce6f98b35256717c6e932007658f19c5a255de9e70ddaa026aa676a3a1a6e96aff4a3df2335788d614d54fb4db1c3c48551ce1fa7ac5e52
FeePayerPrivateKey 0xb9d5558443585bca6f225b935950e3f6e69f9da8a5809a83f51c3365dff53936
FeePayerPublicKey.X 0x327434d4cfc66ef8857d431419e9deebdc53a3e415edcc55382e2d417b8dd102
FeePayerPublicKey.Y 0x65fc97045707faf7b8f81ac65089d4cc71f69ad0bf1bc8559bc24f13fc284ced
SigRLPFeePayer 0xf85eb844f842218204d219830f424094a94f5374fce5edbc8e2a8697c15331677e6ebf0ba302a1033a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33d945a0043070275d9f6054307ee7348bd660849d90f018080
SigHashFeePayer 0x1026d3ac74f56b52453d656b084d06798479b8bcfda1868d8beaa23e36f3aeb3
SignatureFeePayer f845f84326a0f295cd69b4144d9dbc906ba144933d2cc535d9d559f7a92b4672cc5485bf3a60a0784b8060234ffd64739b5fc2f2503939340ab4248feaa6efcf62cb874345fe40
TxHashRLP 0x21f8e48204d219830f424094a94f5374fce5edbc8e2a8697c15331677e6ebf0ba302a1033a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33df845f84326a0ab69d9adca15d9763c4ce6f98b35256717c6e932007658f19c5a255de9e70ddaa026aa676a3a1a6e96aff4a3df2335788d614d54fb4db1c3c48551ce1fa7ac5e52945a0043070275d9f6054307ee7348bd660849d90ff845f84326a0f295cd69b4144d9dbc906ba144933d2cc535d9d559f7a92b4672cc5485bf3a60a0784b8060234ffd64739b5fc2f2503939340ab4248feaa6efcf62cb874345fe40
TxHash 756ff5d3912a4089659614d42a218eee59e602a5992bddca383c2d295c6637bb
SenderTxHashRLP 0x21f8888204d219830f424094a94f5374fce5edbc8e2a8697c15331677e6ebf0ba302a1033a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33df845f84326a0ab69d9adca15d9763c4ce6f98b35256717c6e932007658f19c5a255de9e70ddaa026aa676a3a1a6e96aff4a3df2335788d614d54fb4db1c3c48551ce1fa7ac5e52
SenderTxHash f56937017bd3b75c637ba5b4ce90df20c166006a2a529b42e808bc806159b98f

    TX(756ff5d3912a4089659614d42a218eee59e602a5992bddca383c2d295c6637bb)
    Type:          TxTypeFeeDelegatedAccountUpdate
    From:          0xa94f5374Fce5edBC8E2a8697C15331677e6EbF0B
    Nonce:         1234
    GasPrice:      0x19
    GasLimit:      0xf4240
    Key:           AccountKeyPublic: S256Pubkey:{"x":"0x3a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33d","y":"0x8072e77939dc03ba44790779b7a1025baf3003f6732430e20cd9b76d953391b3"}
    Signature:     [{"V":"0x26","R":"0xab69d9adca15d9763c4ce6f98b35256717c6e932007658f19c5a255de9e70dda","S":"0x26aa676a3a1a6e96aff4a3df2335788d614d54fb4db1c3c48551ce1fa7ac5e52"}]
    FeePayer:      0x5A0043070275d9f6054307Ee7348bD660849D90f
    FeePayerSig:   [{"V":"0x26","R":"0xf295cd69b4144d9dbc906ba144933d2cc535d9d559f7a92b4672cc5485bf3a60","S":"0x784b8060234ffd64739b5fc2f2503939340ab4248feaa6efcf62cb874345fe40"}]
    Hex:           21f8e48204d219830f424094a94f5374fce5edbc8e2a8697c15331677e6ebf0ba302a1033a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33df845f84326a0ab69d9adca15d9763c4ce6f98b35256717c6e932007658f19c5a255de9e70ddaa026aa676a3a1a6e96aff4a3df2335788d614d54fb4db1c3c48551ce1fa7ac5e52945a0043070275d9f6054307ee7348bd660849d90ff845f84326a0f295cd69b4144d9dbc906ba144933d2cc535d9d559f7a92b4672cc5485bf3a60a0784b8060234ffd64739b5fc2f2503939340ab4248feaa6efcf62cb874345fe40
```

### RPC 輸出 （示例）<a id="rpc-output-example"></a>

下面顯示的是通過 JSON RPC 返回的事務對象。

```javascript
{
  "blockHash": "0x82983fe294d286e76486760e6904369285554e1744af16786c2393a956fb4ec4",
  "blockNumber": "0x2",
  "contractAddress": null,
  "feePayer": "0x0fcda0f2efbe1b4e61b487701ce4f2f8abc3723d",
  "feePayerSignatures": [
    {
      "V": "0x25",
      "R": "0x3b019642e5ae37f3ecbf85e6fc1ee77e51d1618299367bcedd816d0da6afb1e0",
      "S": "0x5c12c87811a74183f8b56b707fa90a916b1c641652c93e52300f5cee36141d73"
    }
  ],
  "from": "0x636f6c696e322e6b6c6179746e00000000000000",
  "gas": "0x174876e800",
  "gasPrice": "0x5d21dba00",
  "gasUsed": "0xc738",
  "key": "0x02a1034ef27ba4b7d1ae09b166744c5b7ee4a7a0cc5c76b2e5d74523a0a4fb56db3191",
  "logs": [],
  "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "nonce": "0x0",
  "senderTxHash": "0xf4e7ef082451d4a3c8ad7c4348fc99c965a9c130bfc98d7971f3103e3dcfda3c",
  "signatures": [
    {
      "V": "0x26",
      "R": "0xd4cb16abcdf92969dc45efacaa5827ad55738fbda08a3dbaf0f0553643084a6",
      "S": "0x23f8055933b416cf15568a017e0a11e0a5c0a8f65477f6ec71de0bf837f4a681"
    }
  ],
  "status": "0x1",
  "transactionHash": "0xeb0c14d903db38deee116ac8a0d620e6ca6aa79e4f91393abbddfa30810b9d43",
  "transactionIndex": "0x2",
  "type": "TxTypeFeeDelegatedAccountUpdate",
  "typeInt": 33
},
```

## TxTypeFeeDelegatedCancel <a id="txtypefeedelegatedcancel"></a>

TxTypeFeeDelegatedCancel 取消執行事務池中具有相同 nonce 的事務。 更多詳情，請參閱 [TxTypeCancel](./basic.md#txtypecancel)。

以下更改將適用於該交易類型。 1. 付費者的餘額會減少交易費的金額。 2. 發送方的 nonce 增加一個。

### 屬性<a id="attributes"></a>

| 屬性                 | 類型                                                                                                                                                                          | 說明                                                                                                                                                                  |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| type               | uint8\(Go\)                                                                                                                                            | TxTypeCancel 的類型。 必須為 0x39。                                                                                                                                         |
| nonce              | uint64 \(Go\)                                                                                                                                          | 用於唯一標識發件人交易的值。 如果一個發送方生成了兩個具有相同 nonce 的交易，則只執行其中一個。                                                                                                                 |
| gasPrice           | \*big.Int （ Go\)                                                                                                                                           | 以 `kei` 為單位的氣體單價，發件人將支付交易費。 交易費的計算公式為 `gas`\* `gasPrice`。 例如，如果交易消耗了 10 單位天然氣，而 gasPrice 為 10^18，則交易費為 10 KAIA。 參見[KAIA 單位](../kaia-native-token.md#units-of-klay)。 |
| gas                | uint64 \(Go\)                                                                                                                                          | 交易允許使用的最高交易費金額。                                                                                                                                                     |
| from               | common.Address\(Go\)                                                                                                                   | 發件人地址。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |
| txSignatures       | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 發件人簽名。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |
| feePayer           | common.Address\(Go\)                                                                                                                   | 繳費人地址。                                                                                                                                                              |
| feePayerSignatures | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 繳費人簽名。                                                                                                                                                              |

### 寄件人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-sender"></a>

要對發送方進行簽名，RLP 序列化的方法如下：

```javascript
SigRLP = encode([encode([type, nonce, gasPrice, gas, from]), chainid, 0, 0])
SigHash = keccak256(SigRLP)
Signature = sign(SigHash, <the sender's private key>)
```

### 繳費人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-fee-payer"></a>

要製作繳費人簽名，RLP 序列化工作應如下進行：

```javascript
SigFeePayerRLP = encode([encode([type, nonce, gasPrice, gas, from]), feePayer, chainid, 0, 0])
SigFeePayerHash = keccak256(SigFeePayerRLP)
SignatureFeePayer = sign(SigFeePayerHash, <the fee payer's private key>)
```

### SenderTxHash 的 RLP 編碼<a id="rlp-encoding-for-sendertxhash"></a>

要製作 SenderTxHash，RLP 序列化過程如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
SenderTxHashRLP = type + encode([nonce, gasPrice, gas, from, txSignatures])
SenderTxHash = keccak256(SenderTxHashRLP)
```

### 交易哈希的 RLP 編碼<a id="rlp-encoding-for-transaction-hash"></a>

要製作事務哈希值，RLP 序列化的步驟如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
feePayerSignatures (a single signature) = [[v, r, s]]
feePayerSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
TxHashRLP = type + encode([nonce, gasPrice, gas, from, txSignatures, feePayer, feePayerSignatures])
TxHash = keccak256(TxHashRLP)
```

### RLP 編碼 （示例）<a id="rlp-encoding-example"></a>

下面顯示的是 RLP 序列化的結果和事務對象：

```javascript
ChainID 0x1
PrivateKey 0x45a915e4d060149eb4365960e6a7a45f334393093061116b197e3240065ff2d8
PublicKey.X 0x3a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33d
PublicKey.Y 0x8072e77939dc03ba44790779b7a1025baf3003f6732430e20cd9b76d953391b3
SigRLP 0xe39fde398204d219830f424094a94f5374fce5edbc8e2a8697c15331677e6ebf0b018080
SigHash 0xd36c4277f4aa1d483a5fc4d656aeea50416c28adddb27a234d320290bd2a343c
Signature f845f84326a08409f5441d4725f90905ad87f03793857d124de7a43169bc67320cd2f020efa9a060af63e87bdc565d7f7de906916b2334336ee7b24d9a71c9521a67df02e7ec92
FeePayerPrivateKey 0xb9d5558443585bca6f225b935950e3f6e69f9da8a5809a83f51c3365dff53936
FeePayerPublicKey.X 0x327434d4cfc66ef8857d431419e9deebdc53a3e415edcc55382e2d417b8dd102
FeePayerPublicKey.Y 0x65fc97045707faf7b8f81ac65089d4cc71f69ad0bf1bc8559bc24f13fc284ced
SigRLPFeePayer 0xf8389fde398204d219830f424094a94f5374fce5edbc8e2a8697c15331677e6ebf0b945a0043070275d9f6054307ee7348bd660849d90f018080
SigHashFeePayer 0x15859ecc06acbd2dd5820c5968a85590826d1f6affb938e89559558ac4f86a24
SignatureFeePayer f845f84326a0044d5b25e8c649a1fdaa409dc3817be390ad90a17c25bc17c89b6d5d248495e0a073938e690d27b5267c73108352cf12d01de7fd0077b388e94721aa1fa32f85ec
TxHashRLP 0x39f8c08204d219830f424094a94f5374fce5edbc8e2a8697c15331677e6ebf0bf845f84326a08409f5441d4725f90905ad87f03793857d124de7a43169bc67320cd2f020efa9a060af63e87bdc565d7f7de906916b2334336ee7b24d9a71c9521a67df02e7ec92945a0043070275d9f6054307ee7348bd660849d90ff845f84326a0044d5b25e8c649a1fdaa409dc3817be390ad90a17c25bc17c89b6d5d248495e0a073938e690d27b5267c73108352cf12d01de7fd0077b388e94721aa1fa32f85ec
TxHash 96b39d3ab849127d31a5f7b5c882ca9ba408cd9d875052640d51a64f8c4acbb2
SenderTxHashRLP 0x39f8648204d219830f424094a94f5374fce5edbc8e2a8697c15331677e6ebf0bf845f84326a08409f5441d4725f90905ad87f03793857d124de7a43169bc67320cd2f020efa9a060af63e87bdc565d7f7de906916b2334336ee7b24d9a71c9521a67df02e7ec92
SenderTxHash cc6c2673398903b3d906a3023b41636fc08bd1bddd5aa1602116091638f48447

    TX(96b39d3ab849127d31a5f7b5c882ca9ba408cd9d875052640d51a64f8c4acbb2)
    Type:          TxTypeFeeDelegatedCancel
    From:          0xa94f5374Fce5edBC8E2a8697C15331677e6EbF0B
    Nonce:         1234
    GasPrice:      0x19
    GasLimit:      0xf4240
    Signature:     [{"V":"0x26","R":"0x8409f5441d4725f90905ad87f03793857d124de7a43169bc67320cd2f020efa9","S":"0x60af63e87bdc565d7f7de906916b2334336ee7b24d9a71c9521a67df02e7ec92"}]
    FeePayer:      0x5A0043070275d9f6054307Ee7348bD660849D90f
    FeePayerSig:   [{"V":"0x26","R":"0x44d5b25e8c649a1fdaa409dc3817be390ad90a17c25bc17c89b6d5d248495e0","S":"0x73938e690d27b5267c73108352cf12d01de7fd0077b388e94721aa1fa32f85ec"}]
    Hex:           39f8c08204d219830f424094a94f5374fce5edbc8e2a8697c15331677e6ebf0bf845f84326a08409f5441d4725f90905ad87f03793857d124de7a43169bc67320cd2f020efa9a060af63e87bdc565d7f7de906916b2334336ee7b24d9a71c9521a67df02e7ec92945a0043070275d9f6054307ee7348bd660849d90ff845f84326a0044d5b25e8c649a1fdaa409dc3817be390ad90a17c25bc17c89b6d5d248495e0a073938e690d27b5267c73108352cf12d01de7fd0077b388e94721aa1fa32f85ec
```

### RPC 輸出 （示例）<a id="rpc-output-example"></a>

下面顯示的是通過 JSON RPC 返回的事務對象。

```javascript
{
  "blockHash": "0x82983fe294d286e76486760e6904369285554e1744af16786c2393a956fb4ec4",
  "blockNumber": "0x2",
  "contractAddress": null,
  "feePayer": "0x029fdce0457db02f05c4be9f67b7115cb8ea15ca",
  "feePayerSignatures": [
    {
      "V": "0x25",
      "R": "0x26a7c88e1fc77400f2a4c7911966a5e51b0873e3f26daf9d6519b93e3f3db6a3",
      "S": "0x560e5fa8d53ebf899eb48353bf14794c76784240a6a212f5ddbe7f1684088f3f"
    }
  ],
  "from": "0x0fcda0f2efbe1b4e61b487701ce4f2f8abc3723d",
  "gas": "0x174876e800",
  "gasPrice": "0x5d21dba00",
  "gasUsed": "0x7918",
  "logs": [],
  "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "nonce": "0x11",
  "senderTxHash": "0x2fea0ff37b8b936d4c06f29b98c4bd200827423fb445f931eb64725aefcda053",
  "signatures": [
    {
      "V": "0x26",
      "R": "0xcfdb5b3ff6c87a8f18ae606b371d1e569c56d35a737831b89052c5a8ef19d049",
      "S": "0x1ee63bd5a01c45d0c6f1b36a29e1c01b56baa719f008c556bc9054ac5a64bd8d"
    }
  ],
  "status": "0x1",
  "transactionHash": "0xf475e714b30aef0b79d46c9482289f3fbe51f1e44bcbc99a90ac8e25672bc969",
  "transactionIndex": "0xa",
  "type": "TxTypeFeeDelegatedCancel",
  "typeInt": 57
}
```

## TxTypeFeeDelegatedChainDataAnchoring <a id="txtypefeedelegatedchaindataanchoring"></a>

TxTypeFeeDelegatedChainDataAnchoring 是將服務鏈數據錨定到 Kaia 主鏈的收費委託事務。 服務鏈會定期向 Kaia 主鏈發送此類交易，以確保數據的安全性和可信度。 有關數據錨定的詳細信息，請參閱 [錨定](../../nodes/service-chain/configure/anchoring.md)。 由於這也是一項收費委託交易，其交易費由收費者支付。 請注意，不允許通過 RPC 發送此事務。 目前，出於安全考慮，這種交易是通過私人 P2P 渠道執行的。 這筆交易不會改變 Kaia 區塊鏈的狀態，只是發送者的非ce 增加了一個。

### 屬性<a id="attributes"></a>

| 屬性                 | 類型                                                                                                                                                                          | 說明                                                                                                                                                                  |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| type               | uint8\(Go\)                                                                                                                                            | TxTypeFeeDelegatedChainDataAnchoring 的類型。 必須為 0x49。                                                                                                                 |
| nonce              | uint64 \(Go\)                                                                                                                                          | 用於唯一標識發件人交易的值。 如果一個發送方生成了兩個具有相同 nonce 的交易，則只執行其中一個。                                                                                                                 |
| gasPrice           | \*big.Int （ Go\)                                                                                                                                           | 以 `kei` 為單位的氣體單價，發件人將支付交易費。 交易費的計算公式為 `gas`\* `gasPrice`。 例如，如果交易消耗了 10 單位天然氣，而 gasPrice 為 10^18，則交易費為 10 KAIA。 參見[KAIA 單位](../kaia-native-token.md#units-of-klay)。 |
| gas                | uint64 \(Go\)                                                                                                                                          | 交易允許使用的最高交易費金額。                                                                                                                                                     |
| from               | common.Address\(Go\)                                                                                                                   | 發件人地址。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |
| input              | \[\]byte \(Go\)                                                                                  | 服務鏈數據。                                                                                                                                                              |
| txSignatures       | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 發件人簽名。 更多詳情，請參閱 [交易的簽名驗證](./transactions.md#signature-validation-of-transactions)。                                                                                  |
| feePayer           | common.Address\(Go\)                                                                                                                   | 繳費人地址。                                                                                                                                                              |
| feePayerSignatures | \[\]\{\*big.Int, \*big.Int, \*big.Int\} \(Go\) | 繳費人簽名。                                                                                                                                                              |

### 寄件人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-sender"></a>

要對發送方進行簽名，RLP 序列化的方法如下：

```javascript
SigRLP = encode([encode([type, nonce, gasPrice, gas, from, anchoredData]), chainid, 0, 0])
SigHash = keccak256(SigRLP)
Signature = sign(SigHash, <private key>)
```

### 繳費人簽名的 RLP 編碼<a id="rlp-encoding-for-signature-of-the-fee-payer"></a>

要製作繳費人簽名，RLP 序列化工作應如下進行：

```javascript
SigFeePayerRLP = encode([encode([type, nonce, gasPrice, gas, from, anchoredData]), feePayer, chainid, 0, 0])
SigFeePayerHash = keccak256(SigFeePayerRLP)
SignatureFeePayer = sign(SigFeePayerHash, <the fee payer's private key>)
```

### SenderTxHash 的 RLP 編碼<a id="rlp-encoding-for-sendertxhash"></a>

要製作 SenderTxHash，RLP 序列化過程如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
SenderTxHashRLP = type + encode([nonce, gasPrice, gas, from, anchoredData, txSignatures])
SenderTxHash = keccak256(SenderTxHashRLP)
```

### 交易哈希的 RLP 編碼<a id="rlp-encoding-for-transaction-hash"></a>

要製作事務哈希值，RLP 序列化的步驟如下：

```javascript
txSignatures (a single signature) = [[v, r, s]]
txSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
feePayerSignatures (a single signature) = [[v, r, s]]
feePayerSignatures (two signatures) = [[v1, r1, s1], [v2, r2, s2]]
TxHashRLP = type + encode([nonce, gasPrice, gas, from, anchoredData, txSignatures, feePayer, feePayerSignatures])
TxHash = keccak256(TxHashRLP)
```

### RLP 編碼 （示例）<a id="rlp-encoding-example"></a>

下面顯示的是 RLP 序列化的結果和事務對象：

```javascript
ChainID 0x01
PrivateKey 0x45a915e4d060149eb4365960e6a7a45f334393093061116b197e3240065ff2d8
PublicKey.X 0x3a514176466fa815ed481ffad09110a2d344f6c9b78c1d14afc351c3a51be33d
PublicKey.Y 0x8072e77939dc03ba44790779b7a1025baf3003f6732430e20cd9b76d953391b3
SigRLP 0xf8dbb8d6f8d449118505d21dba0085174876e80094a94f5374fce5edbc8e2a8697c15331677e6ebf0bb8aff8ad80b8aaf8a8a00000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000001a00000000000000000000000000000000000000000000000000000000000000002a00000000000000000000000000000000000000000000000000000000000000003a00000000000000000000000000000000000000000000000000000000000000004058006018080
SigHash 0x92e385b4a162170ee87b2b2e598f686b1d16f385d98ad626147305624abec0b3
Signature 0xf845f84326a0afe41edc9cce1185ab9065ca7dbfb89ab5c7bde3602a659aa258324124644142a0317848698248ba7cc057b8f0dd19a27b52ef904d29cb72823100f1ed18ba2bb3
FeePayerPublicKey.X 0x327434d4cfc66ef8857d431419e9deebdc53a3e415edcc55382e2d417b8dd102
FeePayerPublicKey.Y 0x65fc97045707faf7b8f81ac65089d4cc71f69ad0bf1bc8559bc24f13fc284ced
SigRLPFeePayer 0xf8f0b8d6f8d449118505d21dba0085174876e80094a94f5374fce5edbc8e2a8697c15331677e6ebf0bb8aff8ad80b8aaf8a8a00000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000001a00000000000000000000000000000000000000000000000000000000000000002a00000000000000000000000000000000000000000000000000000000000000003a000000000000000000000000000000000000000000000000000000000000000040580069433f524631e573329a550296f595c820d6c65213f018080
SigHashFeePayer 0x4d58fdf276fde1e221b6bab8c6621ae1639b00a7a70d2bd0a114001692a3a7d1
SignatureFeePayer 0xf845f84325a0309e46db21a1bf7bfdae24d9192aca69516d6a341ecce8971fc69cff481cee76a04b939bf7384c4f919880307323a5e36d4d6e029bae1887a43332710cdd48f174
TxHashRLP 0x49f90176118505d21dba0085174876e80094a94f5374fce5edbc8e2a8697c15331677e6ebf0bb8aff8ad80b8aaf8a8a00000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000001a00000000000000000000000000000000000000000000000000000000000000002a00000000000000000000000000000000000000000000000000000000000000003a00000000000000000000000000000000000000000000000000000000000000004058006f845f84326a0afe41edc9cce1185ab9065ca7dbfb89ab5c7bde3602a659aa258324124644142a0317848698248ba7cc057b8f0dd19a27b52ef904d29cb72823100f1ed18ba2bb39433f524631e573329a550296f595c820d6c65213ff845f84325a0309e46db21a1bf7bfdae24d9192aca69516d6a341ecce8971fc69cff481cee76a04b939bf7384c4f919880307323a5e36d4d6e029bae1887a43332710cdd48f174
TxHash 0xecf1ec12937065617f9b3cd07570452bfdb75dc36404c4f37f78995c6dc462af
SenderTxHashRLP 0x49f9011a118505d21dba0085174876e80094a94f5374fce5edbc8e2a8697c15331677e6ebf0bb8aff8ad80b8aaf8a8a00000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000001a00000000000000000000000000000000000000000000000000000000000000002a00000000000000000000000000000000000000000000000000000000000000003a00000000000000000000000000000000000000000000000000000000000000004058006f845f84326a0afe41edc9cce1185ab9065ca7dbfb89ab5c7bde3602a659aa258324124644142a0317848698248ba7cc057b8f0dd19a27b52ef904d29cb72823100f1ed18ba2bb3
SenderTxHash 0x4f5c00ea8f6346baa7d4400dfefd72efa5ec219561ebcebed7be8a2b79d52bcd

	TX(ecf1ec12937065617f9b3cd07570452bfdb75dc36404c4f37f78995c6dc462af)
	Type:          TxTypeFeeDelegatedChainDataAnchoring
	From:          0xa94f5374Fce5edBC8E2a8697C15331677e6EbF0B
	Nonce:         17
	GasPrice:      0x5d21dba00
	GasLimit:      0x174876e800
	AnchoredData:  f8ad80b8aaf8a8a00000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000001a00000000000000000000000000000000000000000000000000000000000000002a00000000000000000000000000000000000000000000000000000000000000003a00000000000000000000000000000000000000000000000000000000000000004058006
	Signature:     [{"V":"0x26","R":"0xafe41edc9cce1185ab9065ca7dbfb89ab5c7bde3602a659aa258324124644142","S":"0x317848698248ba7cc057b8f0dd19a27b52ef904d29cb72823100f1ed18ba2bb3"}]
	FeePayer:      0x33f524631e573329a550296F595c820D6c65213f
	FeePayerSig:   [{"V":"0x25","R":"0x309e46db21a1bf7bfdae24d9192aca69516d6a341ecce8971fc69cff481cee76","S":"0x4b939bf7384c4f919880307323a5e36d4d6e029bae1887a43332710cdd48f174"}]
	Hex:           49f90176118505d21dba0085174876e80094a94f5374fce5edbc8e2a8697c15331677e6ebf0bb8aff8ad80b8aaf8a8a00000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000001a00000000000000000000000000000000000000000000000000000000000000002a00000000000000000000000000000000000000000000000000000000000000003a00000000000000000000000000000000000000000000000000000000000000004058006f845f84326a0afe41edc9cce1185ab9065ca7dbfb89ab5c7bde3602a659aa258324124644142a0317848698248ba7cc057b8f0dd19a27b52ef904d29cb72823100f1ed18ba2bb39433f524631e573329a550296f595c820d6c65213ff845f84325a0309e46db21a1bf7bfdae24d9192aca69516d6a341ecce8971fc69cff481cee76a04b939bf7384c4f919880307323a5e36d4d6e029bae1887a43332710cdd48f174
```

### RPC 輸出 （示例）<a id="rpc-output-example"></a>

下面顯示的是通過 JSON RPC 返回的事務對象。

```javascript
{
    "blockHash": "0x170a32e16b6fdced144d5104f5aecf753878bd9f1a7d87ddccc2e6d2ba27354c",
    "blockNumber": "0x2",
    "contractAddress": null,
    "feePayer": "0x33f524631e573329a550296f595c820d6c65213f",
    "feePayerSignatures": [
        {
            "V": "0x25",
            "R": "0x309e46db21a1bf7bfdae24d9192aca69516d6a341ecce8971fc69cff481cee76",
            "S": "0x4b939bf7384c4f919880307323a5e36d4d6e029bae1887a43332710cdd48f174"
        }
    ],
    "from": "0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b",
    "gas": "0x174876e800",
    "gasPrice": "0x5d21dba00",
    "gasUsed": "0xbd74",
    "input": "0xf8ad80b8aaf8a8a00000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000001a00000000000000000000000000000000000000000000000000000000000000002a00000000000000000000000000000000000000000000000000000000000000003a00000000000000000000000000000000000000000000000000000000000000004058006",
    "logs": [],
    "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "nonce": "0x11",
    "senderTxHash": "0x4f5c00ea8f6346baa7d4400dfefd72efa5ec219561ebcebed7be8a2b79d52bcd",
    "signatures": [
        {
            "V": "0x26",
            "R": "0xafe41edc9cce1185ab9065ca7dbfb89ab5c7bde3602a659aa258324124644142",
            "S": "0x317848698248ba7cc057b8f0dd19a27b52ef904d29cb72823100f1ed18ba2bb3"
        }
    ],
    "status": "0x1",
    "transactionHash": "0xecf1ec12937065617f9b3cd07570452bfdb75dc36404c4f37f78995c6dc462af",
    "transactionIndex": "0xa",
    "type": "TxTypeFeeDelegatedChainDataAnchoring",
    "typeInt": 73
}
```
