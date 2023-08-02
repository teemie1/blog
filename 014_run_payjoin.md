# Pay Bitcoin Onchain with Payjoin for Privacy

## For receiver, run payjoin receiver services
~~~
dan@mutiny:~/rust-payjoin/payjoin-cli$ RUST_LOG=debug cargo run  -- -r "http://127.0.0.1:38332/wallet/taproot" receive 10000
    Finished dev [unoptimized + debuginfo] target(s) in 0.25s
     Running `/home/dan/rust-payjoin/target/debug/payjoin -r 'http://127.0.0.1:38332/wallet/taproot' receive 10000`
[2023-08-02T02:41:27Z DEBUG bitcoincore_rpc] JSON-RPC request: getnewaddress [null,null]
Listening at 0.0.0.0:3000. Configured to accept payjoin at BIP 21 Payjoin Uri:
BITCOIN:TB1QJCR6W4J9ENS5G8RJWJ6HRK93A8D3SJJ4ZQGUPK?amount=0.0001&pj=https://localhost:3010
[2023-08-02T02:41:28Z DEBUG tiny_http] Server listening on 0.0.0.0:3000
[2023-08-02T02:41:28Z DEBUG tiny_http] Running accept thread

~~~

## For sender, check balance in joinmarket
~~~
tee@mutiny:/tmp/joinmarket-clientserver-0.9.9$ source jmvenv/bin/activate
(jmvenv) tee@mutiny:/tmp/joinmarket-clientserver-0.9.9$ cd scripts/
(jmvenv) tee@mutiny:/tmp/joinmarket-clientserver-0.9.9/scripts$ python wallet-tool.py wallet.jmdat
User data location: /home/tee/.joinmarket/
Enter passphrase to decrypt wallet:
JM wallet
mixdepth        0       tpubDDPr3UDE1iLeg6koucT2hvcjBe7EmY8sWkhqicgjEJUedrWf3w9SdQiEv37ZJQPKahXxFJvtV8nhqt11VAP1SAv4uBhJiyLf2o7R3pNbUn5
external addresses      m/84'/1'/0'/0   tpubDEU8dDKPssyF3cjBtQ8QDkgBZRPtuLdSweayTtCXNuJirkiUcjwgDi8gyugC5Bjxwp62SSvUN9rd8msjgbMtbCYKqYxpMf1EL69mYYg2MuY
m/84'/1'/0'/0/0         tb1qztyfxm6vwwn888aveappzajxxdgn46ul2clw3a      0.01000000      deposit
m/84'/1'/0'/0/2         tb1q73yggtfr2nrvtn94vqtnec9wq2fqnjaxc3jw2y      0.00000000      new
m/84'/1'/0'/0/3         tb1ql5nxwqt9wucrrk2ngg6efd4qjmznx0z04h6xm6      0.00000000      new
m/84'/1'/0'/0/4         tb1q0hka49lvyqve8huwqkp8g4cvnsj66a9r235uek      0.00000000      new
m/84'/1'/0'/0/5         tb1q6jyyrwuhzmr3rmupfwqgkl08pltwlef077q7lj      0.00000000      new
m/84'/1'/0'/0/6         tb1qcyddvzx2zsfh5ralhj0cehjaf7kslkpamhuxp2      0.00000000      new
m/84'/1'/0'/0/7         tb1qrq6s8ea5xh2d7peyxvh64kr3l6ttjzyesl7gdt      0.00000000      new
Balance:        0.01000000
internal addresses      m/84'/1'/0'/1
m/84'/1'/0'/1/1         tb1q4gekt3vpyex6h8c8t8l7kja2l3jv5qd4ps249c      0.00270986      non-cj-change
Balance:        0.00270986
Balance for mixdepth 0: 0.01270986
mixdepth        1       tpubDDPr3UDE1iLegv8N9Bb6K4X2tpN1tSuUmsgL4Ecdmyd5LY8eDB3GQ27z5iuSHwGLCWqcZFpcxXZnatSa4eKG3VXhZePYDgSPnhFVUQ6c3mN
external addresses      m/84'/1'/1'/0   tpubDEi5TnXGqa8GogvZeDMwC2o2UkTgpUR2ZANEwvw3u9mo7PDgBuB5Vansq2ayyidVPUajNgEvJ3yokRLhr4Sq42JsrsbhBmfEeEHisyT6tpW
m/84'/1'/1'/0/0         tb1qxx4nvp96x6ldl8xnwl848n560xv3u5redv8uz2      0.00000000      new
m/84'/1'/1'/0/1         tb1qungv9n48yzl4hv95n5868g2prqd0nk2d2vw83m      0.00000000      new
m/84'/1'/1'/0/2         tb1qq5mqxvchgh4g7spuqg32l46ml04zxqqxywn9sj      0.00000000      new
m/84'/1'/1'/0/3         tb1qhvp46wnfsar4mu5cgq00h458skaazf0mfghqn6      0.00000000      new
m/84'/1'/1'/0/4         tb1qmk273qfq4jpvmlz8j75ys89654k37rpvsjsufv      0.00000000      new
m/84'/1'/1'/0/5         tb1qf29yet30ae37puuc0t5zaq8xhy98jcvs96auyw      0.00000000      new
Balance:        0.00000000
internal addresses      m/84'/1'/1'/1
Balance:        0.00000000
Balance for mixdepth 1: 0.00000000
mixdepth        2       tpubDDPr3UDE1iLekVDhGyfrnaGXx9WzYVgcPZ7H8ccLDAazziwx94m9yB67Br6w769gXmDBJE3Q7qcz9BmT7iMy3BsTnht3GpfzMj9kyrdspSL
external addresses      m/84'/1'/2'/0   tpubDFKdf5p3a7A4xCQi6mjVPxQogjSM2gZgjCNHE21dnoGCPns6V8FugS5oP2bng9kRLRST6LKadzkvBbPHHv8KQGCX5mmPVsjhDxRcBUiCAHY
m/84'/1'/2'/0/0         tb1qy573d5qtr5prt44yx8fh3y925260cehpyksvuj      0.00000000      new
m/84'/1'/2'/0/1         tb1qkw6s3muc0f096sr4g8cq76pjttvfdrgqj4gxrj      0.00000000      new
m/84'/1'/2'/0/2         tb1qvmmslzjc98uk320fq8ek9fdgfnkpg2kc9072sr      0.00000000      new
m/84'/1'/2'/0/3         tb1qdm2hnknr4dms9rja3vct4cveza9pquv9zwak92      0.00000000      new
m/84'/1'/2'/0/4         tb1q9gfyulylpc2au8lkdu9rffmlrqg8u8w45p6r3u      0.00000000      new
m/84'/1'/2'/0/5         tb1qs63u63v38nl0zkrdhp3gs9g2rrxtrywzwl5tck      0.00000000      new
Balance:        0.00000000
internal addresses      m/84'/1'/2'/1
Balance:        0.00000000
Balance for mixdepth 2: 0.00000000
mixdepth        3       tpubDDPr3UDE1iLep2y23kTukfN7YxmTFrUPvFUAGHUrYc4TDMoMGpGoX6GDemDqjKzPR5Zx55b9PJzMRJ6jxZuJmHjaSRmtP6oibhoR7BNwwUL
external addresses      m/84'/1'/3'/0   tpubDEqFHAcXNidomhqhiD7HEZqWZXJUTGjqNM3gc4du73S67yg6L2V7qw5qNHHBSVJg71WBTAb6A11fEx3VUUJj1eGX7HRooY9EjLnfzraGnjw
m/84'/1'/3'/0/0         tb1q5y8jxvdezqpq5szrtqssq69d3kwn3lnvq2qe5a      0.00000000      new
m/84'/1'/3'/0/1         tb1qwdpp4uck9skax5wncaar32yjcye807zgd9ugzm      0.00000000      new
m/84'/1'/3'/0/2         tb1qgav09h9uq5auglc8yxj7t8mxql37gsf2unecj2      0.00000000      new
m/84'/1'/3'/0/3         tb1q9ltwm4pftf0lsvkx5g0s3kpjyvanjjlwndexxs      0.00000000      new
m/84'/1'/3'/0/4         tb1q23ajx8rd3puur36vp7agqdhwqkdwl2pk5az5rh      0.00000000      new
m/84'/1'/3'/0/5         tb1qg8nxfguvjfl84y2ydzm9qruuqvqj4hul0aptxl      0.00000000      new
Balance:        0.00000000
internal addresses      m/84'/1'/3'/1
Balance:        0.00000000
Balance for mixdepth 3: 0.00000000
mixdepth        4       tpubDDPr3UDE1iLepZgMyXWgzhqLNfTmeqitKeT2nGqGFvHsr9Mr6ZdEYMg9tFZTEbY8vLaGAcvNxePo3FpAkvEkaz9XMwW8yqoS9PCpjXqsgQj
external addresses      m/84'/1'/4'/0   tpubDE1ZPqQrMULfxHF93H59Cku9LYTbu5iK6NdfFow3qTLrnmKkdRg6myejbVK64GD4XbHJA5tktDRuhxfJEcAVnfBS8AhDuLanRoXd6yfEMq6
m/84'/1'/4'/0/0         tb1qc6vxgh3pd5xqv7swjfmasaezslyg6jx7d42fzu      0.00000000      new
m/84'/1'/4'/0/1         tb1qgtpj9ydhy57ujj55wnhzsp5ttzc9zflh0szskj      0.00000000      new
m/84'/1'/4'/0/2         tb1qe03wpakgz2tm9k9ukrd725934yyj2llph70wvr      0.00000000      new
m/84'/1'/4'/0/3         tb1q4nar7lms6dga68xvn7aenm3e3fjy8ft8vkfy8p      0.00000000      new
m/84'/1'/4'/0/4         tb1qmnkut8xx9545qe8g3m5ph5svguux6xscay9kx7      0.00000000      new
m/84'/1'/4'/0/5         tb1qpf34y92uy49mtycpd9v77f50luhwah8yxu0y5z      0.00000000      new
Balance:        0.00000000
internal addresses      m/84'/1'/4'/1
Balance:        0.00000000
Balance for mixdepth 4: 0.00000000
Total balance:  0.01270986
~~~
## For sender, try to pay via joinmarket with payjoin
~~~
(jmvenv) tee@mutiny:/tmp/joinmarket-clientserver-0.9.9/scripts$ python sendpayment.py -m 0 wallet.jmdat  "BITCOIN:TB1QJCR6W4J9ENS5G8RJWJ6HRK93A8D3SJJ4ZQGUPK?amount=0.0001&pj=https://mutiny.payjoin.org:4433"
User data location: /home/tee/.joinmarket/
Attempting to pay via payjoin.
2023-08-02 02:46:20,744 [INFO]  starting sendpayment
Enter passphrase to decrypt wallet:
2023-08-02 02:46:29,490 [INFO]  BIP78 daemon listening on port 25183
2023-08-02 02:46:29,493 [WARNING]  Could not source a fee estimate from Core, falling back to default: 10000 sat/vkB (10.0 sat/vB).
2023-08-02 02:46:29,493 [INFO]  Using bitcoin network feerate for 3 block confirmation target (randomized for privacy): 9459 sat/vkB (9.4 sat/vB)
2023-08-02 02:46:29,501 [WARNING]  Could not source a fee estimate from Core, falling back to default: 10000 sat/vkB (10.0 sat/vB).
2023-08-02 02:46:29,501 [INFO]  Using bitcoin network feerate for 3 block confirmation target (randomized for privacy): 9336 sat/vkB (9.3 sat/vB)
2023-08-02 02:46:29,556 [INFO]  Using a fee of: 0.00001316 BTC (1316 sat).
2023-08-02 02:46:29,556 [INFO]  Using a change value of: 0.00259670 BTC (259670 sat).
Completed PSBT created:
{
    "psbt-version": 0,
    "unsigned-tx": {
        "hex": "020000000171c6d089a1904e7a12c81cdd7506db9f17c109e41fdc4449fe7d14fe27de80410100000000fdffffff0210270000000000001600149607a75645cce1441c7274b571d8b1e9db184a5556f6030000000000160014e170436a7f24ae458ed161c678b94aa1d4596892106c0400",
        "inputs": [
            {
                "outpoint": "4180de27fe147dfe4944dc1fe409c1179fdb0675dd1cc8127a4e90a189d0c671:1",
                "scriptSig": "",
                "nSequence": 4294967293
            }
        ],
        "outputs": [
            {
                "value_sats": 10000,
                "scriptPubKey": "00149607a75645cce1441c7274b571d8b1e9db184a55",
                "address": "tb1qjcr6w4j9ens5g8rjwj6hrk93a8d3sjj4zqgupk"
            },
            {
                "value_sats": 259670,
                "scriptPubKey": "0014e170436a7f24ae458ed161c678b94aa1d4596892",
                "address": "tb1qu9cyx6nlyjhytrk3v8r83w225829j6yj0uujq2"
            }
        ],
        "txid": "440b289e3a16acbc5570534e67438fa82971e4282b843a00f7be09c2c7437e8b",
        "nLockTime": 289808,
        "nVersion": 2
    },
    "psbt-inputs": [
        {
            "input-index": 0,
            "utxo": {
                "value_sats": 270986,
                "scriptPubKey": "0014aa3365c581264dab9f0759ffeb4baafc64ca01b5",
                "address": "tb1q4gekt3vpyex6h8c8t8l7kja2l3jv5qd4ps249c"
            },
            "final-scriptSig": "",
            "final-scriptWitness": "0248304502210088585d727cfe6da7105cb3b01e608318d523172dbd12fba7c62b197c286b3c020220174c516f5be54175d837f56ebdea10ae6528f0e00470bf10f43306138871597f012102d13e7245638b0db174e402386bea882a85c506985e3d7e772177137a8e9d9c3a"
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
2023-08-02 02:46:29,583 [INFO]  Starting transaction monitor in walletservice
2023-08-02 02:46:29,809 [WARNING]  Receiver returned error code: 400, message: <html>
<head><title>400 Bad Request</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<hr><center>nginx/1.18.0 (Ubuntu)</center>
</body>
</html>

2023-08-02 02:46:29,809 [WARNING]  Payjoin did not succeed, falling back to non-payjoin payment.
2023-08-02 02:46:29,809 [WARNING]  Error message was: <html>
<head><title>400 Bad Request</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<hr><center>nginx/1.18.0 (Ubuntu)</center>
</body>
</html>

2023-08-02 02:46:30,018 [INFO]  Payment made without coinjoin. Transaction:
2023-08-02 02:46:30,020 [INFO]  {
    "hex": "0200000000010171c6d089a1904e7a12c81cdd7506db9f17c109e41fdc4449fe7d14fe27de80410100000000fdffffff0210270000000000001600149607a75645cce1441c7274b571d8b1e9db184a5556f6030000000000160014e170436a7f24ae458ed161c678b94aa1d45968920248304502210088585d727cfe6da7105cb3b01e608318d523172dbd12fba7c62b197c286b3c020220174c516f5be54175d837f56ebdea10ae6528f0e00470bf10f43306138871597f012102d13e7245638b0db174e402386bea882a85c506985e3d7e772177137a8e9d9c3a106c0400",
    "inputs": [
        {
            "outpoint": "4180de27fe147dfe4944dc1fe409c1179fdb0675dd1cc8127a4e90a189d0c671:1",
            "scriptSig": "",
            "nSequence": 4294967293,
            "witness": "0248304502210088585d727cfe6da7105cb3b01e608318d523172dbd12fba7c62b197c286b3c020220174c516f5be54175d837f56ebdea10ae6528f0e00470bf10f43306138871597f012102d13e7245638b0db174e402386bea882a85c506985e3d7e772177137a8e9d9c3a"
        }
    ],
    "outputs": [
        {
            "value_sats": 10000,
            "scriptPubKey": "00149607a75645cce1441c7274b571d8b1e9db184a55",
            "address": "tb1qjcr6w4j9ens5g8rjwj6hrk93a8d3sjj4zqgupk"
        },
        {
            "value_sats": 259670,
            "scriptPubKey": "0014e170436a7f24ae458ed161c678b94aa1d4596892",
            "address": "tb1qu9cyx6nlyjhytrk3v8r83w225829j6yj0uujq2"
        }
    ],
    "txid": "440b289e3a16acbc5570534e67438fa82971e4282b843a00f7be09c2c7437e8b",
    "nLockTime": 289808,
    "nVersion": 2
}
done
~~~

