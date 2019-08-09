<template>
<q-page padding>
  <!-- style='background-image: linear-gradient(white, #00bbff);' -->
  <!-- <div class="row justify-center q-ma-sm">
    <q-btn @click="openCreateCropModal()" v-if="!isRegistered && this.etcBalance > 0" rounded class="bg-indigo-8 text-white text-overline q-ma-sm" label="Create Crop" />
  </div> -->
  <q-page-sticky position='top-right' :offset='[18, 0]' style="z-index: 1">
    <q-btn fab-mini icon='delete' style="cursor: pointer;" class='bg-red-8 text-white' @click="showInsertEncryptionPinDialog=true">
    </q-btn>
  </q-page-sticky>
  <q-dialog v-model="openCreateCropEtcValueToSpentDialog" persistent>
    <q-card>
      <q-card-section>
        <div class="text-h6">Available <span class="text-purple-8">{{typeBalance}}</span> {{getDynBalance}}</div>
        <div class="text-caption">Max Fee <span class="text-red">0.001200011</span> Func</div>
      </q-card-section>
      <q-form
        @submit="onSubmit"
        @reset="onReset"
        class="q-gutter-md"
      >
      <q-card-section v-if="sendingP3C">
        <q-input dense v-model="p3cReceiver" label="Funcoin Address of Receiver" type="text"
        lazy-rules
        :rules="[
          val => val !== null && val !== '' || 'Please type Funcoin address.'
        ]"
      />
      </q-card-section>
      <q-card-section>
        <q-input dense v-model="valueToSpend" label="Amount" type="number" step="0.001" min="0.001" autofocus
        lazy-rules
        :rules="[
          val => val !== null && val !== '' || 'Please type something',
          val => val >= 0.001 || 'Please keep atleast 0.001 as Amount',
          val => val <= this.getDynBalance || 'That much Amount is not available'
        ]"
      />
      </q-card-section>

      <q-card-actions align="right" class="text-primary">
        <q-btn class="text-green" style="cursor: pointer;" flat label="Cancel" v-close-popup />
        <q-btn class='text-white' style="cursor: pointer;" :color="spendEtcBtnColor" :label="btnSpentEtcText" type="submit" />
      </q-card-actions>
      </q-form>
    </q-card>
  </q-dialog>
  <div class='justify-center' align='center'>
    <span class='text-h5 text-white text-weight-bold'>{{user}}</span>
  </div>
<q-pull-to-refresh @refresh="refresh">
  <q-card bordered>
    <!-- <q-card-section v-if='isRegistered'>
      <span class='text-h6 text-weight-light text-green'>P3C Dividends:</span> {{p3cDividends}}
      <q-btn v-if="p3cDividends > 0" class="q-ml-sm text-white bg-green-6" icon="get_app" round @click="withdrawDividendsP3C()"></q-btn>
    </q-card-section>
    <q-card-section v-if='isRegistered'>
      <span class='text-h6 text-weight-light text-green'>P3C Balance:</span> {{p3cBalance}}
    </q-card-section>
      <q-separator inset v-if='isRegistered' />
    <q-card-section v-if='isRegistered'>
      <span class='text-h6 text-weight-light text-green'>IN ETC:</span> {{p3cBalanceInEtc}}
    </q-card-section>
      <q-separator inset v-if='isRegistered' />
    <q-card-section v-if='isRegistered'>
      <span class='text-h6 text-weight-light text-green'>IN USD:</span> ${{p3cBalanceInUsd}}
    </q-card-section>
      <q-separator inset v-if='isRegistered' /> -->
    <q-card-section>
      <span class='text-h6 text-weight-light text-purple-9'>FUNC:</span> {{etcBalance}}
      <q-btn class="q-ml-sm text-white bg-grey-6" style="cursor: pointer;" @click="openExplorer('ETC')" icon="open_in_new" round></q-btn>
    </q-card-section>
    <!-- <q-separator inset v-if='isRegistered' />
    <q-card-section v-if='isRegistered'>
      <span class='text-h6 text-weight-light text-blue-4'>IN USD:</span> ${{p3cBalanceInUsd}}
    </q-card-section> -->
  </q-card>
