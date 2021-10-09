<template>
<div class="content">
  <div >
  <div class="mnemonic">
    <span>助记词：</span>
    <textarea  class="mnemoniccontent" type="textarea" id="mnemonic" v-model="mnemonic" >
    </textarea>
  </div>

  <div class="mnemonic">
    <span>私钥：</span>
    <textarea class="mnemoniccontent" type="textarea" id="privateKey" placeholder="291c09fc38eb9a253ee9e2d8e90c91b4f08cd87aa3d9bf707d5fd54a3b4aa442" v-model="privateKey" >
    </textarea>
  </div>
</div>
  <div>
    <select class="select" v-model="selected" name="fruit" @change="selectFn($event)">
      <option value="">选择网络</option>
      <option v-for="item in networks" :key="item.networkId" :value="item.id">{{item.name}}</option>
    </select>
  </div>
  <div>
    <h4>address： {{account}}</h4>
    <h4>NONCE： {{nonce}}</h4>
    <h4>Balance: {{balance}} ether</h4>
  </div>
  <div>
    <div>Nonce    : <input id="nonce" v-model="nonce" ></div>
    <div>Gas Price: <input id="gasPrice" v-model="gasPrice" placeholder='GWEI'></div>
    <div>Gas Limit: <input id="gas" v-model="gasLimit" placeholder='gas limit'></div>
    <div>Pay      : <input id="pay" v-model="payETH" placeholder='(ether)'></div>
    <div>To       : <input id="to" v-model="to" placeholder='address'></div>
    <div>Data     : <input id="data" v-model="sendData" placeholder='0x00'></div>
  </div>
  <div><button :disabled="dis" @click="signedTransaction">signedTransaction</button></div>
  <div>SignData: <input id="data" v-model="signData"></div>
  <div><button :disabled="dis" @click="sendSignedTransaction">sendSignedTransaction</button></div>
  <!-- <div><button @click="enableEthereum">Enable Ethereum</button></div> -->
</div>
</template>

<script>
const Web3 = require('web3')
const bip39 = require('bip39')
const Tx = require('ethereumjs-tx').Transaction;
const Common=require('ethereumjs-common').default
const {hdkey} = require('ethereumjs-wallet')
const util = require('ethereumjs-util')
// const web3js = window.web3
// let mnemonic = "depart expose  labor robust  raven shaft toss curious nephew just end stairs"

// let web3 = new Web3(new Web3.providers.HttpProvider('https://ropsten.infura.io/v3/d3d7efacf489482785f091bbf019f2a9'));

