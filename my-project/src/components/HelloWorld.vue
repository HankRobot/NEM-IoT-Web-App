<template>
  <div class="hello">
    <h1>{{ msg }}</h1>

    <h2>Controlling Multiple IoT Devices using NEM Catapult</h2>
    
    <table class="center">
      <tr>
        <td>LED1</td>
        <td>
          <input id="led1" type="text" name="FirstName" value="1">
        </td>
        <td>LED2</td>
        <td>
          <input id="led2" type="text" name="FirstName" value="1">
        </td>
      </tr>
    </table>
    <br/>
    <button @click="aggregate">Send</button>

    
  </div>
</template>

<script>
import {Account,
        Deadline, 
        NetworkType, 
        TransferTransaction,
        AggregateTransaction,
        TransactionHttp,
        PlainMessage,
        Address,
        Mosaic,
        UInt64,
        MosaicId} from 'nem2-sdk';

import request from 'request'

const node = 'http://103.3.60.174:3000';

export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: "Welcome to Hank's NEM IoT App"
    }
  },
  methods:{
    checkvalidity(hashstring){
      var url = node + '/transaction/' + hashstring + '/status';
      console.log(url);
      request(url, function (error, response, body) {
        const user = JSON.parse(body);
        if ( (response && response.statusCode) == 200 || user["status"] == "Success") {
          console.log("Transaction Success!");
        }
        else{
          console.log("Transaction failed");
        }
      });
    },

    aggregate(){
      /* start block 01 */
      const transactionHttp = new TransactionHttp(node);

      const privateKey = "5D7E87DF82EB6986646B7DBB38CE792117BD106D5179ADC2E1CE0DDE8DBAEE78"; //Hank Bot
      const account = Account.createFromPrivateKey(privateKey, NetworkType.MIJIN_TEST);

      const brotherAddress = 'SCQVB7TIAWI7OJFGRRRU57UPJ4BFNHORCE2JTV2J';  //Anthony Bot
      const brotherAccount = Address.createFromRawAddress(brotherAddress); 

      const sisterAddress = 'SC7APJ3C6BWK3DWVMNUJRQXETYMIC6Y2OG557QHU';  //HuiLum Bot
      const sisterAccount = Address.createFromRawAddress(sisterAddress);

      const mosaicId = "77a1969932d987d7";  
      const mosaicamount = 1;
      const amount = [new Mosaic(new MosaicId(mosaicId), UInt64.fromUint(mosaicamount))]; // 10 cat.currency represent 10 000 000 micro cat.currency

      const led1 = document.getElementById("led1").value;
      const led2 = document.getElementById("led2").value;
      
      console.log(led1);
      console.log(led2);

      const brotherTransferTransaction = TransferTransaction.create(Deadline.create(), brotherAccount, amount, PlainMessage.create(led1), NetworkType.MIJIN_TEST);
      const sisterTransferTransaction = TransferTransaction.create(Deadline.create(), sisterAccount, amount, PlainMessage.create(led2), NetworkType.MIJIN_TEST);

      const aggregateTransaction = AggregateTransaction.createComplete(
        Deadline.create(),
        [brotherTransferTransaction.toAggregate(account.publicAccount),
        sisterTransferTransaction.toAggregate(account.publicAccount)],
        NetworkType.MIJIN_TEST,
        []
      );
      aggregateTransaction.maxFee = UInt64.fromUint(parseInt(aggregateTransaction.serialize().substring(0,4),16)*100);
      console.log(parseInt(aggregateTransaction.serialize().substring(0,4),16)*100);
      /* end block 01 */

      /* start block 02 */
      const networkGenerationHash = "9F1979BEBA29C47E59B40393ABB516801A353CFC0C18BC241FEDE41939C907E7";
      const signedTransaction = account.sign(aggregateTransaction, networkGenerationHash);
      console.log(signedTransaction.hash);

      transactionHttp.announce(signedTransaction);

      var self = this;
      setTimeout(function(){
        self.checkvalidity(signedTransaction.hash.toString());
      },2000);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.center {
  margin: auto;
  width: 60%;
  border: 3px solid #73AD21;
  padding: 10px;
}
</style>
