<template>
  <div class="section">
    <div class="columns is-centered">
      <div class="panel" style="width: 350px">
        <p class="panel-heading">Sign up</p>
        <div class="panel-block">
          <div class="field">
            <label class="label">Username</label>
            <div class="control has-icons-left">
              <input
                type="text"
                :class="`input ${$v.user.username.$invalid ? 'is-danger' : ''}`"
                placeholder="Your username"
                v-model.trim="user.username"
              />
              <span class="icon is-small is-left">
                <i class="fas fa-user"></i>
              </span>
            </div>
            <p v-if="!$v.user.username.required" class="help is-danger">
              Username is required
            </p>
            <p v-if="!$v.user.username.isUnique" class="help is-danger">
              This username is not unique
            </p>
          </div>
        </div>
        <div class="panel-block">
          <div class="field">
            <label class="label">First name</label>
            <div class="control">
              <div class="control has-icons-left">
                <input
                  type="text"
                  class="input"
                  placeholder="Your last name"
                  v-model.trim="user.firstName"
                />
                <span class="icon is-small is-left">
                  <i class="fas fa-user"></i>
                </span>
              </div>
            </div>
            <label class="label">Last name</label>
            <div class="control">
              <div class="control has-icons-left">
                <input
                  type="text"
                  class="input"
                  placeholder="Your first name"
                  v-model.trim="user.lastName"
                />
                <span class="icon is-small is-left">
                  <i class="fas fa-user"></i>
                </span>
              </div>
            </div>
          </div>
        </div>
        <div class="panel-block">
          <div class="field">
            <label class="label">Email</label>
            <div class="control has-icons-left">
              <input
                type="email"
                :class="`input ${$v.user.email.$invalid ? 'is-danger' : ''}`"
                placeholder="Your email"
                v-model.trim="user.email"
              />
              <span class="icon is-small is-left">
                <i class="fas fa-envelope" />
              </span>
            </div>
            <p v-if="!$v.user.email.required" class="help is-danger">
              Email is required
            </p>
            <p v-if="!$v.user.email.isUnique" class="help is-danger">
              This email is not unique
            </p>
            <p v-if="!$v.user.email.email" class="help is-danger">
              Not an email
            </p>
          </div>
        </div>
        <div class="panel-block">
          <div class="field">
            <label class="label">Password</label>
            <div class="control has-icons-left">
              <input
                type="password"
                :class="`input ${$v.user.password.$invalid ? 'is-danger' : ''}`"
                placeholder="Your password"
                v-model.trim="user.password"
              />
              <span class="icon is-small is-left">
                <i class="fas fa-lock"></i>
              </span>
            </div>
            <p v-if="!$v.user.password.required" class="help is-danger">
              Password is required
            </p>
            <p v-if="!$v.user.password.minLength" class="help is-danger">
              Needs to be at least
              {{ $v.user.password.$params.minLength.min }} charachters long
            </p>
            <p v-if="!$v.user.password.maxLength" class="help is-danger">
              Can't be more than
              {{ $v.user.password.$params.maxLength.max }} charachters long
            </p>
            <p v-if="!$v.user.password.alphaNum" class="help is-danger">
              Needs to be alphanumeric
            </p>
            <label class="label">Repeat password</label>
            <div class="control has-icons-left">
              <input
                type="password"
                :class="`input ${
                  $v.user.repeatPassword.$invalid ? 'is-danger' : ''
                }`"
                placeholder="Repeat your passowrd"
                v-model.trim="user.repeatPassword"
              />
              <span class="icon is-small is-left">
                <i class="fas fa-lock"></i>
              </span>
            </div>
            <p v-if="!$v.user.repeatPassword.required" class="help is-danger">
              Repeat the password
            </p>
            <p
              v-if="!$v.user.repeatPassword.sameAsPassword"
              class="help is-danger"
            >
              Needs to be the same as password
            </p>
          </div>
        </div>

        <div class="panel-block">
          <div class="field">
            <div class="control">
              <a
                class="button is-primary w-100"
                :disabled="$v.user.$invalid || uploadProgressBar.isUploading"
                @click="registerClicked()"
                >Register</a
              >
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  required,
  minLength,
  alphaNum,
  email,
  sameAs,
  maxLength,
} from "vuelidate/lib/validators";
import { mapActions } from "vuex";

export default {
  name: "SignUp",
  data() {
    return {
      user: {
        username: "",
        password: "",
        repeatPassword: "",
        email: "",
        firstName: null,
        lastName: null,
        picture: null,
      },
      picture: null,
      notificationModal: {
        isVisible: false,
        modalColor: "is-success",
        title: "",
        text: "",
      },
      uploadProgressBar: {
        isUploading: false,
        uploadProgress: 0,
      },
    };
  },
  validations: {
    user: {
      username: {
        required,
        isUnique() {
          return true;
        },
      },
      password: {
        required,
        minLength: minLength(8),
        maxLength: maxLength(64),
        alphaNum,
      },
      repeatPassword: {
        required,
        sameAsPassword: sameAs("password", this),
      },
      email: {
        required,
        email,
        isUnique() {
          return true;
        },
      },
    },
  },
  methods: {
    ...mapActions(["signUp"]),

    registerClicked() {
      console.log("register");
      if (!this.$v.user.$invalid) {
        this.signUp(this.user);
      }
      // this.$router.push('/dashboard')
    },

    pictureSelected(e) {
      this.picture = e.target.files[0];
    },
    setUploadProgress: function (uploadEvent) {
      this.uploadProgressBar.uploadProgress = Math.round(
        (uploadEvent.loaded / uploadEvent.total) * 100
      );
    },
    successModalClosed() {
      this.clearForm();
      this.$router.push("/signIn");
    },
    clearForm() {
      this.picture = null;
      var userToClear = this.user;
      userToClear.username = "";
      userToClear.email = "";
      userToClear.password = "";
      userToClear.repeatPassword = "";
      userToClear.firstName = "";
      userToClear.lastName = "";
    },
  },
};
</script>

<style scoped>
.w-100,
.field {
  width: 100%;
}
.h-100perc {
  height: 100%;
}
</style>