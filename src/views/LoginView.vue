<template>
  <form-page-template>
    <form class="uc-form">
      <div class="uc-form-header">
        <div class="uc-form-header-link--active">Log In</div>
        <router-link to="/signup" class="uc-form-header-link"
          >Sign Up</router-link
        >
      </div>

      <div class="uc-form-body">
        <div
          v-if="error || $route.query['401'] === 'true'"
          class="alert alert-danger"
          role="alert"
        >
          {{ error }}
        </div>

        <div class="uc-column">
          <label for="inputEmail" class="uc-form-label">Email</label>
          <input
            id="inputEmail"
            v-model="credentials.email"
            type="email"
            class="uc-form-input"
            required
            autofocus
          />
        </div>

        <div class="uc-column">
          <label for="inputPassword" class="uc-form-label">Password</label>
          <input
            id="inputPassword"
            v-model="credentials.password"
            type="password"
            class="uc-form-input"
            required
          />
          <router-link to="/resetpassword" class="uc-form-subtext">
            Forgot password?
          </router-link>
        </div>

        <button class="uc-form-button" type="submit" @click.prevent="submit">
          Login
        </button>
      </div>

      <form-footer v-if="!isMobileApp" />
    </form>
  </form-page-template>
</template>

<script>
import { mapState } from "vuex";
import AuthService from "@/services/AuthService";
import FormPageTemplate from "@/components/FormPageTemplate";
import FormFooter from "@/components/FormFooter";

export default {
  components: {
    FormPageTemplate,
    FormFooter
  },
  created() {
    this.$store.dispatch("app/hideNavigation");
  },
  computed: {
    ...mapState({
      isMobileApp: state => state.app.isMobileApp
    })
  },
  data() {
    let error;
    if (this.$route.query["401"] === "true") {
      error = "Your session has expired. Please log in again";
    }
    return {
      credentials: {
        email: "",
        password: ""
      },
      error
    };
  },
  methods: {
    submit() {
      AuthService.login(this, {
        email: this.credentials.email,
        password: this.credentials.password
      })
        .then(data => {
          this.$store.commit("user/setUser", data.user);
          this.$router.push(this.$route.query.redirect || "/");
        })
        .catch(() => {
          this.error = "Oops! The email or password you entered is incorrect.";
        });
    }
  }
};
</script>

<style lang="scss" scoped>
.alert {
  margin-bottom: 0;
}
</style>
