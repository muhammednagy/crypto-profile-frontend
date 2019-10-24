<template>
  <div id="app">
    <h1>Welcome to Your crypto portfolio!</h1>
    <div class="container">
      <b-table striped hover :busy="isBusy" :items="trades" :fields="fields">
        <template v-slot:table-busy>
          <div class="text-center text-danger my-2">
            <b-spinner class="align-middle"></b-spinner>
            <strong>Loading...</strong>
          </div>
        </template>
        <template  v-slot:cell(delete_link)="row">
          <b-button size="sm" @click="onDelete(row.item.id)" class="mr-2">
            Delete
          </b-button>
        </template>

      </b-table>
      <b-form @submit="addTrade" @reset="onReset" v-if="show">
        <b-form-group
                id="input-group-1"
                label="Amount: "
                label-for="input-1"
        >
          <b-form-input
                  id="input-1"
                  v-model="form.amount"
                  required
                  placeholder="Enter Amount"
          ></b-form-input>
        </b-form-group>

        <b-form-group id="input-group-2" label="Notes:" label-for="input-2">
          <b-form-input
                  id="input-2"
                  v-model="form.note"
                  required
                  placeholder="Write notes about the trade"
          ></b-form-input>
        </b-form-group>

        <b-form-group id="input-group-3" label="Date of purchase:" label-for="input-3">
          <b-form-input
                  id="input-2"
                  v-model="form.date_of_purchase"
                  type="date"
                  required
          ></b-form-input>
        </b-form-group>

        <b-form-group id="input-group-4" label="Currencies:" label-for="input-4">
          <b-form-select
                  id="input-4"
                  v-model="form.currency"
                  :options="currencies"
                  required
          ></b-form-select>
        </b-form-group>



        <b-button type="submit" variant="primary">Submit</b-button>
        <b-button type="reset" variant="danger">Reset</b-button>
      </b-form>
    </div>




  </div>
</template>

<script>
  import axios from 'axios';

  export default {
    data() {
      return {
        isBusy: false,
        fields: ['currency', 'amount', 'date_of_purchase', 'note', 'Current market value', 'delete_link'],
        trades: [],
        trade: [],
        form: {
          amount: '',
          note: '',
          date_of_purchase: new Date().toISOString().slice(0,10) ,
          currency: null
        },
        currencies: [{ text: 'Select One', value: null }, 'bitcoin', 'ethereum', 'ripple'],
        show: true,
        errors: []
      }
    },
    methods: {
      getTrades(){
        axios.get(process.env.VUE_APP_API + '/trades')
                .then(response => {
                  // JSON responses are automatically parsed.
                  this.trades = response.data
                  this.toggleBusy()
                })
                .catch(e => {
                  this.errors.push(e)
                })
      },
      addTrade(evt) {
        evt.preventDefault()
        return new Promise((resolve, reject) => {
          axios.post(process.env.VUE_APP_API + '/trades', {
            amount: this.form.amount,
            note: this.form.note,
            date_of_purchase: this.form.date_of_purchase,
            currency: this.form.currency,
          })
                  .then(response => {
                    resolve(response)
                    this.getTrades()
                    this.toggleBusy()
                  })
                  .catch(error => {
                    reject(error.response.data)
                  })
        })

      },
      onReset(evt) {
        evt.preventDefault()
        // Reset our form values
        this.form.amount = ''
        this.form.note = ''
        this.form.date_of_purchase = new Date().toISOString().slice(0,10)
        this.form.currency = null
      },

      onDelete(id){
        var retVal = confirm("Are you sure you want to delete this trade");
        if( retVal == true ) {
        return new Promise((resolve, reject) => {
          axios.delete(process.env.VUE_APP_API + '/trades/' + id)
                  .then(response => {
                    this.toggleBusy()
                    resolve(response)
                    this.getTrades()
                  })
                  .catch(error => {
                    reject(error.response.data)
                  })
        })
        }
      },
      toggleBusy() {
        this.isBusy = !this.isBusy
      },
    },

    created() {
      this.toggleBusy()
      this.getTrades()
    }
  }
</script>
<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
