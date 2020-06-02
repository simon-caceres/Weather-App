<template>
  <q-page class="flex column page">
    <div class="col q-pt-log q-px-md">
        <q-input 
          bottom-slots 
          v-model="search" 
          placeholder="Search"
          :dense="dense"
          dark
          >
        <template
          v-slot:prepend
          >
          <q-icon 
            name="place" 
            @click="getLocation"
            />
        </template>
        <template v-slot:append>
          <q-icon 
            name="search" 
            @click.prevent="getWeatherCity" 
            class="cursor-pointer" />
        </template>

      </q-input>
    </div>

  <template v-if="weatherData">
    <div class="col text-white text-center">
      <div class="text-h4 text-weight-Light">
        {{weatherData.sys.country}} / {{weatherData.name}} 
      </div>
      <div class="text-h6 text-weight-Light">
        {{weatherData.weather[0].description}}
      </div>
      <div class="text-h1 text-weight-thin-q-my-lg relative-position">
        <span>{{Math.round(weatherData.main.temp)}}</span>
        <span class="text-h4 relative-position degree">&deg;</span>
      </div>
    </div>

    <div class="col text-center">
      <img  :src="dataUrl" alt="" id="imgWeather">
    </div>

  </template>

  <template v-else>
    <div class="col column text-center text-white">
      <div class="col text-h2 text-weight-thin">
        World<br>Weather
      </div>
      <q-btn 
        @click="getLocation"
        class="col"
        flat>
      <q-icon left size="3em" name="my_location" />
      <div>Find my location</div>
    </q-btn>
    </div>
  </template>
    <div class="col skyline">

    </div>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',

  data () {
    return {
      search: '',
      ph: '',
      dense: false,
      weatherData: null,
      lat: null,
      long: null,
      apiKey: 'e9ab09d59edb02dae83be573fb132d30',
      dataUrl: ''
    }
  },

  methods: { 
    getLocation () {
      this.$q.loading.show()
      if (this.$q.platform.is.electron) {
        this.$axios.get('https://freegeoip.app/json/')
          .then(res => {
            this.lat = res.data.latitude;
            this.long = res.data.longitude;
            this.getWeather()
          })
        
      }
      else {
        navigator.geolocation.getCurrentPosition(position => {
          this.lat = position.coords.latitude
          this.long = position.coords.longitude
          this.getWeather()
        })
      }

    },

    getWeather() {
      this.$q.loading.show()
      this.$axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${this.lat}&lon=${this.long}&appid=${this.apiKey}&units=metric`)
      .then(response=>{
        this.weatherData = response.data
        this.dataUrl = `http://openweathermap.org/img/wn/${this.weatherData.weather[0].icon}@2x.png`
        this.$q.loading.hide()
      })
      .catch(error => {
        console.log(error)
      })
    },

    getWeatherCity() {
      this.$q.loading.show()
      this.$axios.get(`https://api.openweathermap.org/data/2.5/weather?q=${this.search}&appid=${this.apiKey}&units=metric`)
        .then(response => {
          this.weatherData = response.data
          this.dataUrl = `http://openweathermap.org/img/wn/${this.weatherData.weather[0].icon}@2x.png`
          this.$q.loading.hide()
        })
        .catch(error => {
          console.log(error)
        })
    }
  }
}
</script>

<style lang="scss">
.page {
   background: linear-gradient(to right, #24c6dc, #514a9d);
}

.degree {
  top: -44px;
}

.skyline {
  flex: 0 0 100px;
  background: url(../statics/icons/skyline.png);
  background-size: contain;
  background-position: center bottom;
}

#imgWeather {
  width: 15rem;
}
</style>