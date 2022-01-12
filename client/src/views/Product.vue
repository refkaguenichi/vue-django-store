<template>
  <div class="page-product">
    <div class="columns is-multiline">
      <div class="column is-8">
        <figure class="image mb-6">
          <img :src="product.get_image" />
        </figure>
      </div>
      <div class="column is-4">
        <h2 class="subtitle">Product informations</h2>
        <h1 class="title">{{ product.name }}</h1>
        <p><strong>Description:</strong>{{ product.description }}</p>
        <p><strong> Price: </strong> ${{ product.price }}</p>
        <div class="field has-addons mt-6">
          <input type="number" class="input" min="1" v-model="quantity" />
          <div class="control">
            <a class="button is-dark" @click="addToCart">Add to cart</a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from "axios";
  import { toast } from "bulma-toast";
  export default {
    name: "Product",
    data() {
      return {
        product: {},
        quantity: 1,
      };
    },
    mounted() {
      this.getProduct();
    },
    methods: {
      async getProduct() {
        this.$store.commit("setIsLoad", true);
        const category_slug = this.$route.params.category_slug;
        const product_slug = this.$route.params.product_slug;
        await axios
          .get(`/api/v1/products/${category_slug}/${product_slug}`)
          .then((res) => {
            this.product = res.data;
            document.title = this.product.name + " | VDJ";
          })
          .catch((err) => console.log(err));
        this.$store.commit("setIsLoad", false);
      },
      addToCart() {
        if (isNaN(this.quantity) || this.quantity < 1) {
          this.quantity = 1;
        }
        const item = {
          product: this.product,
          quantity: this.quantity,
        };
        this.$store.commit("addToCart", item);

        toast({
          message: "The product was added to the cart",
          type: "is-success",
          dismissible: true,
          pauseOnHover: true,
          duration: 2000,
          position: "bottom-right",
        });
      },
    },
  };
</script>

<style scoped></style>
