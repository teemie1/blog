## 1. Run payjoin
~~~
dan@mutiny:~/rust-payjoin/payjoin-cli$ RUST_LOG=debug cargo run  --  receive 10000
    Finished dev [unoptimized + debuginfo] target(s) in 0.22s
     Running `/home/dan/rust-payjoin/target/debug/payjoin receive 10000`
[2023-08-02T09:28:11Z DEBUG bitcoincore_rpc] JSON-RPC request: getnewaddress [null,null]
Listening at 0.0.0.0:3000. Configured to accept payjoin at BIP 21 Payjoin Uri:
BITCOIN:TB1Q3F2NZL7XQQ3X9NQ8A5XADXEAGF2J8FDH3AF6Y0?amount=0.0001&pj=https://localhost:3010
[2023-08-02T09:28:12Z DEBUG tiny_http] Server listening on 0.0.0.0:3000
[2023-08-02T09:28:12Z DEBUG tiny_http] Running accept thread

~~~

## 2. Issue joinmarket to pay for payjoin with http
~~~
(jmvenv) tee@mutiny:/tmp/joinmarket-clientserver-0.9.9/scripts$ python sendpayment.py -m 0 wallet.jmdat  "BITCOIN:TB1Q3F2NZL7XQQ3X9NQ8A5XADXEAGF2J8FDH3AF6Y0?amount=0.0001&pj=http://127.0.0.1:3000"
User data location: /home/tee/.joinmarket/
Attempting to pay via payjoin.
2023-08-02 10:09:54,830 [INFO]  starting sendpayment
Enter passphrase to decrypt wallet:
2023-08-02 10:10:04,362 [INFO]  BIP78 daemon listening on port 25183
2023-08-02 10:10:04,366 [WARNING]  Could not source a fee estimate from Core, falling back to default: 10000 sat/vkB (10.0 sat/vB).
2023-08-02 10:10:04,366 [INFO]  Using bitcoin network feerate for 3 block confirmation target (randomized for privacy): 8305 sat/vkB (8.3 sat/vB)
2023-08-02 10:10:04,376 [WARNING]  Could not source a fee estimate from Core, falling back to default: 10000 sat/vkB (10.0 sat/vB).
2023-08-02 10:10:04,376 [INFO]  Using bitcoin network feerate for 3 block confirmation target (randomized for privacy): 9847 sat/vkB (9.8 sat/vB)
2023-08-02 10:10:04,422 [INFO]  Using a fee of: 0.00001388 BTC (1388 sat).
2023-08-02 10:10:04,422 [INFO]  Using a change value of: 0.00977123 BTC (977123 sat).
Completed PSBT created:
{
    "psbt-version": 0,
    "unsigned-tx": {
        "hex": "020000000182e9375743e3ef976a4fa44a450880bfe98ea981d7db7e23d3d7dcf1016066870100000000fdffffff02e3e80e0000000000160014c45edd801ca3a8f86459a5c2e9292c943597252210270000000000001600148a55317fc6002262cc07ed0dd69b3d425523a5b7176f0400",
        "inputs": [
            {
                "outpoint": "87666001f1dcd7d3237edbd781a98ee9bf8008454aa44f6a97efe3435737e982:1",
                "scriptSig": "",
                "nSequence": 4294967293
            }
        ],
        "outputs": [
            {
                "value_sats": 977123,
                "scriptPubKey": "0014c45edd801ca3a8f86459a5c2e9292c9435972522",
                "address": "tb1qc30dmqqu5w50seze5hpwj2fvjs6ewffzht8p6j"
            },
            {
                "value_sats": 10000,
                "scriptPubKey": "00148a55317fc6002262cc07ed0dd69b3d425523a5b7",
                "address": "tb1q3f2nzl7xqq3x9nq8a5xadxeagf2j8fdh3af6y0"
            }
        ],
        "txid": "0bf3a449c2227c25e861cbb90c239f3051fad0bb08b46e98cb09efa58e2d4f46",
        "nLockTime": 290583,
        "nVersion": 2
    },
    "psbt-inputs": [
        {
            "input-index": 0,
            "utxo": {
                "value_sats": 988511,
                "scriptPubKey": "00140dc819d26b2c91d34aee70bd4280425bd29a4330",
                "address": "tb1qphypn5nt9jgaxjhwwz759qzzt0ff5sesuqnctf"
            },
            "final-scriptSig": "",
            "final-scriptWitness": "02483045022100da17abc3ad4337312e9101f50a8a0cbcf5e926b4c86439a666ca0e09d5981c37022067641f7337d54a8057e8dcd240e326aafa44ac3c8547726c0b9b856d49a856fb01210236a55a20c8e577b8074eff6d14e0b9347f5a6fe4f28e60d4f34b7c6fdb625523"
        }
    ],
    "psbt-outputs": [
        {
            "output-index": 0
        },
        {
            "output-index": 1
        }
    ]
}
2023-08-02 10:10:04,448 [INFO]  Starting transaction monitor in walletservice
2023-08-02 10:10:04,497 [WARNING]  Receiver returned error code: 400, message: { "errorCode": "sender-params-error", "message": "could not parse feerate" }
2023-08-02 10:10:04,497 [WARNING]  Payjoin did not succeed, falling back to non-payjoin payment.
2023-08-02 10:10:04,497 [WARNING]  Error message was: { "errorCode": "sender-params-error", "message": "could not parse feerate" }
2023-08-02 10:10:04,613 [INFO]  Payment made without coinjoin. Transaction:
2023-08-02 10:10:04,615 [INFO]  {
    "hex": "0200000000010182e9375743e3ef976a4fa44a450880bfe98ea981d7db7e23d3d7dcf1016066870100000000fdffffff02e3e80e0000000000160014c45edd801ca3a8f86459a5c2e9292c943597252210270000000000001600148a55317fc6002262cc07ed0dd69b3d425523a5b702483045022100da17abc3ad4337312e9101f50a8a0cbcf5e926b4c86439a666ca0e09d5981c37022067641f7337d54a8057e8dcd240e326aafa44ac3c8547726c0b9b856d49a856fb01210236a55a20c8e577b8074eff6d14e0b9347f5a6fe4f28e60d4f34b7c6fdb625523176f0400",
    "inputs": [
        {
            "outpoint": "87666001f1dcd7d3237edbd781a98ee9bf8008454aa44f6a97efe3435737e982:1",
            "scriptSig": "",
            "nSequence": 4294967293,
            "witness": "02483045022100da17abc3ad4337312e9101f50a8a0cbcf5e926b4c86439a666ca0e09d5981c37022067641f7337d54a8057e8dcd240e326aafa44ac3c8547726c0b9b856d49a856fb01210236a55a20c8e577b8074eff6d14e0b9347f5a6fe4f28e60d4f34b7c6fdb625523"
        }
    ],
    "outputs": [
        {
            "value_sats": 977123,
            "scriptPubKey": "0014c45edd801ca3a8f86459a5c2e9292c9435972522",
            "address": "tb1qc30dmqqu5w50seze5hpwj2fvjs6ewffzht8p6j"
        },
        {
            "value_sats": 10000,
            "scriptPubKey": "00148a55317fc6002262cc07ed0dd69b3d425523a5b7",
            "address": "tb1q3f2nzl7xqq3x9nq8a5xadxeagf2j8fdh3af6y0"
        }
    ],
    "txid": "0bf3a449c2227c25e861cbb90c239f3051fad0bb08b46e98cb09efa58e2d4f46",
    "nLockTime": 290583,
    "nVersion": 2
}
done
(jmvenv) tee@mutiny:/tmp/joinmarket-clientserver-0.9.9/scripts$

