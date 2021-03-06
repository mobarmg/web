<template>
  <form-page-template>
    <div class="uc-form">
      <div class="uc-form-header">
        <router-link to="/login" class="uc-form-header-link"
          >Log In</router-link
        >
        <div class="uc-form-header-link--active">Sign Up</div>
      </div>

      <form
        v-if="formStep === 'step-1'"
        class="uc-form-body"
        @submit.prevent="formStepTwo()"
      >
        <div v-if="errors.length" class="step-errors">
          <h5>Please correct the following problems:</h5>
          <ul>
            <li v-for="error in errors" v-bind:key="error">{{ error }}</li>
          </ul>
        </div>

        <div class="step-header">
          <div class="step-header__title">
            {{ welcomeTitle }}
          </div>
          <div class="step-header__subtitle">
            Not with {{ volunteerPartner.name }}?
            <a href="https://upchieve.org/volunteer">Click here</a>
          </div>
        </div>

        <div class="uc-column">
          <label for="inputEmail" class="uc-form-label">{{
            emailInputLabel
          }}</label>
          <input
            id="inputEmail"
            type="email"
            class="uc-form-input"
            v-bind:class="{
              'uc-form-input--invalid': invalidInputs.indexOf('inputEmail') > -1
            }"
            v-model="formData.email"
            required
            autofocus
            autocomplete="email"
          />
        </div>

        <div class="uc-column">
          <label for="inputEmail" class="uc-form-label"
            >Create a password</label
          >
          <input
            id="inputPassword"
            type="password"
            class="uc-form-input"
            v-bind:class="{
              'uc-form-input--invalid':
                invalidInputs.indexOf('inputPassword') > -1
            }"
            v-model="formData.password"
            required
            autocomplete="new-password"
          />
          <p class="uc-form-subtext">
            Must contain at least one number, one uppercase letter, and one
            lowercase letter.
          </p>
        </div>

        <button
          class="uc-form-button"
          type="submit"
          @click.prevent="formStepTwo()"
        >
          Continue
        </button>

        <div v-if="serverErrorMsg !== ''">{{ serverErrorMsg }}</div>
      </form>
      <form
        v-if="formStep === 'step-2'"
        class="uc-form-body"
        @submit.prevent="submitSignupForm()"
      >
        <div v-if="errors.length" class="step-errors">
          <h5>Please correct the following problems:</h5>
          <ul>
            <li v-for="error in errors" v-bind:key="error">{{ error }}</li>
          </ul>
        </div>

        <div class="step-header">
          <div class="step-header__title">Finish creating your account</div>
          <div class="step-header__subtitle">
            You’re just a few minutes away from helping low-income students!
          </div>
        </div>

        <div class="uc-column">
          <!-- Fix for bug in Chrome where the first two fields are parsed as a username and password
          even if the HTML5 autocomplete attributes are set to the right values -->
          <label for="username" class="d-none">Username</label>
          <input
            type="text"
            class="d-none"
            id="username"
            v-model="formData.email"
            autocomplete="username"
          />
          <label for="password" class="d-none">Password</label>
          <input
            type="password"
            class="d-none"
            id="password"
            v-model="formData.password"
            autocomplete="new-password"
          />

          <div class="uc-column">
            <label for="firstName" class="uc-form-label"
              >What's your name?</label
            >
            <div class="uc-row name-fields">
              <div class="uc-column">
                <input
                  id="firstName"
                  type="text"
                  class="uc-form-input"
                  v-bind:class="{
                    'uc-form-input--invalid':
                      invalidInputs.indexOf('firstName') > -1
                  }"
                  v-model="formData.firstName"
                  required
                  autofocus
                  autocomplete="given-name"
                />
                <p class="uc-form-subtext">First Name</p>
              </div>
              <div class="uc-column">
                <input
                  id="lastName"
                  type="text"
                  class="uc-form-input"
                  v-bind:class="{
                    'uc-form-input--invalid':
                      invalidInputs.indexOf('lastName') > -1
                  }"
                  v-model="formData.lastName"
                  required
                  autocomplete="family-name"
                />
                <p class="uc-form-subtext">Last Name</p>
              </div>
            </div>
          </div>
        </div>

        <div class="uc-column">
          <label for="phoneNumber" class="uc-form-label"
            >What's your phone number?</label
          >
          <vue-phone-number-input
            id="phoneNumber"
            class="phone-input"
            v-model="formData.phone"
            :required="true"
            :error="
              invalidInputs.indexOf('phone') > -1 && !phoneInputInfo.isValid
            "
            color="#555"
            valid-color="#16ba97"
            @update="onPhoneInputUpdate"
          />
          <p class="uc-form-subtext">
            We only use this to notify you of incoming student requests. You can
            customize when you receive requests.
          </p>
        </div>

        <div class="uc-form-checkbox">
          <input
            id="userAgreement"
            v-model="formData.terms"
            type="checkbox"
            required
          />
          <label for="userAgreement">
            I have read and accept the
            <a href="/legal" target="_blank">user agreement</a> and
            <a href="/legal#volunteer" target="_blank">volunteer agreement</a>.
          </label>
        </div>

        <button class="uc-form-button" type="submit">
          Sign Up
        </button>

        <div v-if="serverErrorMsg !== ''">{{ serverErrorMsg }}</div>
      </form>
      <div v-if="formStep === 'success'" class="uc-form-body">
        You've been sent a verification email! Check your work email for a link
        to confirm your account.
      </div>
    </div>
  </form-page-template>
</template>

<script>
import validator from "validator";
import * as Sentry from "@sentry/browser";

import FormPageTemplate from "@/components/FormPageTemplate";
import AuthService from "@/services/AuthService";
import NetworkService from "@/services/NetworkService";

