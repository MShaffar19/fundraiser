<template>
  <form-struct id="form-donate-btc-01">
    <div slot="title">Donate BTC</div>
    <div slot="subtitle">Send BTC to the address below to claim your Atoms.</div>

    <form-group>
      <div class="donate-btc-key-values">
        <div class="key-value">
          <div class="key">Exchange Rate</div>
          <div class="value">1 BTC : {{ btcExchangeRate }} ATOM</div>
        </div>
        <div class="key-value">
          <div class="key">Min Donation</div>
          <div class="value">{{ config.COINS.BTC.MIN_DONATION }} BTC</div>
        </div>
        <div class="key-value">
          <div class="key">Max Donation</div>
          <div class="value">{{ config.COINS.BTC.MAX_DONATION }} BTC</div>
        </div>
      </div>
      <form-msg body="This address is for your intermediate BTC wallet. You will be asked to confirm your contribution on the next page."></form-msg>
    </form-group>

    <form-group>
      <label for="donate-btc-donation-address">Donation Address</label>
      <field-group>
        <field
          id="donate-btc-donation-address"
          type="textarea"
          v-model="btcAddress">
        </field>
      </field-group>
      <btn-group>
        <btn-copy :value="btcAddress"></btn-copy>
        <btn
          @click.native="qrCodeToggle(true)"
          icon="qrcode"
          value="QR Code">
        </btn>
        <btn
          @click.native="donateBitcoin"
          icon="btc"
          value="Open Wallet">
        </btn>
      </btn-group>
    </form-group>

    <form-group id="donate-btc-loading">
      <div>
        <div class="container">
          <i class="fa fa-circle-o-notch fa-spin"></i>
          <span>Waiting for donation&hellip;</span>
        </div>
      </div>
    </form-group>

    <modal v-if="qrCodeVisible" id="donate-btc-qr">
      <div slot="title">QR Code</div>
      <div>
        <img
          alt="Donation QR Code"
          :src="qrcode">
        </img>
      </div>
      <div slot="footer">
        <btn value="Return to Page" @click.native="qrCodeToggle(false)"></btn>
      </div>
    </modal>

  </form-struct>
</template>

<script>
import { bitcoin } from 'cosmos-fundraiser'
import { mapGetters } from 'vuex'
import num from '../scripts/num.js'
import qr from 'qr-image'
import Btn from '@nylira/vue-button'
import BtnCopy from './BtnCopy'
import BtnGroup from './BtnGroup'
import Field from '@nylira/vue-input'
import FieldGroup from './FieldGroup'
import FormGroup from './FormGroup'
import FormMsg from '@nylira/vue-form-msg'
import FormStruct from './FormStruct'
import Modal from './Modal'
export default {
  name: 'donate-btc-01',
  components: {
    Btn,
    BtnCopy,
    BtnGroup,
    Field,
    FieldGroup,
    FormGroup,
    FormMsg,
    FormStruct,
    Modal
  },
  computed: {
    btcExchangeRate () {
      return num.pretty(this.config.COINS.BTC.EXCHANGE_RATE)
    },
    btcAddress () {
      return this.donation.wallet.addresses.bitcoin
    },
    qrcode () {
      let data = qr.imageSync(this.btcAddress, { margin: 0 })
      let base64 = Buffer(data).toString('base64')
      return `data:image/png;base64,${base64}`
    },
    ...mapGetters(['donation', 'config'])
  },
  data: () => ({
    qrCodeVisible: false,
    transactionInterval: null
  }),
  methods: {
    donateBitcoin () {
      window.location.href =
        `bitcoin:${this.btcAddress}?label=My%20Cosmos%20Fundraiser%20wallet`
    },
    qrCodeToggle (value) {
      this.qrCodeVisible = value
    }
  },
  mounted () {
    document.body.scrollTop = document.documentElement.scrollTop = 0

    let el = document.querySelector('#donate-btc-donation-address')
    el.addEventListener('focus', function () {
      el.select()
    })

    console.log('waiting for tx to ' + this.btcAddress)
    this.transactionInterval = bitcoin.waitForPayment(this.btcAddress, (err, inputs) => {
      if (err) {
        console.error(err)
        return this.$store.commit('notifyError', {
          title: 'Bitcoin Error',
          body: 'An error occurred when trying to get Bitcoin transaction data.'
        })
      }
      console.log('got inputs:', inputs)
      this.$store.commit('setBtcDonationTx', inputs)
      this.$store.commit('setDonationProgress', 4)
    })
  },
  beforeDestroy () {
    if (!this.transactionInterval) return
    console.log('DonateBtc01 beforeDestroy, clearing poll interval')
    clearInterval(this.transactionInterval)
    this.transactionInterval = null
  }
}
</script>

<style lang="stylus">
@import '../styles/variables.styl'

#form-donate-btc-01
  .ni-form-main
    border-bottom none
  .ni-form-footer
    display none

#donate-btc-donation-address
  height 3.5rem !important
  mono()

.donate-btc-key-values
  margin-bottom 1rem
  .key-value
    padding 0.125rem 0
    display flex
    align-items center
    font-size 0.875rem
    .key
      flex 2
      color dim
    .value
      flex 3
      font-weight 500

#donate-btc-loading
  border-bottom none
  > div
    border-bottom none
    background lighten(mcolor, 95%)
    border-radius 0.25rem
    margin 0.5rem 0
    padding 0.75rem 1rem
    text-align center
    i.fa
      margin-right 0.25rem
      color mcolor
    span
      font-weight 400
      color txt

#donate-btc-qr
  img
    width 66.666vw
    max-width 20rem
    display block
    margin 0 auto
  .ni-modal-footer > div
    justify-content center
</style>
