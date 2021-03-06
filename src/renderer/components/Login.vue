<!--
Copyright (C) 2018-2020 AI Atelier Ltd.

This file is part of Invizi.

Invizi is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or (at
your option) any later version.

Invizi is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.
You should have received a copy of the GNU General Public License
along with Invizi.  If not, see <https://www.gnu.org/licenses/>.
-->
<template>
  <section style="height: 30vh; padding: 10rem; text-align: center;">
    <LogoTitle style="margin-bottom: 10rem;"/>
    <div class="col-lg-6 offset-lg-2" style="margin-left: 20%;">
      <div>
        <form v-if="hasPassword">
          <div class="form-row" v-if="!decryptingProgress">
            <v-text-field
              @keyup.enter="login"
              id="orangeForm-pass"
              label="Password"
              type="password"
              single-line
              v-model="password"
              autofocus @input="onPasswordChange()"
            ></v-text-field>

            <div class="md-form form-group text-center">
              <button type="button" class="btn waves-effect btn--large waves-light" @click="login()">Login</button>
            </div>
          </div>
          <v-progress-linear
            v-model="decryptingProgress"
            v-if="decryptingProgress"
            color="primary-color"
          ></v-progress-linear>
          <div class="md-form form-group text-left">
            <span class="red-text">{{errorMsg}}</span>

            <span class="" v-if="decryptingProgress"><em>Decrypting</em></span>
          </div>
        </form>
        <div v-if="!hasPassword">
          <div class="md-form">
            <v-text-field
              @keyup.enter="newLogin"
              id="orangeForm-pass"
              label="New Password"
              type="password"
              single-line
              v-model="newPassword"
              autofocus
              @blur="clean()"
            ></v-text-field>
          </div>
          <div class="md-form">
            <v-text-field
              @keyup.enter="newLogin"
              id="orangeForm-pass"
              label="Confirm Password"
              type="password"
              single-line
              v-model="confirmPassword"
              @blur="clean()"
            ></v-text-field>
          </div>

          <div class="text-left">
            <p class="red-text">{{errorMsg}}</p>
          </div>

          <div class="text-left" style="display: flex; margin-bottom: 40px;">
            <button type="button" class="btn btn-primary waves-effect
                          waves-light" @click="newLogin()" style="margin-left: 0;">Create Account</button>
          </div>
          <div class="text-left">
            <em class="text-small">If you lose or forget your password, Invizi cannot recover your data. Remember that passwords are case-sensitive.</em>
          </div>
        </div>
      </div>

      <div>
      </div>
    </div>
  </section>
</template>

<script>
  import UserManager from '@/components/UserManager'
  import OnlineAccountClient from '@/models/OnlineAccountClient'
  import LogoTitle from '@/components/LogoTitle'

  export default {
    name: 'settings',
    components: {
      LogoTitle
    },
    data () {
      return {
        password: undefined,
        decryptingProgress: undefined,
        newPassword: undefined,
        confirmPassword: undefined,
        hasPassword: true,
        errorMsg: undefined
      }
    },
    methods: {
      onPasswordChange () {
        this.errorMsg = ''
      },
      login () {
        this.decryptingProgress = 30
        UserManager.login(this.password).then((result) => {
          this.decryptingProgress = 60
          if (result) {
            // Check if he has any accounts
            OnlineAccountClient.all().then((accounts) => {
              this.decryptingProgress = 100
              if (accounts && accounts.length > 0) {
                this.$router.push('main-dashboard')
              } else {
                this.$router.push({ path: '/accounts/local1' })
              }
            })
          } else {
            this.errorMsg = 'Incorrect password'
            this.decryptingProgress = undefined
          }
        })
      },
      newLogin () {
        // Check both password are the same
        if (this.newPassword && this.newPassword === this.confirmPassword) {
          this.password = this.newPassword
          this.login()
        } else {
          this.errorMsg = 'The two passwords do no match.'
        }
      },
      clean () {
        this.errorMsg = undefined
      }
    },
    mounted () {
      UserManager.hasPassword().then((result) => {
        this.hasPassword = result
      })
    }
  }
</script>

<style scoped>

 .subtitles h2 {
   font-size: 1rem;
   text-align: left;
 }
</style>
