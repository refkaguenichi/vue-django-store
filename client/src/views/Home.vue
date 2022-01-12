<template>
  <div class="home">
    <section class="hero is-dark p-6">
      <div class="hero-body has-text-centered">
        <p class="title mb-6">Welcome to VDJ store</p>
        <p class="subtitle">The best store online</p>
      </div>
    </section>
    <div class="columns is-multiline">
      <div class="column is-12">
        <h2 class="is-size-2 has-text-centered">Latest products</h2>
      </div>
      <product-box
        v-for="product in latestProducts"
        :key="product.id"
        :product="product"
      />
    </div>
  </div>
</template>

<script>
  import axios from "axios";
  import ProductBox from "../components/ProductBox.vue";
  export default {
    name: "Home",
    components: { ProductBox },
    data() {
      return {
        latestProducts: [],
      };
    },
    mounted() {
      this.getLatestProducts();
      document.title = "Home | VDJ";
    },
    methods: {
      async getLatestProducts() {
        this.$store.commit("setIsLoad", true);
        await axios
          .get("/api/v1/latest-products/")
          .then((res) => {
            this.latestProducts = res.data;
          })
          .catch((err) => console.log(err));
        this.$store.commit("setIsLoad", false);
      },
    },
  };
</script>
