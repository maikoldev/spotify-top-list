<template>
  <div>
    <h1>Spotify Top List</h1>
    <button v-if="!isAuth" @click="authUser()">Autorizar</button>
  </div>
</template>

<script>
const qs = require('qs');
const axios = require('axios');
export default {
  data() {
    return {
      isAuth: false,
      authParams: {
        client_id: '7ce263a70ae8490192b3e50b92ee9376',
        client_secret: '02560420ffbe4078ad28b8875ef6c832',
        redirect_uri: process.env.VUE_APP_BASE_URL,
        response_type: 'code',
        scope: 'user-read-private user-read-email user-top-read',
        state: this.generateRandomString(16)
      }
    };
  },
  created() {
    this.makeAuthToken();
  },
  methods: {
    generateRandomString(length) {
      let text = '',
        possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';

      for (let i = 0; i < length; i++) {
        text += possible.charAt(Math.floor(Math.random() * possible.length));
      }
      return text;
    },
    makeAuthToken() {
      const queryString = window.location.search;
      const code = new URLSearchParams(queryString).get('code');
      if (code == null) {
        return;
      } else {
        const headerCode = new Buffer(
          `${this.authParams.client_id}:${this.authParams.client_secret}`
        ).toString('base64');

        const params = {
          grant_type: 'authorization_code',
          code: code,
          redirect_uri: this.authParams.redirect_uri
        };

        axios
          .post('https://accounts.spotify.com/api/token', qs.stringify(params), {
            headers: {
              'Content-Type': 'application/x-www-form-urlencoded',
              Authorization: `Basic ${headerCode}`
            }
          })
          .then(res => {
            localStorage.setItem('access_token', res.data.access_token);
            localStorage.setItem('refresh_token', res.data.refresh_token);
            this.$router.push({name: 'TopList', params: {type: 'artists'}});
          })
          .catch(error => {
            console.log(error);
          });
      }
    },
    authUser() {
      const url = 'https://accounts.spotify.com/authorize?';
      window.location.href = `${url}${qs.stringify(this.authParams)}`;
    }
  }
};
</script>

<style lang="scss" scoped></style>
