<template>
  <div class="page-sign-up">
    <div class="columns">
      <div class="column is-4 is-offset-4">
        <h1 class="title">Log in</h1>
        <form @submit.prevent="submitForm">
          <div class="field">
            <label for="">Username</label>
            <div class="control">
              <input type="text" class="input" v-model="username" />
            </div>
          </div>
          <div class="field">
            <label for="">Password</label>
            <div class="control">
              <input type="password" class="input" v-model="password" />
            </div>
          </div>
          <div v-if="errors.length">
            <p
              class="notification is-danger"
              v-for="error in errors"
              :key="error"
            >
              {{ error }}
            </p>
          </div>
          <div class="field">
            <div class="control">
              <button class="button is-dark">Log in</button>
            </div>
          </div>
          <hr />
          Or<router-link to="/sign-up"> click here </router-link>to sign up!
        </form>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from "axios";
  import { toast } from "bulma-toast";
  export default {
    name: "LogIn",
    data() {
      return {
        username: "",
        password: "",
        errors: [],
      };
    },
    mounted() {
      document.title = "Log In | VDJ";
    },
    methods: {
      async submitForm() {
        axios.defaults.headers.common["Authorization"] = "";
        localStorage.removeItem("token");
        const formData = {
          username: this.username,
          password: this.password,
        };
        await axios
          .post("/api/v1/token/login/", formData)
          .then((res) => {
            const token = res.data.auth_token;
            this.$store.commit("setToken", token);
            axios.defaults.headers.common["Authorization"] = "Token" + token;
            localStorage.setItem("token", token);
            const toPath = this.$route.query.to || "/cart";
            this.$router.push(toPath);
          })
          .catch((err) => {
            if (err.res) {
              for (const property in error.res.data) {
                this.errors.push(`${property}:${err.res.data[property]}`);
              }
            } else {
              this.errors.push("Something went wrong, please try again");
              console.log(JSON.stringify(err));
            }
          });
      },
    },
  };
</script>

<style></style>
