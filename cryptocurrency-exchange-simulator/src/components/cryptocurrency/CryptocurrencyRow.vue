import {TransactionStatus} from "@/models/TransactionModel";
<script src="../../models/TransactionModel.ts"></script>
<template>
  <b-container class="cryptoContainer">
    <b-row align-h="center">
      <b-col cols="6" sm="2" md="2" lg="1">Nazwa:</b-col>
      <b-col cols="6" sm="2" md="2" lg="1"><b> {{ cryptocurrencyDetailsModel.cryptocurrency }} </b></b-col>
      <b-col cols="6" sm="2" md="2" lg="1">BID:</b-col>
      <b-col cols="6" sm="2" md="2" lg="1"><b> {{ cryptocurrencyDetailsModel.tickerModel.bid }} </b></b-col>
      <b-col cols="6" sm="2" md="2" lg="1">ASK:</b-col>
      <b-col cols="6" sm="2" md="2" lg="1"><b> {{ cryptocurrencyDetailsModel.tickerModel.ask }} </b></b-col>
      <b-col cols="6" sm="2" md="2" lg="1">Posiadana kwota</b-col>
      <b-col cols="6" sm="2" md="2" lg="1"><b> {{ cryptocurrencyDetailsModel.ownedAmount }} </b></b-col>
      <b-col>
        <b-button cols="6" sm="4" md="4" lg="1" v-b-modal="this.cryptocurrencyDetailsModel.cryptocurrency + 'buy-modal'" variant="success">Kup</b-button>
      </b-col>
      <b-col>
        <b-button cols="6" sm="4" md="4" lg="1" v-b-modal="this.cryptocurrencyDetailsModel.cryptocurrency + 'sell-modal'" variant="danger">Sprzedaj</b-button>
      </b-col>
      <b-col>
        <b-button cols="12" sm="4" md="4" lg="2" v-b-modal="this.cryptocurrencyDetailsModel.cryptocurrency + 'details-modal'" variant="info">Szczegóły</b-button>
      </b-col>
      <b-col>
        <b-button cols="12" sm="4" md="4" lg="2" variant="secondary" v-on:click="this.stopFollowingCryptocurrency">Usuń</b-button>
      </b-col>
    </b-row>

    <b-modal v-bind:id="this.cryptocurrencyDetailsModel.cryptocurrency + 'buy-modal'"
             title="Kupno kryptowaluty"
             @ok="handleBuyCryptocurrency">

      <p>kryptowaluta: {{this.cryptocurrencyDetailsModel.cryptocurrency}}</p>
      <p>cena kupna: {{this.cryptocurrencyDetailsModel.tickerModel.bid}}</p>

      <form ref="form" @submit.stop.prevent="submitBuyingCryptocurrency">
        <b-form-group
          label="Ilość"
          label-for="amount-input"
          invalid-feedback="Nie podano wartości!"
        >
          <b-form-input
            id="amount-input"
            v-model="amount"
            required
          ></b-form-input>
        </b-form-group>
      </form>

      <template v-slot:modal-footer="{ ok, cancel }">
        <b-button size="sm" variant="success" @click="ok()">Kup</b-button>
        <b-button size="sm" variant="danger" @click="cancel()">Anuluj</b-button>
      </template>
    </b-modal>
    <b-modal v-bind:id="this.cryptocurrencyDetailsModel.cryptocurrency + 'sell-modal'"
           title="Kupno kryptowaluty"
           @ok="handleSellCryptocurrency">

    <p>kryptowaluta: {{this.cryptocurrencyDetailsModel.cryptocurrency}}</p>
    <p>cena sprzedaży: {{this.cryptocurrencyDetailsModel.tickerModel.ask}}</p>
    <p>posiadana ilość: {{this.cryptocurrencyDetailsModel.ownedAmount}}</p>
    <p></p>

    <form ref="form" @submit.stop.prevent="submitSellingCryptocurrency">
      <b-form-group
        label="Ilość"
        label-for="amount-input"
        invalid-feedback="Nie podano wartości!"
      >
        <b-form-input
          id="amount-input"
          v-model="amount"
          required
        ></b-form-input>
      </b-form-group>
    </form>

    <template v-slot:modal-footer="{ ok, cancel }">
      <b-button size="sm" variant="success" @click="ok()">Sprzedaj</b-button>
      <b-button size="sm" variant="danger" @click="cancel()">Anuluj</b-button>
    </template>
    </b-modal>
      <b-modal v-bind:id="this.cryptocurrencyDetailsModel.cryptocurrency + 'details-modal'" v-bind:title="'Szczegóły dla '+cryptocurrencyDetailsModel.cryptocurrency">
      <cryptocurrency-details v-bind:cryptocurrency=this.cryptocurrencyDetailsModel.cryptocurrency></cryptocurrency-details>
      <template v-slot:modal-footer="{ cancel }">
        <b-button size="sm" variant="success" @click="cancel()">Zamknij</b-button>
      </template>
    </b-modal>
  </b-container>
