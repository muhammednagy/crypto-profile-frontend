<template>
  <div id="app">
    <h1>Welcome to Your crypto portfolio!</h1>
    <div class="container">
      <b-table striped hover :items="trades" :fields="fields"></b-table>
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
        fields: ['currency', 'amount', 'date_of_purchase', 'note', 'Current market value'],
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
        axios.get(`https://crypto-profile-api.herokuapp.com/api/v1/trades`)
                .then(response => {
                  // JSON responses are automatically parsed.
                  this.trades = response.data
                })
                .catch(e => {
                  this.errors.push(e)
                })
      },
      addTrade(evt) {
        evt.preventDefault()
        alert(JSON.stringify(this.form))

        return new Promise((resolve, reject) => {
          axios.post('https://crypto-profile-api.herokuapp.com/api/v1/trades', {
            amount: this.form.amount,
            note: this.form.note,
            date_of_purchase: this.form.date_of_purchase,
            currency: this.form.currency,
          })
                  .then(response => {
                    resolve(response)
                    this.getTrades()
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
        // Trick to reset/clear native browser form validation state
        this.show = false
        this.$nextTick(() => {
          this.show = true
        })
      }
    },

    // Fetches posts when the component is created.
    created() {
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
