<template>
  <div class="page-search">
    <div class="columns is-multiline">
      <div class="column is-12">
        <h1 class="title">Search</h1>
        <p class="is-size-5 has-text-grey">Search term:"{{ query }}"</p>
      </div>
    </div>
    <product-box
      v-for="product in products"
      :key="product.id"
      :product="product"
    />
  </div>
</template>

<script>
  import axios from "axios";
  import ProductBox from "../components/ProductBox.vue";
  export default {
    components: { ProductBox },
    name: "Search",
    data() {
      return {
        products: [],
        query: "",
      };
    },
    mounted() {
      document.title = "Search | VGJ";
      let url = window.location.search.substring(1);
      let params = new URLSearchParams(url);
      if (params.get("query")) {
        this.query = params.get("query");
        this.performSearch();
      }
    },
    methods: {
      async performSearch() {
        this.$store.commit("setIsLoad", true);
        await axios
          .post("/api/v1/products/search/", { query: this.query })
          .then((res) => {
            this.products = res.data;
          })
          .catch((err) => console.log(err));
        this.$store.commit("setIsLoad", false);
      },
    },
  };
</script>

<style></style>