~~~

## 3. Back to payjoin screen the output come
~~~
dan@mutiny:~/rust-payjoin/payjoin-cli$ RUST_LOG=debug cargo run  --  receive 10000
    Finished dev [unoptimized + debuginfo] target(s) in 0.22s
     Running `/home/dan/rust-payjoin/target/debug/payjoin receive 10000`
[2023-08-02T09:28:11Z DEBUG bitcoincore_rpc] JSON-RPC request: getnewaddress [null,null]
Listening at 0.0.0.0:3000. Configured to accept payjoin at BIP 21 Payjoin Uri:
BITCOIN:TB1Q3F2NZL7XQQ3X9NQ8A5XADXEAGF2J8FDH3AF6Y0?amount=0.0001&pj=https://localhost:3010
[2023-08-02T09:28:12Z DEBUG tiny_http] Server listening on 0.0.0.0:3000
[2023-08-02T09:28:12Z DEBUG tiny_http] Running accept thread



[2023-08-02T10:10:04Z DEBUG payjoin::app] Received request: Request { method: "POST", url: "?v=1&disableoutputsubstitution=false&additionalfeeoutputindex=0&maxadditionalfeecontribution=803&minfeerate=1.1", headers: [("Connection", "close"), ("Content-Length", "360"), ("Content-Type", "text/plain"), ("Host", "127.0.0.1:3000")], https: false, remote_addr: Some(127.0.0.1:48244) }
[2023-08-02T10:10:04Z DEBUG payjoin::receive] Received original psbt: PartiallySignedTransaction { unsigned_tx: Transaction { version: 2, lock_time: Blocks(Height(290583)), input: [TxIn { previous_output: OutPoint { txid: 0x87666001f1dcd7d3237edbd781a98ee9bf8008454aa44f6a97efe3435737e982, vout: 1 }, script_sig: Script(), sequence: Sequence(4294967293), witness: Witness { content: [], witness_elements: 0, indices_start: 0 } }], output: [TxOut { value: 977123, script_pubkey: Script(OP_0 OP_PUSHBYTES_20 c45edd801ca3a8f86459a5c2e9292c9435972522) }, TxOut { value: 10000, script_pubkey: Script(OP_0 OP_PUSHBYTES_20 8a55317fc6002262cc07ed0dd69b3d425523a5b7) }] }, version: 0, xpub: {}, proprietary: {}, unknown: {}, inputs: [Input { non_witness_utxo: None, witness_utxo: Some(TxOut { value: 988511, script_pubkey: Script(OP_0 OP_PUSHBYTES_20 0dc819d26b2c91d34aee70bd4280425bd29a4330) }), partial_sigs: {}, sighash_type: None, redeem_script: None, witness_script: None, bip32_derivation: {}, final_script_sig: None, final_script_witness: Some(Witness { content: [72, 48, 69, 2, 33, 0, 218, 23, 171, 195, 173, 67, 55, 49, 46, 145, 1, 245, 10, 138, 12, 188, 245, 233, 38, 180, 200, 100, 57, 166, 102, 202, 14, 9, 213, 152, 28, 55, 2, 32, 103, 100, 31, 115, 55, 213, 74, 128, 87, 232, 220, 210, 64, 227, 38, 170, 250, 68, 172, 60, 133, 71, 114, 108, 11, 155, 133, 109, 73, 168, 86, 251, 1, 33, 2, 54, 165, 90, 32, 200, 229, 119, 184, 7, 78, 255, 109, 20, 224, 185, 52, 127, 90, 111, 228, 242, 142, 96, 212, 243, 75, 124, 111, 219, 98, 85, 35, 0, 0, 0, 0, 73, 0, 0, 0], witness_elements: 2, indices_start: 107 }), ripemd160_preimages: {}, sha256_preimages: {}, hash160_preimages: {}, hash256_preimages: {}, tap_key_sig: None, tap_script_sigs: {}, tap_scripts: {}, tap_key_origins: {}, tap_internal_key: None, tap_merkle_root: None, proprietary: {}, unknown: {} }], outputs: [Output { redeem_script: None, witness_script: None, bip32_derivation: {}, tap_internal_key: None, tap_tree: None, tap_key_origins: {}, proprietary: {}, unknown: {} }, Output { redeem_script: None, witness_script: None, bip32_derivation: {}, tap_internal_key: None, tap_tree: None, tap_key_origins: {}, proprietary: {}, unknown: {} }] }
[2023-08-02T10:10:04Z ERROR payjoin::app] Error handling request: { "errorCode": "sender-params-error", "message": "could not parse feerate" }

~~~
