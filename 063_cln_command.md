# Commands for Core Lightning

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