export default {
  name: "volunteer-partner-signup-view",
  components: {
    FormPageTemplate
  },
  beforeRouteEnter(to, from, next) {
    const partnerId = to.params.partnerId;

    NetworkService.getVolunteerPartner(partnerId)
      .then(data => {
        const volunteerPartner = data.body.volunteerPartner;
        if (!volunteerPartner) return next("/signup");
        return next(_this => _this.setVolunteerPartner(volunteerPartner));
      })
      .catch(err => {
        if (err.status !== 404) {
          // we shouldn't get 422 here, since semantics of GET request are expected
          // to be correct regardless of user input
          Sentry.captureException(err);
        }
        return next("/signup");
      });
  },
  created() {
    this.$store.dispatch("app/hideNavigation");
  },
  data() {
    return {
      volunteerPartner: {
        name: "",
        requiredEmailDomains: []
      },
      formStep: "step-1",
      formData: {
        email: "",
        password: "",
        firstName: "",
        lastName: "",
        phone: "",
        terms: false
      },
      phoneInputInfo: {},
      errors: [],
      invalidInputs: [],
      serverErrorMsg: ""
    };
  },
  computed: {
    hasEmailValidation() {
      const requiredDomains = this.volunteerPartner.requiredEmailDomains;
      return requiredDomains && requiredDomains.length;
    },

    welcomeTitle() {
      const typeOfPerson = this.hasEmailValidation ? "Employee" : "Volunteer";
      return `Welcome ${this.volunteerPartner.name} ${typeOfPerson}!`;
    },

    emailInputLabel() {
      return this.hasEmailValidation
        ? "What's your work email?"
        : "What's your email?";
    }
  },
  methods: {
    setVolunteerPartner(volunteerPartner) {
      this.volunteerPartner = volunteerPartner;
    },

    isValidPartnerEmail(email) {
      const requiredDomains = this.volunteerPartner.requiredEmailDomains;
      if (!(requiredDomains && requiredDomains.length)) return true;

      const domain = email.split("@")[1];
      return domain && requiredDomains.indexOf(domain) >= 0;
    },

    onPhoneInputUpdate(phoneInputInfo) {
      this.phoneInputInfo = phoneInputInfo;
    },

    formStepTwo() {
      // validate input
      this.errors = [];
      this.invalidInputs = [];

      if (!this.formData.email) {
        this.errors.push("An email address is required.");
        this.invalidInputs.push("inputEmail");
      } else if (!validator.isEmail(this.formData.email)) {
        // this is necessary because browsers ignore <input type="email"> until the
        // user actually tries to submit the form, which does not occur until step 2
        this.errors.push(
          this.formData.email + " is not a valid email address."
        );

        this.invalidInputs.push("inputEmail");
      } else if (!this.isValidPartnerEmail(this.formData.email)) {
        this.errors.push(
          `Email must end with ${this.volunteerPartner.requiredEmailDomains.join(
            " or "
          )}`
        );
        this.invalidInputs.push("inputEmail");
      }

      if (!this.formData.password) {
        this.errors.push("A password is required.");
        this.invalidInputs.push("inputPassword");
      }
      if (this.errors.length) {
        return;
      }

      // Check credentials
      AuthService.checkRegister(this, {
        email: this.formData.email,
        password: this.formData.password
      })
        .then(() => {
          this.formStep = "step-2";
          this.serverErrorMsg = "";
        })
        .catch(err => {
          this.serverErrorMsg = err.message;
          if (err.status !== 409 && err.status !== 422) {
            Sentry.captureException(err);
          }
        });
    },

    submitSignupForm() {
      this.errors = [];
      this.invalidInputs = [];

      // validate input
      if (!this.formData.firstName || !this.formData.lastName) {
        this.errors.push("You must enter your first and last name.");
      }
      if (!this.formData.firstName) {
        this.invalidInputs.push("firstName");
      }
      if (!this.formData.lastName) {
        this.invalidInputs.push("lastName");
      }
      if (!this.formData.phone) {
        this.errors.push("You must enter a phone number.");
        this.invalidInputs.push("phone");
      } else if (!this.phoneInputInfo.isValid || !this.phoneInputInfo.e164) {
        this.errors.push(this.formData.phone + " is not a valid phone number.");
        this.invalidInputs.push("phone");
      }
      if (!this.formData.terms) {
        this.errors.push(
          "You must read and accept the user agreement and volunteer agreement."
        );
      }

      if (!this.errors.length) {
        this.register();
      }
    },

    register() {
      AuthService.register(this, {
        isVolunteer: true,
        volunteerPartnerOrg: this.$route.params.partnerId,
        partnerUserId: this.$route.query.uid,
        email: this.formData.email,
        password: this.formData.password,
        firstName: this.formData.firstName,
        lastName: this.formData.lastName,
        phone: this.phoneInputInfo.e164,
        terms: this.formData.terms
      })
        .then(() => {
          this.formStep = "success";
        })
        .catch(err => {
          this.serverErrorMsg = err.message;
          if (err.status !== 422) {
            Sentry.captureException(err);
          }
        });
    }
  }
};
</script>

<style lang="scss" scoped>
.uc-form-body {
  @include child-spacing(top, 25px);
}

.step-header {
  text-align: left;

  &__title {
    font-size: 18px;
    font-weight: bold;
  }

  &__subtitle {
    font-size: 14px;
    color: $c-secondary-grey;
  }

  a {
    color: $c-information-blue;
  }
}

.name-fields {
  @include child-spacing(right, 15px);

  input {
    box-sizing: border-box;
    width: 100%;
    height: 45px;
  }
}

.phone-input {
  margin: 10px 0 2px;
}

.step-errors {
  color: #bf0000;
  font-size: 14px;
  text-align: left;
}

.d-none {
  display: none !important;
}
</style>
