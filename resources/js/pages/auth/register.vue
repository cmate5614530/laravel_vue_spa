<template>
  <div class="row">
    <div class="col-lg-8 m-auto">
      <div id="first">
        <div class="myform form ">
          <div class="logo mb-3">
            <div class="col-md-12 text-left row">
              <h1 class="text-middle">
                Welcome to
              </h1><img src="logo.png" algin="left" style="padding:0px 10px;margin:auto;max-width:300px">
            </div>
            <div class="col-md-12 text-left row">
              <h5>Create your plany account</h5>
            </div>
          </div>
          <div v-if="mustVerifyEmail" :title="$t('register')">
            <div class="alert alert-success" role="alert">
              {{ $t('verify_email_address') }}
            </div>
          </div>
          <div v-else :title="$t('register')">
            <form @submit.prevent="register" @keydown="form.onKeydown($event)">
              <div class="form-group">
                <label for="exampleInputEmail1">Company name</label>
                <input v-model="form.company_name" :class="{ 'is-invalid': form.errors.has('company_name') }" class="form-control" type="text" name="companyname">
                <has-error :form="form" field="company_name" />
                <div class="text-right">
                  <input
                    v-if="domain.edit" v-model="form.domain" v-focus
                    class="main tx-tfm-lower"
                    type="text"
                    name="company_name"
                    :class="{ 'is-invalid': form.errors.has('domain') }"
                    @blur="domain.edit = false; $emit('update')"
                    @keyup.enter="domain.edit=false; $emit('update')"
                  >
                  <div v-else>
                    <a class="main tx-tfm-lower" :href="`https://${form.domain ? form.domain : form.company_name}.plany.io`">{{ form.domain ? form.domain :form.company_name }}.plany.io</a>
                    <fa class="main" :icon="['fas', 'edit']" @click="domain.edit = true;" />
                  </div>
                  <has-error :form="form" field="company_name" />
                </div>
                <div v-show="domainError" class="text-danger">
                  <small>{{domainErrorMessage}}</small>
                </div>
              </div>
              <div class="form-group">
                <label for="exampleInputEmail1">Email address</label>
                <input v-model="form.email" :class="{ 'is-invalid': form.errors.has('email') }" class="form-control" type="text" name="email">
                <has-error :form="form" field="email" />
              </div>
              <div class="form-group">
                <label for="exampleInputEmail1">Set a password</label>
                <input v-model="form.password" :class="{ 'is-invalid': form.errors.has('password') }" class="form-control" type="password" name="password">
                <has-error :form="form" field="password" />
              </div>
              <div class="form-group">
                <div class="text-center">
                  <v-button :loading="form.busy" class="btn btn-block mainbtn">
                    {{ $t('signup') }}
                  </v-button>
                </div>
              </div>
              <div class="form-group">
                <p class="text-center">
                  By sign up your account you agree with plany's <a href="#">Privacy Policy</a> and <a href="#">Terms of use</a>.
                </p>
              </div>
              <div class="">
                <div class="login-or">
                  <hr class="hr-or">
                  <span class="span-or">or</span>
                </div>
              </div>
              <login-with-google :domain="domainValidated" />
              <div class="form-group">
                <p class="text-left">
                  Already have an account? <a id="signup" href="/login">Sign in</a>
                </p>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Form from 'vform'
import LoginWithGithub from '~/components/LoginWithGithub'
import LoginWithGoogle from '~/components/LoginWithGoogle'

import { library } from '@fortawesome/fontawesome-svg-core'
import { faEdit } from '@fortawesome/free-solid-svg-icons'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import * as axios from "axios";

library.add(faEdit)

export default {
  middleware: 'guest',

  components: {
    LoginWithGithub,
    LoginWithGoogle
  },

  metaInfo () {
    return { title: this.$t('register') }
  },
  directives: {
    focus: {
      inserted (el) {
        el.focus()
      }
    }
  },

  data: () => ({
    form: new Form({
      company_name: '',
      domain: '',
      email: '',
      password: ''
    }),
    domain: { edit: false },
    mustVerifyEmail: false,
    domainError: false,
    domainErrorMessage: null
  }),

  computed: {
    domainValidated () {
      return this.domainError ? null : this.form.domain
    }
  },

  watch: {
    'form.domain': {
      handler (newVal, oldVal) {
        if (newVal.length > 1) {
          this.domainError = false
          this.domainErrorMessage = null
        }
      }
    },
    'domain.edit': {
      handler (editing, wasEditing) {
        if (wasEditing && !editing) {
          this.verifyDomain()
        }
      }
    }
  },

  methods: {
    async register () {
      // Register the user.
      const { data } = await this.form.post('/api/register')

      // Must verify email fist.
      if (data.status) {
        this.mustVerifyEmail = true
      } else {
        // Log in the user.
        const { data: { token } } = await this.form.post('//' + data.base_url + '/api/login')

        window.location.href = '//' + data.base_url + '/token/' + token
      }
    },
    async verifyDomain () {
      let { data } = await axios.post('/api/domain/verify', { domain: this.form.domain })
      if (data.available) {
        this.domainError = false
        this.domainErrorMessage = null
      } else {
        this.domainError = true
        this.domainErrorMessage = 'Domain is already taken.'
      }
    }
  }
}
</script>
