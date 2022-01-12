<template>
  <div class="page-checkout">
    <div class="columns is-multiline">
      <div class="coulumn is-12">
        <h1 class="title">Checkout</h1>
      </div>
      <div class="column is-12 box">
        <table class="table is-fullwidth">
          <thead>
            <tr>
              <th>Product</th>
              <th>Price</th>
              <th>Quantity</th>
              <th>Total</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in cart.items" :key="item.product.id">
              <td>{{ item.product.name }}</td>
              <td>${{ item.product.price }}</td>
              <td>{{ item.quantity }}</td>
              <td>${{ getItemTotal(item).toFixed(2) }}</td>
            </tr>
          </tbody>
          <tfoot>
            <tr>
              <td colspan="2">Total</td>
              <td>{{ cartTotalLength }}</td>
              <td>${{ cartTotalPrice.toFixed(2) }}</td>
            </tr>
          </tfoot>
        </table>
      </div>
      <div class="column is-12 box">
        <h2 class="subtitle">Shipping details</h2>
        <p class="has-text-grey mb-4">*All fields are required</p>
        <div class="columns is-multiline">
          <div class="column is-6">
            <div class="field">
              <label>First name*</label>
              <input type="text" class="input" v-model="first_name" />
            </div>
            <div class="field">
              <label>Last name*</label>
              <input type="text" class="input" v-model="last_name" />
            </div>
            <div class="field">
              <label>E-mail*</label>
              <input type="email" class="input" v-model="email" />
            </div>
            <div class="field">
              <label>Phone*</label>
              <input type="text" class="input" v-model="phone" />
            </div>
          </div>
          <div class="column is-6">
            <div class="field">
              <label>Address*</label>
              <input type="text" class="input" v-model="address" />
            </div>
            <div class="field">
              <label>Zip code*</label>
              <input type="text" class="input" v-model="zipcode" />
            </div>
            <div class="field">
              <label>Place</label>
              <input type="text" class="input" v-model="place" />
            </div>
          </div>
        </div>
        <div class="notification is-danger mt-4" v-if="errors.length">
          <p v-for="error in errors" :key="error">
            {{ error }}
          </p>
        </div>
        <hr />
        <div id="card-element" class="mb-5">
          <!--Stripe.js injects the Payment Element-->
        </div>
        <template v-if="cartTotalLength">
          <hr />
          <button class="button is-dark" @click="submitForm">
            Pay with Stripe
          </button>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from "axios";
  export default {
    name: "CheckOut",
    data() {
      return {
        cart: {
          items: [],
        },
        stripe: {},
        card: {},
        first_name: "",
        last_name: "",
        email: "",
        phone: "",
        address: "",
        zipcode: "",
        place: "",
        errors: [],
      };
    },
    mounted() {
      document.title = "Checkout | VDJ";
      this.cart = this.$store.state.cart;
      if (this.cartTotalLength > 0) {
        this.stripe = Stripe(
          "pk_test_51K3fxIFkV7Vls8jC6bnZB5vv1bzm25DwEuw7hRPtS1p4g1mNTMZ1BhDw5RoT62NXtSQbWaIPTsblCdDfkVPYNsU200eh5j0uZK"
        );
        const elements = this.stripe.elements();
        this.card = elements.create("card", { hidePostalCode: true });

        this.card.mount("#card-element");
      }
    },
    methods: {
      getItemTotal(item) {
        return item.quantity * item.product.price;
      },
      submitForm() {
        this.errors = [];
        if (this.first_name === "") {
          this.errors.push("The first name filed is missing!");
        }
        if (this.last_name === "") {
          this.errors.push("The last name filed is missing!");
        }
        if (this.email === "") {
          this.errors.push("The email filed is missing!");
        }
        if (this.phone === "") {
          this.errors.push("The phone filed is missing!");
        }
        if (this.address === "") {
          this.errors.push("The address filed is missing!");
        }
        if (this.zipcode === "") {
          this.errors.push("The zipcode filed is missing!");
        }
        if (this.place === "") {
          this.errors.push("The place filed is missing!");
        }
        if (!this.errors.length) {
          this.$store.commit("setIsLoad", true);
          this.stripe.createToken(this.card).then((res) => {
            if (res.error) {
              this.$store.commit("setIsLoad", false);
              this.errors.push(
                "Something went wrong with Stripe. Please try again"
              );
              console.log(res.error.message);
            } else {
              this.stripeTokenHandler(res.token);
            }
          });
        }
      },
      async stripeTokenHandler(token) {
        const items = [];
        for (let i = 0; i < this.cart.items.length; i++) {
          const item = this.cart.items[i];
          const obj = {
            product: item.product.id,
            quantity: item.quantity,
            price: item.product.price * item.quantity,
          };
          items.push(obj);
          console.log(obj.price);
        }
        const data = {
          first_name: this.first_name,
          last_name: this.last_name,
          email: this.email,
          phone: this.phone,
          address: this.address,
          zipcode: this.zipcode,
          place: this.place,
          items: items,
          stripe_token: token.id,
        };

        await axios
          .post("/api/v1/checkout/", data)
          .then((res) => {
            this.$store.commit("clearCart");
            this.$router.push("/cart/success");
          })
          .catch((err) => {
            this.errors.push("Something went wrong. Please try again");
            console.log(err);
          });
        this.$store.commit("setIsLoad", false);
      },
    },
    computed: {
      cartTotalPrice() {
        return this.cart.items.reduce((acc, curVal) => {
          return (acc += curVal.product.price * curVal.quantity);
        }, 0);
      },
      cartTotalLength() {
        return this.cart.items.reduce((acc, curVal) => {
          return (acc += curVal.quantity);
        }, 0);
      },
    },
  };
</script>

<style></style>