</q-pull-to-refresh>
  <div class="row justify-center" v-if="etcBalance <= 0 && decryptedData">
    <q-btn rounded style="cursor: pointer;" @click="showWalletViewEtc = true" class="bg-red-9 text-white text-overline q-ma-sm" label="Deposit FUNC" />
  </div>
  <div class="row justify-center" v-if="!decryptedData">
    <q-btn rounded style="cursor: pointer;" to="/" class="bg-red-8 text-white text-overline q-ma-sm" label="Go Back" />
  </div>
  <div v-if='isRegistered'>
  <q-page-sticky position='bottom-left' :offset='[18, 18]'>
    <q-btn fab-mini style="cursor: pointer;" icon='arrow_upward' class='bg-green-8 text-white' @click="showOnlyP3CCropAddress = true" >
      <q-badge color="green-8" class="q-ma-sm">Deposit</q-badge>
    </q-btn>
  </q-page-sticky>
  <q-page-sticky position='bottom-right' :offset='[18, 18]'>
    <q-btn fab-mini style="cursor: pointer;" icon='arrow_downward' class='bg-purple-8 text-white' @click="makeTxs('SEND')">
      <q-badge color="purple-8" class="q-ma-sm">Send</q-badge>
    </q-btn>
  </q-page-sticky>
  </div>
  <q-page-sticky position="bottom" :offset="[18, 18]" v-if='isRegistered'>
    <q-fab
      icon="menu"
      direction="up"
      color="yellow-8 text-white"
    >
    <q-chip color="grey-8" style="height: max-content;">
      <q-fab-action @click="showWalletViewEtc = true" color="white text-yellow-8" icon="account_balance_wallet" />
      <span class="text-overline text-white">Wallet</span>
    </q-chip>
    <q-chip color="grey-8" style="height: max-content;">
      <q-fab-action @click="showHistoryDialog = true" color="white text-yellow-8" icon="history" />
      <span class="text-overline text-white">History</span>
    </q-chip>
    <q-chip color="grey-8" style="height: max-content;">
      <q-fab-action @click="openExplorer('P3C')" color="white text-yellow-8" icon="open_in_new" />
      <span class="text-overline text-white">Explorer</span>
    </q-chip>
    </q-fab>
  </q-page-sticky>
  <!-- Dialog for showing history -->
  <q-dialog
      v-model="showHistoryDialog"
      persistent
      :maximized="true"
      transition-show="slide-up"
      transition-hide="slide-down"
    >
    <q-card class="bg-grey-2 text-green-9">
      <q-btn dense style="cursor: pointer;" flat icon="close" v-close-popup>
        <q-tooltip content-class="bg-white text-primary">Close</q-tooltip>
      </q-btn>
      <q-card-section>
        <div class="text-red-8 bg-grey-2 row justify-center q-mb-sm">
         <span class="text-h6">Sending Transactions</span>
        </div>
    <q-markup-table dark class="bg-green-9">
      <thead>
        <tr>
          <th class="text-left">Type</th>
          <th class="text-right">Value</th>
          <th class="text-right">Token</th>
          <th class="text-left">Hash</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="txs in historyTransactions" :key="txs.hash">
          <td class="text-left">{{txs.type}}</td>
          <td class="text-right">{{txs.etcSpent}}</td>
          <td class="text-right" v-if="txs.inFUNC">FUNC</td>
          <td class="text-right" v-if="!txs.inFUNC">N/A</td>
          <td >
           <q-btn style="cursor: pointer;" round class="text-green-5 bg-grey-1" icon="open_in_new" @click="openHashTxs(txs.hash)">
           </q-btn>
          </td>
        </tr>
      </tbody>
    </q-markup-table>
        </q-card-section>
    </q-card>
  </q-dialog>
 <q-dialog
      v-model="showWalletViewEtc"
      persistent
      maximized
      transition-show="slide-up"
      transition-hide="slide-down"
    >
      <q-card class="bg-white text-blue-9">
        <q-bar class="bg-white">
          <q-space />
          <q-btn style="cursor: pointer;" dense flat icon="close" v-close-popup>
            <q-tooltip content-class="bg-white text-primary">Close</q-tooltip>
          </q-btn>
        </q-bar>
      <div v-if="walletSaved">
        <q-card-section>
          <div class="text-h6 row justify-center">FUNCOIN Wallet
            </div>
        </q-card-section>
        <q-card-section align="center">
          <span class="text-red">Address to share (Please store it yourself safely)</span>
        <q-input v-if="walletSaved.signingKey.address" id="wallet-account" standout @click.native="copyText('wallet-account')" v-model="walletSaved.signingKey.address" readonly>
        <template v-slot:prepend>
          <q-icon name="file_copy" />
        </template>
        </q-input>
        <div class="row justify-center q-ma-sm">
         <q-btn style="cursor: pointer;" class="text-blue-6 bg-white" @click="openExplorer('ETC')" round icon="launch" />
        </div>
        </q-card-section>
        <q-card-section align="center">
          <span class="text-red">Public Key (Please store it yourself safely)</span>
        <q-input v-if="walletSaved" id="wallet-public" standout v-model="walletSaved.signingKey.publicKey" @click.native="copyText('wallet-public')" readonly>
        <template v-slot:prepend>
          <q-icon name="file_copy" />
        </template>
        </q-input>
        </q-card-section>
      <q-card-section align="center">
        <span class="text-red">Private Key (Please store it yourself safely)</span>
        <q-input v-if="walletSaved" id="wallet-private" standout v-model="walletSaved.signingKey.privateKey" @click.native="copyText('wallet-private')" :type="isPwd ? 'password' : 'text'" readonly>
        <template v-slot:prepend>
          <q-icon name="file_copy" />
        </template>
        </q-input>
        </q-card-section>
        <q-card-section align="center">
        <q-btn style="cursor: pointer;" round color="green-9" @click="isPwd = !isPwd">
          <q-icon
            :name="isPwd ? 'visibility_off' : 'visibility'"
            class="cursor-pointer text-white"
          />
          </q-btn>
        </q-card-section>
        <!-- <q-card-section align="center" v-if="isRegistered">
        <span class="text-red">P3C Crop Address (Please store it yourself safely)</span>
        <q-input id="wallet-p3c-a" standout v-model="p3cCropAddress" @click.native="copyText('wallet-p3c-a')" readonly>
        <template v-slot:prepend>
          <q-icon name="file_copy" />
        </template>
        </q-input>
        <div class="row justify-center q-ma-sm">
         <q-btn class="text-green bg-white" @click="openExplorer('P3C')" round icon="launch" />
        </div>
        </q-card-section> -->
        </div>
      </q-card>
    </q-dialog>

 <q-dialog
      v-model="showOnlyP3CCropAddress"
      persistent
      transition-show="slide-up"
      transition-hide="slide-down"
    >
      <q-card class="bg-white text-blue-9">
        <q-bar class="bg-white">
          <q-space />
          <q-btn dense flat style="cursor: pointer;" icon="close" v-close-popup>
            <q-tooltip content-class="bg-white text-primary">Close</q-tooltip>
          </q-btn>
        </q-bar>
        <q-card-section align="center" v-if="isRegistered">
        <span class="text-red">FUNC Address (Send it to the FUNC sender to receive FUNC)</span>
        <q-input v-if="walletSaved" id="wallet-p3c" standout v-model="walletSaved.signingKey.address" @click.native="copyText('wallet-p3c')" readonly>
        <template v-slot:prepend>
          <q-icon name="file_copy" />
        </template>
        </q-input>
        <div class="row justify-center q-ma-sm">
         <q-btn style="cursor: pointer;" class="text-green bg-white" @click="openExplorer('P3C')" round icon="launch" />
        </div>
        </q-card-section>
      </q-card>
    </q-dialog>
