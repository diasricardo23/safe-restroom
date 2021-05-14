<template>
  <div class="container">
    <b-navbar sticky fixed="" variant="white" type="light" style="width: 100vw">
      <b-navbar-brand href="#" class="d-flex">
        <img src="../assets/logo.png" style="width: 30px; height: 30px" alt="Kitten">
        <div class="ml-2">Safe Restrooms</div>
      </b-navbar-brand>
    </b-navbar>
    <div class="flex-grow-1 d-flex flex-column align-items-center justify-content-center pt-2 pb-2">
      <div class="d-flex flex-column">
        <div v-if="!loading" class="mb-3">
          <b-form-input v-model="search" placeholder="Busque por aqui!" v-on:change="onChangeText" class="mb-2"></b-form-input>
          <div class="d-flex">
            <b-form-checkbox
              class="mr-3"
              id="checkbox-1"
              v-model="unisex"
              name="checkbox-1"
              v-on:change="onChange"
            >
              <small style="color: white">Banheiros Unisex</small>
            </b-form-checkbox>

            <b-form-checkbox
              id="checkbox-2"
              v-model="acessible"
              name="checkbox-2"
              v-on:change="onChange"
            >
              <small style="color: white">Banheiros Acessíveis</small>
            </b-form-checkbox>
          </div>
        </div>
        <div v-if="!loading && restrooms.length > 0" class="flex-grow-1">
          <div v-for="(restroom, index) in restrooms" :key="restroom.id" class="d-flex flex-column restroom-container mb-2">
            <div class="d-flex justify-content-between full-width">
              <div class="text-left overflow flex-grow-1" style="max-width: 80%">
                <div class="title">{{restroom.name}}</div>
                <div class="d-flex align-center">
                  <div class="subtitle mr-1 align-self-center">{{restroom.street}}, {{restroom.city}} - {{restroom.state}}</div>
                  <b-icon v-if="restroom.directions != ''" class="align-self-center" v-b-toggle="`collapse-${index}`" :icon="collapse[index] ? 'plus' : 'x'" variant="white" font-scale="1" @click="collapse[index] = !collapse[index]"></b-icon>
                </div>
                <b-collapse v-if="restroom.directions != ''" :id="`collapse-${index}`" class="mt-2">
                  <div class="subscription ml-3">{{restroom.directions}}</div>
                </b-collapse>
              </div>
              <country-flag :country="restroom.country" size='normal'/>
            </div>
            <div class="d-flex justify-content-between mt-2">
              <div>
                <div v-if="restroom.upvote == 0 && restroom.downvote == 0" class="ranking" style="background-color: grey">
                  <div class="ranking-text">não avaliado</div>
                </div>
                <div v-if="restroom.upvote < restroom.downvote" class="ranking" style="background-color: red" >
                  <div class="ranking-text">mal avaliado</div>
                </div>
                <div v-if="restroom.upvote > restroom.downvote" class="ranking" style="background-color: green" :id="`tooltip-success-${index}`">
                  <div class="ranking-text">bem avaliado</div>
                </div>
                <b-tooltip :target="`tooltip-success-${index}`" triggers="hover">
                  <div class="ranking-text">{{((restroom.upvote/(restroom.upvote + restroom.downvote))*100).toFixed(2)}} % aprovado</div>
                </b-tooltip>
                <b-tooltip :target="`tooltip-failure-${index}`" triggers="hover">
                  <div class="ranking-text">{{((restroom.downvote/(restroom.upvote + restroom.downvote))*100).toFixed(2)}} % não aprovado</div>
                </b-tooltip>
              </div>
              
              <div>
                <b-icon v-if="restroom.accessible" class="align-self-center" icon="shield-fill-check" variant="white" font-scale="1" ></b-icon>
                <b-icon v-if="restroom.unisex" class="align-self-center" icon="shield-shaded" variant="white" font-scale="1" ></b-icon>
              </div>
            </div>
          </div>
        </div>
        <div v-else class="d-flex flex-column">
          <b-spinner class="align-self-center" type="grow" variant="white" label="Spinning"></b-spinner>
          <small class="mt-1" style="color: white">Loading...</small>
        </div>
        
        <div v-if="!loading" class="d-flex justify-content-between">
          <b-button variant="dark" :disabled="page == 1" @click="searchBack()">Voltar</b-button>
          <b-button variant="dark" @click="searchNext()">Próxima</b-button>
        </div>
      </div>
      
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import CountryFlag from 'vue-country-flag'

