<template>
  <div class="modal-container">
    <b-modal
      ref="confirmation"
      hide-footer
      centered
      class="bootstrap-modal-wide ens-modal nopadding"
      title="Confirm Bid">
      <div class="modal-content">
        <div class="addresses-container">
          <address-block
            :address="from"
            direction="From"/>
          <div
            v-show="to !== '' && to !== undefined"
            class="direction">
            <img src="~@/assets/images/icons/right-arrow.svg">
          </div>
          <address-block
            v-show="to !== '' && to !== undefined"
            :address="to"
            direction="To"/>
        </div>
        <div class="ens-name">
          <h2>{{ ens.name }}.eth</h2>
        </div>
        <div class="detail-info">
          <div class="detail-info-item">
            <span class="detail-title">Actual Bid Amount</span>
            <span class="detail-value">{{ ens.bidAmount }} ETH</span>
          </div>
          <div class="detail-info-item">
            <span class="detail-title">Secret Phrase</span>
            <span class="detail-value">{{ ens.secretPhrase }}</span>
          </div>
          <div class="detail-info-item">
            <span class="detail-title">Reveal Date</span>
            <span class="detail-value">{{ formatDate(ens.revealDate) }}</span>
          </div>
          <div class="detail-info-item">
            <span class="detail-title">Bid Mask</span>
            <span class="detail-value">{{ ens.bidMask }} ETH</span>
          </div>
          <div class="detail-info-item">
            <span class="detail-title">Auction Ends</span>
            <span class="detail-value">{{ formatDate(ens.auctionDateEnd) }}</span>
          </div>
          <div>
            <div class="json-container">
              <span class="json-title">JSON String</span>
              <span
                class="json-copy"
                @click="copyString">{{ $t('common.copy') }}</span>
            </div>
            <textarea
              ref="json"
              v-model="jsonText"
              class="json-content" />
          </div>
        </div>
        <div class="button-container">
          <button
            type="button"
            class="cancel-button"
            name="button"
            @click="close">{{ $t('common.cancel') }}</button>
          <button
            v-if="isHardwareWallet"
            :class="[signedTx !== ''? 'submit-button': 'disabled']"
            type="button"
            name="button"
            @click="sendTx"> Confirm with HW wallet </button>
          <button
            v-else-if="$store.state.wallet && $store.state.wallet.brand === 'Web3'"
            type="button"
            class="submit-button"
            name="button"
            @click="close">Proceed to your Web3 wallet provider.</button>
          <button
            v-else
            type="button"
            class="submit-button"
            name="button"
            @click="sendTx">{{ $t('common.confirmAndSave') }}</button>
        </div>
      </div>
    </b-modal>
  </div>
</template>

<script>
// eslint-disable-next-line
const unit = require('ethjs-unit');
import AddressBlock from '../AddressBlock';
import { Misc } from '@/helpers';
export default {
  components: {
    'address-block': AddressBlock
  },
  props: {
    confirmSendTx: {
      type: Function,
      default: function() {}
    },
    fee: {
      type: Number,
      default: 0
    },
    signedTx: {
      type: String,
      default: ''
    },
    data: {
      type: String,
      default: ''
    },
    from: {
      type: String,
      default: ''
    },
    gas: {
      type: Number,
      default: 0
    },
    gasPrice: {
      type: Number,
      default: 0
    },
    nonce: {
      type: Number,
      default: 0
    },
    to: {
      type: String,
      default: ''
    },
    value: {
      type: Number,
      default: 0
    },
    isHardwareWallet: {
      type: Boolean,
      default: false
    },
    ens: {
      type: Object,
      default: function() {
        return {};
      }
    }
  },
  data() {
    return {
      jsonText: '',
      formatDate: Misc.formatDate
    };
  },
  watch: {
    ens(val) {
      this.jsonText = JSON.stringify({
        name: val.name,
        nameSHA3: val.nameSHA3,
        bidAmount: val.bidAmount,
        bidMask: val.bidMask,
        value: this.$store.state.web3.utils.toWei(
          val.bidAmount.toString(),
          'ether'
        ),
        secretPhrase: val.secretPhrase,
        secretPhraseSHA3: val.secretPhraseSHA3
      });
    }
  },
  methods: {
    sendTx() {
      if (this.signedTx !== '') {
        this.confirmSendTx();
      }
    },
    copyString() {
      this.$refs['json'].select();
      document.execCommand('copy');
      window.getSelection().removeAllRanges();
    },
    close() {
      this.$refs.confirmation.hide();
    }
  }
};
</script>

<style lang="scss" scoped>
@import 'EnsConfirmModal.scss';
</style>
