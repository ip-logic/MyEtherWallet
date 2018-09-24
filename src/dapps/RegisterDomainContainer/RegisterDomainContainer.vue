<template>
  <div class="register-domain-container">
    <back-button :reset-view="resetView"/>
    <ens-bid
      v-if="uiState === 'nameAvailableAuctionNotStarted'"
      :cancel="cancel"
      :bid-amount="bidAmount"
      :bid-mask="bidMask"
      :secret-phrase="secretPhrase"
      :create-bid="createBid"
      :domain-name="domainName"
      @updateSecretPhrase="updateSecretPhrase"
      @updateBidAmount="updateBidAmount"
      @updateBidMask="updateBidMask"
    />
    <initial-state
      v-if="uiState === 'initial'"
      :domain-buy-button-click="domainBuyButtonClick"
      :check-domain="checkDomain"
      :domain-name="domainName"
      :loading="loading"
      @domainNameChange="updateDomainName"
    />
    <name-forbidden
      v-if="uiState === 'nameIsForbidden'"
      :cancel="cancel"
      domain-name="hellothere!"
    />
    <already-owned
      v-if="uiState === 'nameOwned'"
      :name-hash="nameHash"
      :label-hash="labelHash"
      :owner="owner"
      :resolver-address="resolverAddress"
      :deed-owner="deedOwner"
      :domain-name="domainName"
      :cancel="cancel"
    />
  </div>
</template>

<script>
import BackButton from '@/layouts/InterfaceLayout/components/BackButton';
import EnsBid from './components/EnsBid';
import NameForbidden from './components/NameForbidden';
import InitialState from './components/InitialState';
import AlreadyOwned from './components/AlreadyOwned';
import RegistrarAbi from '@/helpers/registrarAbi';
import { Misc } from '@/helpers';
import bip39 from 'bip39';

export default {
  components: {
    'back-button': BackButton,
    'ens-bid': EnsBid,
    'initial-state': InitialState,
    'name-forbidden': NameForbidden,
    'already-owned': AlreadyOwned
  },
  props: {
    resetView: {
      type: Function,
      default: function() {}
    }
  },
  data() {
    return {
      domainName: '',
      loading: false,
      uiState: 'initial',
      bidAmount: 0.01,
      bidMask: 0.02,
      nameHash: '',
      labelHash: '',
      owner: '',
      resolverAddress: '',
      deedOwner: '',
      secretPhrase: '',
      auctionRegistrarContract: function() {}
    };
  },
  async mounted() {
    const ownerAddress = await this.getRegistrarAddress();
    this.auctionRegistrarContract = new this.$store.state.web3.eth.Contract(
      RegistrarAbi,
      ownerAddress
    );

    console.log(this.auctionRegistrarContract.methods);
  },
  methods: {
    async getRegistrarAddress() {
      const registrarAddress = await this.$store.state.ens.owner('eth');
      return registrarAddress;
    },
    async checkDomain() {
      const web3 = this.$store.state.web3;
      this.loading = true;
      this.labelHash = web3.utils.sha3(this.domainName);

      try {
        const domainStatus = await this.auctionRegistrarContract.methods
          .entries(this.labelHash)
          .call();
        this.processResult(domainStatus);
      } catch (e) {
        console.log(e);
      }
    },
    processResult(res) {
      switch (res[0]) {
        case '0':
          this.generateKeyPhrase();
          this.uiState = 'nameAvailableAuctionNotStarted';
          this.loading = false;
          break;
        case '1':
          this.loading = false;
          console.log('Name is available and the auction has been started');
          break;
        case '2':
          this.getMoreInfo(res[1]);
          break;
        case '3':
          this.loading = false;
          this.uiState = 'nameIsForbidden';
          break;
        case '4':
          this.loading = false;
          console.log('Name is currently in the ‘reveal’ stage of the auction');
          break;
      }
    },
    cancel() {
      this.uiState = 'initial';
      this.clearInputs();
    },
    updateDomainName(value) {
      this.domainName = value;
    },
    async getMoreInfo(deedOwner) {
      let owner;
      let resolverAddress;
      try {
        owner = await this.$store.state.ens.owner(this.domainName + '.eth');
      } catch (e) {
        owner = '0x';
      }

      try {
        resolverAddress = await this.$store.state.ens
          .resolver(this.domainName + '.eth')
          .resolverAddress();
      } catch (e) {
        resolverAddress = '0x';
      }

      this.nameHash = Misc.nameHash(
        this.domainName + '.eth',
        this.$store.state.web3
      );

      this.deedOwner = deedOwner;
      this.owner = owner;
      this.resolverAddress = resolverAddress;
      this.uiState = 'nameOwned';
      this.loading = false;
    },
    async createBid() {
      const address = this.$store.state.wallet.getAddressString();
      const utils = this.$store.state.web3.utils;
      const bidHash = await this.auctionRegistrarContract.methods
        .shaBid(
          utils.sha3(this.domainName),
          address,
          utils.toWei(this.bidAmount.toString(), 'ether'),
          utils.sha3(this.secretPhrase)
        )
        .call();
      const data = this.auctionRegistrarContract.methods
        .startAuctionsAndBid([utils.sha3(this.domainName)], bidHash)
        .encodeABI();

      console.log(data);
    },
    clearInputs() {
      this.loading = false;
      this.uiState = 'initial';
      this.bidAmount = 0.01;
      this.bidMask = 0.02;
      this.nameHash = '';
      this.labelHash = '';
      this.owner = '';
      this.resolverAddress = '';
      this.deedOwner = '';
      this.secretPhrase = '';
      this.domainName = '';
    },
    domainBuyButtonClick() {},
    updateSecretPhrase(e) {
      this.secretPhrase = e;
    },
    updateBidAmount(val) {
      this.bidAmount = val;
    },
    updateBidMask(val) {
      this.bidMask = val;
    },
    generateKeyPhrase() {
      const wordsArray = [];
      const min = 0;
      const max = bip39.wordlists.EN.length;

      for (let i = 0; i < 3; i++) {
        wordsArray.push(
          bip39.wordlists.EN[Math.floor(Math.random() * (max - min + 1)) + min]
        );
      }

      this.secretPhrase = wordsArray.join(' ');
    }
  }
};
</script>

<style lang="scss" scoped>
@import 'RegisterDomainContainer.scss';
</style>
