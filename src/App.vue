<template>
  <div id="subscription">
    <div id="content-subscription" class="container">
      <div id="title-subscription">
        Subscriptions
      </div>

      <div id="buttons-subscription" class="row justify-content-between pb-4">
        <div id="all-button-subscription" class="col d-flex justify-content-center">
          <div class="btn-dark w-100 text-center">All</div>
        </div>
      </div>

      <div id="main-subscription"
        class="d-flex justify-content-center align-items-center border border-dark border-1 rounded">
        <div v-if="!loading" id="list-main-subscription" class="overflow-auto">
          <div v-for="(product, idx) in products" class="card" :class="getClass(idx, products.length - 1)">
            <div class="row align-items-center">
              <div class="col-6">
                <div class="card-title text-center">
                  {{ product.name }}
                </div>
              </div>
              <div class="col-6">
                <div class="card-body">
                  <div class="row pb-1 justify-content-end align-items-center text-center">{{ subscriptions[idx].active ?
                    'Subscribed' : 'Not Subscribed' }}</div>
                  <div v-if="subscriptions[idx].active" class="row py-1 justify-content-end align-items-center text-center">
                    <div class="col px-0 d-flex justify-content-end">
                      <a href="https://www.sandbox.paypal.com/myaccount/summary" target="_blank" class="btn btn-secondary p-0">Invoice</a>
                    </div> 
                  </div>
                  <div class="row pt-1">
                    <div class="col px-0 d-flex justify-content-end">
                      <button v-if="!subscriptions[idx].active"
                        @click="showSubPopup = true; planClicked = idx; clickSub(planClicked)" 
                        class="btn btn-primary p-0">Subscribe</button>
                      <button v-else @click="showCancelPopup = true; planClicked = idx;"
                      class="btn btn-primary p-0">Unsubscribe</button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

    </div>
    <div v-if="showSubPopup" class="modal fade show d-block" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Confirmation</h5>
            <button type="button" class="btn-close" @click="showSubPopup = false"></button>
          </div>
          <div class="modal-body">
            <p>Click the button to subscribe to {{ products[planClicked].name }}</p>
            <div id="paypal-button-container"></div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" @click="showSubPopup = false">Close</button>
          </div>
        </div>
      </div>
    </div>
    <div v-if="showSubPopup" class="modal-backdrop fade show"></div>
    <div v-if="showCancelPopup" class="modal fade show d-block" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Confirmation</h5>
            <button type="button" class="btn-close" @click="showCancelPopup = false"></button>
          </div>
          <div class="modal-body">
            <p>Unsubscribe from {{ products[planClicked].name }}?</p>
            <button @click="clickCancel(planClicked)" class="btn btn-dark">Cancel Subscription</button>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" @click="showCancelPopup = false">Close</button>
          </div>
        </div>
      </div>
    </div>
    <div v-if="showSubPopup" class="modal-backdrop fade show"></div>
  </div>
</template>



<script>
import { ref } from 'vue';
import "bootstrap/dist/css/bootstrap.min.css";
import "bootstrap";

