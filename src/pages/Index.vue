<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
          v-model="search"
          @keyup.enter="getWeatherBySearch()"
          placeholder="Search"
          dark
          borderless
          >        
        <template v-slot:before>
          <q-icon 
            @click="getLocation()"
            name="my_location" />
        </template>

        <template v-slot:hint>
          Field hint
        </template>

        <template v-slot:append>
          <q-btn
          @click="getWeatherBySearch()"
            round
            dense
            flat
            icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img :src="`http://openweathermap.org/img/wn/${ weatherData.weather[0].icon }@2x.png`" >
      </div>
    </template>
    
    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar<br>Weather
        </div>
        <q-btn
          @click="getLocation()"
          class="col"
          flat>
          <q-icon left size="3em" name="my_location" />
          <div>Find My Location</div>
        </q-btn>
      </div>
    </template>
    <div class="col town"></div>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data() {
    return {
      search: '',
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: 'https://api.openweathermap.org/data/2.5/weather',
      apiKey: 'f828c3c60d3d442187f337314defc166',
    }
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night';
        }else {
          return 'bg-day';
        }
      }
    }
  },
  methods: {
    getLocation() {
      this.$q.loading.show();

      if (this.$q.platform.is.electron) {
        this.$axios.get('https://freegeoip.app/json/').then(response => {
          this.lat = response.data.latitude;
          this.lon = response.data.longitude;
          this.getWeatherByCoords();
        })
      } else {
        navigator.geolocation.getCurrentPosition( position => {
        console.log('position:' , position);
        this.lat = position.coords.latitude;
        this.lon = position.coords.longitude;
        this.getWeatherByCoords();
      })
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show();
      this.$axios(`${ this.apiUrl }?lat=${ this.lat }&lon=${ this.lon }&appid=${ this.apiKey }&units=metric`).then(response => {
        this.weatherData = response.data;
        this.$q.loading.hide();
      })
    },
    getWeatherBySearch() {
      if (this.search) {
        this.$q.loading.show();
        // console.log('getWeatherBySearch');
        // console.log('test '+ this.search);
        this.$axios(`${ this.apiUrl }?q=${ this.search }&appid=${ this.apiKey }&units=metric`).then(response => {
          // console.log('response.status: '+ response.status);
          this.weatherData = response.data;
          this.$q.loading.hide();
          // if (response.status == 404) {
          //   console.log('Testtttt');
          // }
        }).catch((error)=> {
          this.$q.loading.hide();
          this.$q.notify({
            type: 'warning',
            message: `This is a "info" type notification.`,
            position: 'top',
            color: 'warning',
          })
          console.log(error);
          // this.$q.loading.hide();
          // if (error.response.status === 404) {
          //     this.$refs.addNewTaskData.errors = error.response.data.errors;
          //     console.warn('Not good man :(');
          // }
        }); 
      }
    }
  }
}
</script>

<style lang="scss">
  .q-page {
    // background: linear-gradient(90deg, rgba(29,242,57,1) 0%, rgba(241,240,68,1) 100%);
    background: linear-gradient(94deg, rgba(204,251,255,1) 0%, rgba(239,150,197,1) 100%);
    &.bg-night {
      // background: linear-gradient(306deg, rgba(163,201,226,1) 0%, rgba(150,24,247,1) 100%);
      // background: linear-gradient(315deg, rgba(229,170,195,1) 0%, rgba(154,82,199,1) 100%);
      background: linear-gradient(144deg, rgba(194,46,208,1) 0%, rgba(95,250,224,1) 100%);
    }
    &.bg-day {
      background: linear-gradient(146deg, rgba(253,128,171,1) 0%, rgba(252,207,66,1) 100%);
    }
  }
  .degree {
    top: -44px;
  }
  .town {
    flex: 0 0 100px;
    background: url(../assets/town.png);
    background-size: contain;
    background-position: center bottom;
  }
</style>