export default {
  name: 'index',
  components: { CountryFlag },
  data() {
    return {
      location: null,
      gettingLocation: false,
      errorStr: null,
      restrooms: {},
      collapse: [],
      page: 1,
      unisex: false,
      acessible: false,
      search: '',
      loading: true
    }
  },
  mounted() {
    navigator.geolocation.getCurrentPosition(
     position => {
       console.log(position.coords.latitude);
       console.log(position.coords.longitude);
     },
     error => {
       console.log(error.message);
     },
  )  

    axios.get(`https://www.refugerestrooms.org/api/v1/restrooms?page=${this.page}&per_page=5&offset=0&ada=${this.acessible}&unisex=${this.unisex}&query=${this.search}`)
      .then(response => {
        console.log(response.data[0])

        response.data.map((dt, index) => {
          this.collapse[index] = true
        })
        this.restrooms = response.data
        this.loading = false
      })

  },

  methods: {
    console() {
      console.log('entrou')
    },

    searchNext() {
      this.loading = true
      this.page++ 

      var URL = '';

      if(this.search == ''){
        URL = `https://www.refugerestrooms.org/api/v1/restrooms?page=${this.page}&per_page=5&offset=0&ada=${this.acessible}&unisex=${this.unisex}&query=${this.search}`
      } else {
        URL = `https://www.refugerestrooms.org/api/v1/restrooms/search?page=${this.page}&per_page=5&offset=0&ada=${this.acessible}&unisex=${this.unisex}&query=${this.search}`
      }

      axios.get(URL)
      .then(response => {
        console.log(response.data[0])

        response.data.map((dt, index) => {
          this.collapse[index] = true
        })
        this.restrooms = response.data
        this.loading = false
      })
    },

    searchBack() {
      this.loading = true
      this.page--

      var URL = '';

      if(this.search == ''){
        URL = `https://www.refugerestrooms.org/api/v1/restrooms?page=${this.page}&per_page=5&offset=0&ada=${this.acessible}&unisex=${this.unisex}&query=${this.search}`
      } else {
        URL = `https://www.refugerestrooms.org/api/v1/restrooms/search?page=${this.page}&per_page=5&offset=0&ada=${this.acessible}&unisex=${this.unisex}&query=${this.search}`
      }

      axios.get(URL)
      .then(response => {
        console.log(response.data[0])

        response.data.map((dt, index) => {
          this.collapse[index] = true
        })
        this.restrooms = response.data
        this.loading = false
      })
    },

    onChange() {
      this.loading = true
      var URL = '';

      if(this.search == ''){
        URL = `https://www.refugerestrooms.org/api/v1/restrooms?page=${this.page}&per_page=5&offset=0&ada=${this.acessible}&unisex=${this.unisex}&query=${this.search}`
      } else {
        URL = `https://www.refugerestrooms.org/api/v1/restrooms/search?page=${this.page}&per_page=5&offset=0&ada=${this.acessible}&unisex=${this.unisex}&query=${this.search}`
      }
      
      axios.get(URL)
        .then(response => {
          
          response.data.map((dt, index) => {
            this.collapse[index] = true
          })
          this.restrooms = response.data
          this.loading = false
        })
    },

    onChangeText() {
      this.loading = true
      var URL = '';

      if(this.search == ''){
        URL = `https://www.refugerestrooms.org/api/v1/restrooms?page=${this.page}&per_page=5&offset=0&ada=${this.acessible}&unisex=${this.unisex}&query=${this.search}`
      } else {
        URL = `https://www.refugerestrooms.org/api/v1/restrooms/search?page=${this.page}&per_page=5&offset=0&ada=${this.acessible}&unisex=${this.unisex}&query=${this.search}`
      }
      
      axios.get(URL)
        .then(response => {
          
          response.data.map((dt, index) => {
            this.collapse[index] = true
          })
          this.restrooms = response.data
          this.loading = false
        })
    }
  }
}
</script>

<style>
.container {
  margin: 0;
  min-height: 100vh;
  min-width: 100vw;
  padding: 0 !important;
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
  background-color: #451775;
}

.restroom-container {
  background-color: #d0b9f7;
  padding: 10px;
  border-radius: 7px;
  width: 50vw;
}

.full-width {
  width: 100%;
}

.overflow {
  overflow-wrap: break-word;
  word-break: break-all;
}

.title {
  font-family:
    'Quicksand',
    'Source Sans Pro',
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    'Helvetica Neue',
    Arial,
    sans-serif;
  display: block;
  font-weight: bold;
  font-size: 14px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: bold;
  font-size: 12px;
  color: white;
}

.subscription {
  font-weight: bold;
  font-size: 10px;
  color: white;
}

.ranking {
  padding: 5px;
  border-radius: 5px;
}

.ranking-text {
  font-size: 10px;
  color: white;
  font-weight: bold;
}

.links {
  padding-top: 15px;
}
</style>
