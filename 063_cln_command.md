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

       1: Get the channel id of the first channel.

       CHANNEL_ID=$(echo $(clncli1 listpeerchannels) | jq -r ".channels[0].channel_id")

       2: Get the PSBT from fundpsbt.

       INITIALPSBT=$(echo $(clncli1 fundpsbt -k satoshi=1000000sat feerate=urgent startweight=800 excess_as_change=true) | jq -r ".psbt")

       3: Initiate the splice by passing channel id and initialpsbt received from above steps.

       PSBT_SPLICE_INIT=$(echo $(clncli1 splice_init $CHANNEL_ID 1000000 $INITIALPSBT) | jq -r ".psbt")

       4: Update PSBTs with the splice_update command.

       RESULT={"commitments_secured":false}
       while [[ $(echo $RESULT | jq -r ".commitments_secured") == "false" ]]
       do
         RESULT=$(clncli1 splice_update $CHANNEL_ID $PSBT_SPLICE_INIT)
         PSBT_SPLICE_UPDATE=$(echo $(RESULT) | jq -r ".psbt")
         echo $RESULT
       done

       5: Sign the updated PSBT.

       SIGNPSBT=$(echo $(clncli1 signpsbt -k psbt="$PSBT_SPLICE_UPDATE") | jq -r ".signed_psbt")

       6: Finally, call splice_signed with channel id and signed PSBT parameters.

       $ clncli1 splice_signed $CHANNEL_ID $SIGNPSBT
~~~
# lightning-cli splice_init <channel_id> <amount> [feerate] [force_feerate]
# lightning-cli splice_update <channel_id> <psbt>
# lightning-cli splice_signed <channel_id> <psbt>

Example
lightning-cli splice_init 3d6f313ba4cbf218b19f30f6b99a87fb82ec3657bfaf5e25363149d7c8553dc2 4000000
lightning-cli splice_update 3d6f313ba4cbf218b19f30f6b99a87fb82ec3657bfaf5e25363149d7c8553dc2 

~~~