// let web3 = new Web3(web3js.currentProvider);
export default {
  name: 'CrowdFund',
  data() {
    return {
      mnemonic: 'script desert mix cabbage drastic inner paper dinner replace innocent material cheap',
      privateKey: '',
      networks: [
        {id: 0, name:'Main Net',networkId:'1',chainId:'1', label: 'mainnet'},
        {id: 1, name:'Ropsten test network',networkId:'3',chainId:'3',label: 'ropsten'},
        {id: 2, name:'Rinkeby test network',networkId:'4',chainId:'4',label: 'rinkeby'},
        {id: 3, name:'Goerli test network',networkId:'5',chainId:'5',label: 'goerli'},
        {id: 4, name:'Kovan test network',networkId:'42',chainId:'42',label: 'kovan'},
        ],
      myWeb3: null,
      chainId: null,
      chainLabel: null,
      selected: '',
      dis: true,
      account: null,
      balance: null,
      nonce: null,
      gasPrice: null,
      gasLimit: null,
      payETH: null,
      to: null,
      sendData: null,
      signData: null
    }
  },


  methods: {
    enableEthereum(){
      console.log('click button');
      window.ethereum.request({ method: 'eth_requestAccounts' });
      this.getAccount();
    },
    async selectFn(e) {
      let index = e.target.value
      if(index === ""){
        console.log("未选择网络")
      }else{
        let item = this.networks[index]
        this.chainLabel = item.label
        this.chainId = Number(item.chainId)
        console.log(item.label)
        let rpc = 'https://'+this.chainLabel+'.infura.io/v3/d3d7efacf489482785f091bbf019f2a9'
        console.log(rpc)
        this.myWeb3 = new Web3(new Web3.providers.HttpProvider(rpc));
        let web3 = this.myWeb3
        // const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' });
        if(this.mnemonic === null){
          console.log("请填入助记词！")
          return
        }
        console.log(this.privateKey)
        if(this.mnemonic === 'script desert mix cabbage drastic inner paper dinner replace innocent material cheap' && this.privateKey === ""){
          console.log("请填入私钥！")
        }
        let address =""
        if(this.privateKey !== "" && this.privateKey.length == 64){
          let publicKey = util.privateToPublic(new Buffer(this.privateKey, 'hex'));
          address ="0x"+ util.publicToAddress(publicKey).toString('hex');
          console.log(address)
        }else{
          let seed = await bip39.mnemonicToSeed(this.mnemonic)
          console.log(this.mnemonic)
          let hdWallet = await hdkey.fromMasterSeed(seed)
          let key1 = await hdWallet.derivePath("m/44'/60'/0'/0/0")

          let addr = await util.pubToAddress(key1._hdkey._publicKey, true)
          address = '0x'+addr.toString('hex');

        }
          this.account = address;
          web3.eth.getTransactionCount(this.account).then(count=>{
            this.nonce = count
            console.log(count)
            })
          web3.eth.getBalance(this.account).then(balance=>{
            this.balance = (balance/1e18).toFixed(4)
            console.log(balance)
            this.dis = false
            })

      }


    },
    async signedTransaction(){
      let web3 = this.myWeb3
      const customCommon = await Common.forCustomChain(
        this.chainLabel,
        {
          name: this.chainLabel,
          networkId: this.chainId,
          chainId: this.chainId,
        },
        'petersburg',
      )

      console.log(customCommon)

      let seed = await bip39.mnemonicToSeed(this.mnemonic)
      
      let hdWallet = await hdkey.fromMasterSeed(seed)
      let key1 = await hdWallet.derivePath("m/44'/60'/0'/0/0")
      let privateKey1 = util.bufferToHex(key1._hdkey._privateKey)
      if(this.privateKey !== "" && this.privateKey.length == 64){
        privateKey1 = "0x" + this.privateKey;
      }
      const privateKey = Buffer.from(
        privateKey1.slice(2),
        'hex',
      )

      console.log('account:', this.account);
      console.log('nonce:', this.nonce);
      console.log('to:', this.to);
      const payETH = web3.utils.toHex(this.payETH*1e18);
      console.log('value:', payETH);
      const gasPrice = web3.utils.toHex(this.gasPrice*1e9);
      console.log('gasPrice:', gasPrice);
      const gasLimit = web3.utils.toHex(this.gasLimit);
      console.log('gas:', gasLimit);
      console.log('data:', this.sendData);

      web3.eth.getTransactionCount(this.account).then(count=>{
        console.log(count)
          var rawTx = {
                nonce: Number(this.nonce),
                from: this.account,
                to: this.to,
                value: payETH,
                gasPrice: gasPrice,
                gas: gasLimit,
                data: this.sendData
            };

          var tx=new Tx(rawTx,{common:customCommon})
          tx.sign(privateKey)
          this.serializedTx = tx.serialize();
          this.signData = '0x' + this.serializedTx.toString('hex')
          console.log('hash：' + web3.utils.sha3(this.serializedTx))
          console.log('sign： ' + '0x' + this.serializedTx.toString('hex'))
      })
    },

    async sendSignedTransaction() {
      let web3 = this.myWeb3
      const fromHexString = hexString => new Uint8Array(hexString.match(/.{1,2}/g).map(byte => parseInt(byte, 16)));
      const serializedTx = fromHexString(this.signData.slice(2))
      console.log('hash：' + web3.utils.sha3(serializedTx))

      web3.eth.sendSignedTransaction(this.signData)
      .on('transactionHash', console.log)
      .on('receipt', console.log)
      .on('error', console.error);
    },
  }
}
</script>



<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
div {
  margin: 16px 8px 0;
  width: 360px;
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
input{
  height: 24px;
  width: 360px;
}
.select{
  margin-top: 20px;
  height: 40px;
}

.mnemonic{
  height: 66px;
  width: 360px;
}
.mnemoniccontent{
    height: 50px;
  width: 360px;
}
</style>