export default {
  data() {
    return {
      accessToken: '',
      products: '',
      plans: '',
      planClicked: 0,
      showSubPopup: false,
      showCancelPopup: false,
      subscriptions: [],
      user: {
        'first_name': 'John',
        'last_name': 'Doe',
        'email': 'sb-xf347326823863@personal.example.com',

      }
    }
  },
  methods: {
    getClass(a, b) {
      if (a == 0) {
        return "mb-3";
      } else if (a == b) {
        return "mt-3";
      } else {
        return "my-3";
      }
    },
    // Method to subscribe
    async clickSub(idx) {
      // Delay to make sure #paypal-button-container exists
      await new Promise((resolve) => setTimeout(resolve, 1000));
      // Setup subscription payment 
      paypal.Buttons({
        createSubscription: (data, actions) => {
          return actions.subscription.create({
            'plan_id': this.plans[idx].id // Creates the subscription
          });
        },
        onApprove: async (data, actions) => {
          alert('You have successfully subscribed to ' + this.products[idx].name + ', Subscription ID = ' + data.subscriptionID); // Optional message given to subscriber
          this.subscriptions[idx] = {
            'subscriptionId': data.subscriptionID,
            'active': true,
          }
          localStorage.subscriptions = JSON.stringify(this.subscriptions);
          this.showSubPopup = false;
        }
      }).render('#paypal-button-container'); // Renders the PayPal button
    },
    // Method to cancel a subscription
    async clickCancel(idx) {
      const url = `https://api-m.sandbox.paypal.com/v1/billing/subscriptions/${this.subscriptions[idx].subscriptionId}/cancel`;
      const headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.accessToken}`
      };
      const body = JSON.stringify({
        reason: 'Not satisfied with the service'
      });
      const response = await fetch(url, { method: 'POST', headers, body });
      this.subscriptions[idx] = {
        'subscriptionId': '',
        'active': false,
      }
      localStorage.subscriptions = JSON.stringify(this.subscriptions);
      this.showCancelPopup = false;
    },
    // Method to get access token 
    async getAccessToken() {
      const clientId = 'AVoXqrAH2A1d4o3gcT5sYu95ITm-RhTKL6OyetvPU-q_gvFRn_rTxR4ccFz9h6AYRcKNhj2cdQdr_E5_';
      const clientSecret = 'EPOKkSLn25PDCWHtUOPM-gqEdqs5CmNzCUWKd6ncr18wJhkfYOrZmgkHqwmZbAp9byKDu0ewb2wJ6tQ3';
      const url = 'https://api-m.sandbox.paypal.com/v1/oauth2/token';
      const auth = btoa(`${clientId}:${clientSecret}`);
      const headers = {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Authorization': `Basic ${auth}`
      };
      const body = 'grant_type=client_credentials';
      const response = await fetch(url, { method: 'POST', headers, body });
      const data = await response.json();
      this.accessToken = data.access_token;
    },
    // Method to get all products
    async getProducts() {
      const url = 'https://api-m.sandbox.paypal.com/v1/catalogs/products?page_size=20&page=1&total_required=true';
      const headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.accessToken}`
      };
      const response = await fetch(url, { method: 'GET', headers });
      const data = await response.json();
      this.products = data.products;
    },
    // Method to get all plans
    async getPlans() {
      const url = 'https://api-m.sandbox.paypal.com/v1/billing/plans?page_size=20&page=1&total_required=true';
      const headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.accessToken}`
      };
      const response = await fetch(url, { method: 'GET', headers });
      const data = await response.json();
      this.plans = data.plans;
    },
    // Method to get all subscriptions
    async getSubscriptions() {
      const url = 'https://api-m.sandbox.paypal.com/v1/billing/subscriptions';
      const headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.accessToken}`
      };
      const response = await fetch(url, { method: 'GET', headers });
      const data = await response.json();
      this.subscriptions = data.subscriptions;
    },
    // Method to create a product using PayPal's POST catalog product API
    // In this project, this method is purely for debug purposes and is thus commented
    async createProduct(name, desc) {
      const url = 'https://api-m.sandbox.paypal.com/v1/catalogs/products';
      const headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.accessToken}`
      };
      const body = JSON.stringify({
        name: name,
        description: desc,
        type: 'SERVICE',
        category: 'SOFTWARE'
      });
      const response = await fetch(url, { method: 'POST', headers, body });
      const data = await response.json();
      console.log(data);
      await this.getProducts();
    },
    async createPlan(productId, name, desc, price) {
      const createUrl = 'https://api-m.sandbox.paypal.com/v1/billing/plans';
      const headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.accessToken}`
      };
      const body = JSON.stringify({
        product_id: productId,
        name: name,
        description: desc,
        billing_cycles: [
          {
            frequency: {
              interval_unit: 'MONTH',
              interval_count: 1
            },
            tenure_type: 'REGULAR',
            sequence: 1,
            total_cycles: 0,
            pricing_scheme: {
              fixed_price: {
                value: price.toString(),
                currency_code: 'USD'
              }
            }
          }
        ],
        payment_preferences: {
          auto_bill_outstanding: true,
          setup_fee: {
            value: '0',
            currency_code: 'USD'
          },
          setup_fee_failure_action: 'CONTINUE',
          payment_failure_threshold: 3
        },
      });
      const createResponse = await fetch(createUrl, { method: 'POST', headers, body });
      const createData = await createResponse.json();
      console.log(createData);
      await this.getPlans();
    },
  },
  async mounted() {
    // Get all buttons
    this.buttons = document.querySelectorAll('#buttons-subscription button');
    for (let button of this.buttons) {
      this.buttonsActive.push(false);
    }
    await this.getAccessToken();
    await this.getProducts();
    //await this.createProduct('Spotify', 'Spotify subscription service');
    //await this.createProduct('OpenAI', 'OpenAI subscription service');
    await this.getPlans();
    //await this.createPlan(this.products[1].id, 'Spotify Standard Plan', 'Spotify\'s standard monthly subscription plan', 5);
    //await this.createPlan(this.products[2].id, 'OpenAI GPT-4 Plan', 'OpenAI\'s monthly subscription plan to use GPT-4', 5);
    //await this.getSubscriptions();
    this.loading = false;
    console.log(this.products);
    console.log(this.plans);

    if (!localStorage.subscriptions) {
      for (let a = 0; a < this.products.length; a++) {
        this.subscriptions.push({
          'subscriptionId': '',
          'active': false,
        })
      }
      localStorage.subscriptions = JSON.stringify(this.subscriptions);
    } else {
      this.subscriptions = JSON.parse(localStorage.subscriptions);
    }
    console.log(this.subscriptions);
  },
  setup() {
    const buttons = ref();
    const buttonsActive = ref([]);
    const loading = ref(true);

    function buttonActive(num) {
      console.log(buttons.value[num].classList.contains('active'));
      return buttons.value[num].classList.contains('active');
    }

    function toggleActivation(num) {
      for (let m = 0; m < buttons.value.length; m++) {
        console.log(buttons.value[m]);
        if (m == num) {
          if (!buttons.value[m].classList.contains('active')) {
            buttons.value[m].classList.add('active');
            buttonsActive.value[m] = true;
          }
        } else {
          buttons.value[m].classList.remove('active');
          buttonsActive.value[m] = false;
        }
      }
    }


    return {
      buttons,
      buttonsActive,
      loading,
      buttonActive,
      toggleActivation
    }
  }
}
</script>

<style>
#buttons-subscription .active {
  background-color: #17a2b8;
  color: white;
}

#title-subscription {
  font-size: 3vw;
}

#main-subscription {
  height: 40vw;
}

#list-main-subscription {
  height: 95%;
  width: 95%;
}
</style>