<!-- Dialog for deleting account -->
<q-dialog v-model="showInsertEncryptionPinDialog" persistent>
  <q-card style="min-width: 300px">
    <q-card-section>
      <div class="text-h6 text-weight-light"> <q-icon color="red" name="warning" /> Deleting your account is irreversable, You sure you want to continue? Make sure you have saved your private key safe. Then enter your pin and click Delete My Accoun Button</div>
    </q-card-section>

    <q-card-section>
      <q-form
        @submit="onEncryption"
        @reset="onReset"
        class="q-gutter-md"
      >
      <q-input dense
         rounded outlined filled label="Pin"
          v-model="encryptionPin" autofocus
          mask= "#-#-#-#"
      >
      </q-input>
      <q-card-actions align="right" class="text-primary">
        <q-btn style="cursor: pointer;" flat label="Close" v-close-popup />
        <q-btn style="cursor: pointer;" class="bg-primary text-white" label="Encrypt Wallet" type="submit" />
    </q-card-actions>
      </q-form>
    </q-card-section>
  </q-card>
</q-dialog>
</q-page>
</template>

<script>
export default {
  name: 'home',
  data () {
    return {
      dialog: false,
      text: '',
      user: '',
      p3cBalance: '',
      p3cBalanceInEtc: '',
      p3cBalanceInUsd: '',
      etcBalance: '0.000',
      p3cDividends: '',
      valueToSpend: '0.01',
      walletSaved: null,
      privateKey: null,
      walletToGet: null,
      contractWithSigner: null,
      address: null,
      isRegistered: true,
      overrides: null,
      openCreateCropEtcValueToSpentDialog: false,
      farmContractWithSigner: null,
      contractAbiWithSigner: null,
      btnSpentEtcText: 'Create Crop',
      historyTransactions: [],
      spendEtcBtnColor: 'green',
      typeBalance: null,
      getDynBalance: null,
      sendingP3C: false,
      p3cReceiver: '',
      showHistoryDialog: false,
      showWalletViewEtc: false,
      isPwd: true,
      p3cCropAddress: '',
      showOnlyP3CCropAddress: false,
      decryptedData: null,
      showInsertEncryptionPinDialog: false,
      encryptionPin: ''
    }
  },
  mounted () {
    // ETC Wallet
    // console.log(cipher)
    // let temUrl = 'https://explorer.funcoin.io/addr/0x22a9ba777572cefb5126b99670e3d9a5794597f8'
    // let win1 = window.open(temUrl, '_blank')
    // win1.focus()
    // let url = 'https://explorer.funcoin.io/addr/' + this.decryptedData.signingKey.address
    // console.log(url)
    // let win2 = window.open(url, '_blank')
    // win2.focus()
    if (!this.$q.localStorage.getItem('wallet')) {
      this.$router.push('/')
    }
    var bytes = this.$cryptojs.AES.decrypt(this.$q.localStorage.getItem('wallet'), this.$q.sessionStorage.getItem('PinEnr'))
    // console.log(bytes)
    this.$q.loading.hide()
    try {
      this.decryptedData = JSON.parse(bytes.toString(this.$cryptojs.enc.Utf8))
    } catch (err) {
      console.log(err)
      this.$q.loading.hide()
    }
    // console.log(this.decryptedData)
    if (this.decryptedData) {
      this.walletSaved = this.decryptedData
    } else {
      this.$q.notify({
        color: 'red',
        icon: 'warning',
        message: 'Pin is not correct, Kindly go back and enter right pin.'
      })
      this.$q.loading.hide()
      return
    }
    if (this.$q.platform.is.cordova) {
      window.StatusBar.backgroundColorByHexString('#ffffff')
    }
    if (this.$q.localStorage.getItem('historyTrxs')) {
      this.historyTransactions = this.$q.localStorage.getItem('historyTrxs')
      console.log(this.historyTransactions)
    }
    this.$q.loading.show()
    if (!this.$q.localStorage.getItem('overrides') || this.$q.localStorage.getItem('overrides') === null) {
      let wei = this.$ethers.utils.parseEther(this.valueToSpend)
      let feeWei = this.$ethers.utils.parseUnits('1.0', 'gwei')
      this.overrides = {
        // The maximum units of gas for the transaction to use
        gasLimit: 1200011,
        // The price (in wei) per unit of gas
        gasPrice: feeWei._hex,
        // The nonce to use in the transaction
        nonce: 9999,
        // The amount to send with the transaction (i.e. msg.value)
        value: wei,
        // The chain ID (or network ID) to use
        chainId: 6832
      }
      this.$q.localStorage.set('overrides', this.overrides)
      let etherString = this.$ethers.utils.formatEther(this.overrides.value)
      this.valueToSpend = parseFloat(etherString).toFixed(2).toString()
    } else {
      // console.log('else')
      // console.log(this.$q.localStorage.getItem('overrides'))
      let etherString = this.$ethers.utils.formatEther(this.$q.localStorage.getItem('overrides').value)
      this.valueToSpend = parseFloat(etherString).toFixed(2).toString()
      // console.log('elsed')
    }
    this.address = this.walletSaved.signingKey.address
    this.privateKey = this.walletSaved.signingKey.keyPair.privateKey
    this.walletToGet = new this.$ethers.Wallet(this.privateKey, this.$etcProvider)
    // Getting ETC Balance...
    this.getETCBalance()
    // Getting contract...
    // let farmContract = new this.$ethers.Contract(this.$contracts.farm.address, this.$contracts.farm.abi, this.$etcProvider)
    // console.log(farmContract)
    // this.farmContractWithSigner = farmContract.connect(this.walletToGet)
    // this.getMyCrop()
  },
  methods: {
    refresh (done) {
      this.getETCBalance()
      setTimeout(() => {
        done()
      }, 2000)
    },
    // async getMyCrop () {
    //   let cropAddress = await this.farmContractWithSigner.myCrop()
    //   if (cropAddress === '0x0000000000000000000000000000000000000000') {
    //     this.isRegistered = false
    //     this.btnSpentEtcText = 'Create Crop'
    //     this.getETCBalance()
    //   } else {
    //     this.isRegistered = true
    //     this.btnSpentEtcText = 'Buy P3C'
    //     this.$q.localStorage.set('cropAddress', cropAddress)
    //     this.p3cCropAddress = cropAddress
    //     // console.log(cropAddress)
    //     let cropAbi = new this.$ethers.Contract(cropAddress, this.$contracts.crop.abi, this.$etcProvider)
    //     this.cropAbi = cropAbi.connect(this.walletToGet)
    //     this.getcontractInfo()
    //     this.getETCBalance()
    //   }
    //   // console.log(currentValue)
    // },
    // async getcontractInfo () {
    //   // if balance is zero then show a button to buy p3c
    //   let currentValue = await this.farmContractWithSigner.myCropTokens()
    //   let p3cdivs = await this.farmContractWithSigner.myCropDividends()
    //   let realBalance = parseFloat(this.$ethers.utils.formatEther(currentValue))
    //   this.p3cBalance = parseFloat(this.$ethers.utils.formatEther(currentValue)).toFixed(2).toString()
    //   this.p3cDividends = parseFloat(this.$ethers.utils.formatEther(p3cdivs)).toFixed(2).toString()
    //   this.$axios('https://api.p3c.io/chart/info').then((res) => {
    //     this.p3cBalanceInEtc = ((res.data.PriceETC * 0.8) * realBalance).toFixed(3)
    //     this.p3cBalanceInUsd = ((res.data.PriceUSD * 0.8) * realBalance).toFixed(3)
    //   })
    //   // console.log(this.p3cBalance)
    // },
    async getETCBalance () {
      // if balance is zero then show a button to buy p3c
      this.$etcProvider.getBalance(this.address).then((balance) => {
        // balance is a BigNumber (in wei); format is as a sting (in ether)
        let etherString = this.$ethers.utils.formatEther(balance)
        this.etcBalance = parseFloat(etherString).toFixed(4).toString()
        this.$q.loading.hide()
        // console.log(this.etcBalance)
      })
    },
    async sendP3C () {
      if (this.p3cReceiver.trim() === this.walletSaved.signingKey.address) {
        this.$q.notify({
          color: 'warning',
          message: 'Do not send Func to your own address'
        })
      }
      this.$q.loading.show()
      this.openCreateCropEtcValueToSpentDialog = false
      let sendingP3CAmount = this.$ethers.utils.parseEther(this.valueToSpend)
      let feeWei = this.$ethers.utils.parseUnits('1.0', 'gwei')

      let sendfunc = {
        to: this.p3cReceiver.trim(),
        gasLimit: 1200011,
        gasPrice: feeWei._hex,
        value: sendingP3CAmount._hex,
        chainId: 6832
      }
      let sendPromise = this.walletToGet.sendTransaction(sendfunc)
      this.$q.loading.show()
      sendPromise.then((sentfunc) => {
        console.log(sentfunc)
        this.$q.loading.hide()
        if (sentfunc.hash) {
          let value = parseInt(sentfunc.value._hex)
          let etcSpent = this.$ethers.utils.formatEther(value)
          this.historyTransactions.push({ type: 'Sent FUNC', inFUNC: true, etcSpent: etcSpent, hash: sentfunc.hash })
          this.$q.localStorage.set('historyTrxs', this.historyTransactions)
          this.$q.notify({
            message: 'Transaction Successful, Pull to refresh...',
            color: 'green'
          })
        } else {
          this.$q.notify({
            message: 'A problem occured while sending transaction but check blockExplorer First.',
            color: 'warning'
          })
        }
        this.$q.loading.hide()
      })
    },
    openBuyP3CDialog () {
      this.openCreateCropEtcValueToSpentDialog = true
    },
    openCreateCropModal () {
      this.openCreateCropEtcValueToSpentDialog = true
    },
    setOvrride () {
      let feeWei = this.$ethers.utils.parseUnits('1.0', 'gwei')
      let valueWei = this.$ethers.utils.parseEther(this.valueToSpend)
      this.overrides = {
        // The maximum units of gas for the transaction to use
        gasLimit: 1200011,
        // The price (in wei) per unit of gas
        gasPrice: feeWei._hex,
        // The nonce to use in the transaction
        // The amount to send with the transaction (i.e. msg.value)
        value: valueWei._hex,
        // The chain ID (or network ID) to use
        chainId: 6832
      }
      console.log(this.overrides)
      this.$q.localStorage.set('overrides', this.overrides)
    },
    onSubmit () {
      // #=> Creating crop
      console.log('in submit')
      if (this.btnSpentEtcText === 'Create Crop') {
        this.setOvrride()
        this.createCrop()
      } else if (this.btnSpentEtcText === 'Buy P3C') {
        this.setOvrride()
        this.buyP3C()
      } else if (this.btnSpentEtcText === 'Sell P3C') {
        this.setOvrride()
        this.sellP3C()
      } else if (this.btnSpentEtcText === 'Send FUNC') {
        this.setOvrride()
        this.sendP3C()
      }
    },
    onReset () {

    },
    makeTxs (txsType) {
      if (txsType === 'SELL') {
        this.btnSpentEtcText = 'Sell P3C'
        this.spendEtcBtnColor = 'red'
        this.typeBalance = 'P3C'
        this.getDynBalance = this.p3cBalance
        this.sendingP3C = false
        this.openCreateCropEtcValueToSpentDialog = true
      } else if (txsType === 'BUY') {
        this.btnSpentEtcText = 'Buy P3C'
        this.spendEtcBtnColor = 'green'
        this.typeBalance = 'ETC'
        this.getDynBalance = this.etcBalance
        this.sendingP3C = false
        this.openCreateCropEtcValueToSpentDialog = true
      } else if (txsType === 'CREATE') {
        this.btnSpentEtcText = 'Create Crop'
        this.spendEtcBtnColor = 'orange'
        this.typeBalance = 'ETC'
        this.getDynBalance = this.etcBalance
        this.sendingP3C = false
        this.openCreateCropEtcValueToSpentDialog = true
      } else if (txsType === 'SEND') {
        this.btnSpentEtcText = 'Send FUNC'
        this.spendEtcBtnColor = 'blue-6'
        this.typeBalance = 'FUNC'
        this.getDynBalance = this.etcBalance
        this.sendingP3C = true
        this.openCreateCropEtcValueToSpentDialog = true
      }
    },
    openExplorer (type) {
      let url = 'https://explorer.funcoin.io/addr/' + this.decryptedData.signingKey.address
      let win = window.open(url, '_blank')
      win.focus()
      // if (type === 'ETC') {
      //   let url = 'https://blockscout.com/etc/mainnet/address/' + this.decryptedData.signingKey.address
      //   let win = window.open(url, '_blank')
      //   win.focus()
      // } else {
      //   let url = 'https://blockscout.com/etc/mainnet/address/' + this.$q.localStorage.getItem('cropAddress')
      //   let win = window.open(url, '_blank')
      //   win.focus()
      // }
    },
    openHashTxs (txs) {
      let url = 'https://explorer.funcoin.io/tx/' + txs
      let win = window.open(url, '_blank')
      win.focus()
    },
    shareCropAddress () {

    },
    async copyText (txt) {
      let copyTextarea = document.querySelector(`#${txt}`)
      copyTextarea.disabled = false
      copyTextarea.focus()
      copyTextarea.select()
      // console.log(copyTextarea)
      if (copyTextarea.type !== 'text') {
        this.$q.notify({
          color: 'primary',
          message: 'Please make it visible first!'
        })
        return
      }
      var successful = null
      if (this.$q.platform.is.ios || this.$q.platform.is.iphone) {
        // this.$q.notify('Ok you are on ios....')
        var editable = copyTextarea.contentEditable
        var readOnly = copyTextarea.readOnly
        copyTextarea.contentEditable = true
        copyTextarea.readOnly = false
        var range = document.createRange()
        range.selectNodeContents(copyTextarea)
        var selection = window.getSelection()
        selection.removeAllRanges()
        selection.addRange(range)
        copyTextarea.setSelectionRange(0, 999999)
        copyTextarea.contentEditable = editable
        copyTextarea.readOnly = readOnly
        successful = await document.execCommand('copy')
        if (successful) {
          this.$q.notify({
            color: 'green',
            message: 'Value copied successfully'
          })
          return
        } else {
          this.$q.notify({
            color: 'red',
            message: 'Value not copied!'
          })
          return
        }
      }
      // copyTextarea.setAttribute('type', 'text')
      // console.log(copyTextarea)
      copyTextarea.disabled = false
      copyTextarea.focus()
      copyTextarea.select()
      try {
        successful = await document.execCommand('copy')
        // console.log(successful)
        var msg = successful ? 'to clipboard!' : 'unsuccessfully'
        if (successful) {
          this.$q.notify({
            color: 'green',
            message: 'Value copied ' + msg
          })
        } else {
          this.$q.notify({
            color: 'red',
            message: 'Value not copied!'
          })
        }
      } catch (err) {
        alert('Oops, unable to copy')
      } finally {
        copyTextarea.disabled = true
      }
    },
    onEncryption () {
      console.log(this.encryptionPin.length)
      let pin = this.encryptionPin.trim().replace(/-/g, '')
      if (this.$q.sessionStorage.getItem('PinEnr') === pin) {
        this.$q.localStorage.clear()
        this.$q.sessionStorage.clear()
        this.$router.push('/')
      } else {
        this.showInsertEncryptionPinDialog = false
        this.$q.notify({
          color: 'red',
          message: 'Wrong Pin, Do backup your wallet private key manually before deleting it.'
        })
      }
    }
  }
}
</script>

<style>
</style>
