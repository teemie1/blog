# Commands for Core Lightning & dual funded & splicing

Generate new address for receiving sats
~~~
# lightning-cli newaddr
# lightning-cli newaddr p2tr
~~~

Send sats onchain
~~~
# lightning-cli withdraw [ADDRESS] [AMOUNT]
~~~

List funds
~~~
# lightning-cli listfunds
~~~

List peers
~~~
# lightning-cli listpeers
~~~

Open normal channel
~~~
# lightning-cli -k fundchannel id=[PARTNER NODE ID] amount=[SATOSHI]
~~~

List channel
~~~
# lightning-cli listpeerchannels
# lightning-cli listpeerchannels | jq -r ".channels[0].channel_id"
~~~

Connect new peers
~~~
# lightning-cli connect [NODE ID]@[ADDRESS]:[PORT]
~~~

Generate invoice
~~~
# lightning-cli invoice 1000sat my-first-invoice "Payment for services"
~~~

Pay invoice
~~~
# lightning-cli pay [invoice]
~~~

Generate bolt12 offer
~~~
# lightning-cli offer amount=10000sat description="My awesome product"
~~~

Pay for offer & invoice
~~~
# lightning-cli xpay [invoice or offer bolt12]
~~~

Splicing

~~~
wallet -> 10000sat+fee
10000sat -> 8338aef0

# clncli1 dev-splice "wallet -> 10000sat+fee; 10000sat -> 3d6f313ba4cbf218b19f30f6b99a87fb82ec3657bfaf5e25363149d7c8553dc2"
# clncli2 dev-splice "3d6f313ba4cbf218b19f30f6b99a87fb82ec3657bfaf5e25363149d7c8553dc2 -> 1000000sat"

~~~

Dual Funded Channel
~~~
# vi /data/lightningdm2/config

experimental-dual-fund
funder-policy=match
funder-policy-mod=100
lease-fee-base-sat=500sat
lease-fee-basis=50
channel-fee-max-base-sat=100sat
channel-fee-max-proportional-thousandths=2

# sudo systemctl restart lightningdm2
# clncli2 funderupdate
# clncli1 listnodes | jq '.nodes[] | select(.option_will_fund != null)'
# clncli1 connect 0287a3b8064006ce87621337c07f05152c7c6d0520fb0b1a7585700287b6a40d93@l32cdhwt2qhjm72c547qma6ucqjo7zu366khaxlj4mhsa7hhxj7ytlid.onion:9735
# clncli1 fundchannel id=0287a3b8064006ce87621337c07f05152c7c6d0520fb0b1a7585700287b6a40d93 amount=1000000 request_amt=1000000 compact_lease="024800320002000001f40186a0"
~~~
