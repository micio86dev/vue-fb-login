<template>
  <div class="container" :ref="ref">
    <div id="fb-root"></div>
    <div class="fb-login-button" @click="buttonClicked" data-size="large" data-button-type="login_with" data-auto-logout-link="true" data-use-continue-as="false"></div>
  </div>
</template>

<script async defer crossorigin="anonymous" :src="`https://connect.facebook.net/it_IT/sdk.js#xfbml=1&version=${version}&appId=${appId}&autoLogAppEvents=1`"></script>

<script>
import { loadFbSdk, getLoginStatus, fbLogout, fbLogin } from './helpers.js'
import icon from './icon.png'
export default {
  name: 'facebook-login',
  data() {
    return {
      isWorking: false,
      isConnected: false,
      icon
    };
  },
  props: {
    ref: {
      type: String,
      required: false
    },
    appId: {
      type: String,
      required: true
    },
    version: {
      type: String,
      default: 'v3.3'
    },
    loginOptions: {
      type: Object,
      default: function() {
        return {
          scope: 'email'
        }
      }
    }
  },
  computed: {
  },
  methods: {
    buttonClicked() {
      this.$emit('click')
      if (this.isConnected) {
        this.logout();
      } else {
        this.login();
      }
    },
    logout() {
      this.isWorking = true;
      fbLogout()
        .then(response => {
          this.isWorking = false;
          this.isConnected = false;
          this.$emit('logout', response)
        });
    },
    login() {
      this.isWorking = true;
      fbLogin(this.loginOptions)
        .then(response => {
          if (response.status === 'connected') {
            this.isConnected = true;
          } else {
            this.isConnected = false;
          }
          this.isWorking = false;
          this.$emit('login', {
            response,
            FB: window.FB
          });
        });
    }
  },
  mounted() {
    this.isWorking = true;
    loadFbSdk(this.appId, this.version)
      .then(loadingResult => {
      })
      .then(() => getLoginStatus())
      .then(response => {
        if (response.status === 'connected') {
          this.isConnected = true;
        }
        this.isWorking = false;
        /** get-initial-status to be depcreated on next major version */
        this.$emit('get-initial-status', response);
        this.$emit('sdk-loaded', {
          isConnected: this.isConnected,
          FB: window.FB
        })
      });
  }
}
</script>
