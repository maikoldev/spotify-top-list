<template>
  <b-container class="py-5">
    <b-row>
      <b-col cols="12">
        <h1 class="title display-4 font-weight-bold text-white text-md-left mb-5">
          Artistas mas escuchados
        </h1>
      </b-col>
      <b-col cols="12" md="3" class="mb-5" v-for="(artist, key) in artists" :key="artist.id">
        <div class="wrap-img mb-4">
          <a :href="artist.external_urls.spotify">
            <div class="overlay-img"></div>
            <b-img-lazy :src="artist.images[1].url" :alt="artist.name" rounded="circle" fluid />
          </a>
        </div>
        <b-link class="artist-name text-white" :href="artist.external_urls.spotify">
          <small class="font-weight-bold">{{ `#${key + 1}: ${artist.name}` }}</small>
        </b-link>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
const qs = require('qs');
const axios = require('axios');
export default {
  name: 'top-list',
  props: {
    type: {
      default: 'artists'
    }
  },
  data() {
    return {
      artists: null,
      authParams: {
        client_id: '7ce263a70ae8490192b3e50b92ee9376',
        client_secret: '02560420ffbe4078ad28b8875ef6c832'
      }
    };
  },
  created() {
    this.getTopArtist();
  },
  methods: {
    getTopArtist() {
      axios
        .get(`https://api.spotify.com/v1/me/top/${this.type}`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('access_token')}`
          }
        })
        .then(res => {
          this.artists = res.data.items;
        })
        .catch(error => {
          this.refreshToken();
          console.log(error);
        });
    },
    refreshToken() {
      const headerCode = new Buffer(
        `${this.authParams.client_id}:${this.authParams.client_secret}`
      ).toString('base64');

      const params = {
        grant_type: 'refresh_token',
        refresh_token: localStorage.getItem('refresh_token')
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
        })
        .catch(error => {
          console.log(error);
        });
    }
  }
};
</script>

<style lang="scss" scoped>
.title {
  @media (max-width: 768px) {
    font-size: 2.5rem;
  }
}

.wrap-img {
  padding-top: 100%;
  position: relative;

  .overlay-img {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    z-index: 99;
    border-radius: 50%;

    &:hover {
      background: rgba(0, 0, 0, 0.5);
    }
  }

  img {
    position: absolute;
    top: 0;
    left: 0;
    object-fit: cover;
    width: 100%;
    height: 100%;
  }
}

.artist-name {
  padding-bottom: 3px;

  &:hover {
    border-bottom: 1px solid;
    text-decoration: none;
  }
}
// .card {
//   &.lift {
//     text-decoration: none;
//     color: inherit;
//   }
//   .card-img-top,
//   .card-img {
//     @media (min-width: 768px) {
//       height: 350px;
//     }
//     object-fit: cover;
//   }
// }
</style>