</template>

<script lang="ts">
import {Component, Prop, Vue} from 'vue-property-decorator'
import {StorageService} from '@/services/storage.service'
import {TransactionStatus} from '@/models/TransactionModel'
import CryptocurrencyDetails from '@/components/cryptocurrency/CryptocurrencyDetails.vue'
import { CryptocurrencyDetailsModel } from '@/models/CryptocurrencyDetailsModel'
import { EventBus } from '@/constants/EventBus'

@Component({
components: { CryptocurrencyDetails }
})
export default class CryptocurrencyRow extends Vue {
  @Prop() private cryptocurrencyDetailsModel!: CryptocurrencyDetailsModel;
  amount: number = 0;

  handleBuyCryptocurrency (bvModalEvent: Event) {
    this.submitBuyingCryptocurrency()
  }

  submitBuyingCryptocurrency () {
    if (!isNaN(this.amount)) {
      if (StorageService.balance > this.cryptocurrencyDetailsModel.tickerModel.bid * this.amount){
        this.buyCryptocurrency()
        this.reloadData()
      } else{
        alert('Nie masz wystarczającej ilości środków, brakuje Ci: '+(this.cryptocurrencyDetailsModel.tickerModel.bid * this.amount-StorageService.balance)+' $');
      }
    } else {
        alert('Podana ilość nie jest liczbą');
      }
  }

  buyCryptocurrency () {
    StorageService.buyCryptocurrency({
      date: new Date().toLocaleString(),
      amount: this.amount,
      price: -(+this.amount * +this.cryptocurrencyDetailsModel.tickerModel.ask),
      bidPrice: this.cryptocurrencyDetailsModel.tickerModel.bid,
      askPrice: this.cryptocurrencyDetailsModel.tickerModel.ask,
      status: TransactionStatus.BOUGHT,
      cryptocurrency: this.cryptocurrencyDetailsModel.cryptocurrency
    })
  }
  handleSellCryptocurrency (bvModalEvent: Event) {
    this.submitSellingCryptocurrency()
  }

  submitSellingCryptocurrency () {
    if(!isNaN(this.amount)) {
      const cryptoCurrencyTemp = StorageService.getCryptocurrencyAmount(this.cryptocurrencyDetailsModel.cryptocurrency)
      if(cryptoCurrencyTemp >= this.amount) {
        this.sellCryptocurrency()
        this.reloadData()
      } else {
        alert('Nie możesz tyle sprzedać, brakuje Ci: '+(this.amount-cryptoCurrencyTemp)+' '+this.cryptocurrencyDetailsModel.cryptocurrency)
      }
    }else {
        alert('Podana ilość nie jest liczbą');
    }
  }

  sellCryptocurrency () {
    StorageService.sellCryptocurrency({
      date: new Date().toLocaleString(),
      amount: this.amount,
      price: +this.amount * +this.cryptocurrencyDetailsModel.tickerModel.bid,
      bidPrice: this.cryptocurrencyDetailsModel.tickerModel.bid,
      askPrice: this.cryptocurrencyDetailsModel.tickerModel.ask,
      status: TransactionStatus.SOLD,
      cryptocurrency: this.cryptocurrencyDetailsModel.cryptocurrency
    })
  }

  reloadData () {
    EventBus.$emit('balance-change')
    this.cryptocurrencyDetailsModel.ownedAmount = StorageService.getCryptocurrencyAmount(this.cryptocurrencyDetailsModel.cryptocurrency)
  }

  stopFollowingCryptocurrency () {
    this.$emit('removedCryptocurrency', this.cryptocurrencyDetailsModel.cryptocurrency)
  }
}
</script>

<style scoped>

.cryptoContainer{
    border-bottom: 2px solid rgba(0, 0, 0, 0.3);
}

.cryptoContainer div{
  padding: 5px;
}
.cryptoContainer div div{
  align-self:center
}

.cryptoContainer div div div{
  justify-content: start;
}

</style>
