<template>
  <div
    class="w-full flex flex-col bg-orange-300 items-center text-orange-500 justify-center rounded-2xl p-5 gap-5"
  >
    <p class="text-base font-medium">
      <span class="mr-2">Today</span> <i class="fa-solid fa-chevron-down"></i>
    </p>
    <div class="text-4xl lg:text-6xl font-bold flex gap-4 items-center">
      <img :src="iconUrl" alt="Weather Icon" class="w-12 h-12 lg:w-20 lg:h-20" />
      <p>{{ temprature }}&#176;</p>
    </div>
    <p class="text-sm font-medium">{{ weatherCondition }}</p>
    <div class="flex flex-col gap-3 justify-center items-center text-xs">
      <p>{{ location }}</p>
      <p>
        {{
          new Date().toLocaleDateString('en-GB', {
            day: '2-digit',
            month: 'short',
            year: 'numeric'
          })
        }}
      </p>
      <p>
        Feels like {{ feelslike }} | Sunset
        {{ sunset }}
      </p>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      location: '',
      sunset: '',
      temprature: '',
      feelslike: '',
      weatherCondition: '',
      iconUrl: '' // URL for the weather icon
    }
  },
  mounted() {
    this.getLocation()
  },
  methods: {
    getLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(this.fetchWeatherData, this.showError)
      } else {
        alert('Geolocation is not supported by this browser.')
      }
    },
    fetchWeatherData(position) {
      const lat = position.coords.latitude
      const lon = position.coords.longitude

      const apiKey = 'd3a107967a684f4aa6383511241907'
      const url = `http://api.weatherapi.com/v1/current.json?key=${apiKey}&q=${lat},${lon}&aqi=no`

      axios
        .get(url)
        .then((response) => {
          const weatherData = response.data
          this.location = `${weatherData.location.name}, ${weatherData.location.region}, ${weatherData.location.country}`
          this.temprature = weatherData.current.temp_c
          this.feelslike = weatherData.current.feelslike_c
          this.weatherCondition = weatherData.current.condition.text
          this.iconUrl = `http:${weatherData.current.condition.icon}` // Set the icon URL

          const astronomyUrl = `http://api.weatherapi.com/v1/astronomy.json?key=${apiKey}&q=${lat},${lon}&dt=${new Date().toISOString().split('T')[0]}`

          return axios.get(astronomyUrl)
        })
        .then((response) => {
          const astronomyData = response.data
          const sunsetTime = astronomyData.astronomy.astro.sunset
          const [time, period] = sunsetTime.split(' ')
          let [hours, minutes] = time.split(':')

          if (period === 'PM' && hours !== '12') {
            hours = String(parseInt(hours) + 12)
          } else if (period === 'AM' && hours === '12') {
            hours = '00'
          }

          this.sunset = `${hours}:${minutes}`
        })
        .catch((error) => {
          console.error(error)
        })
    },
    showError(error) {
      switch (error.code) {
        case error.PERMISSION_DENIED:
          alert('User denied the request for Geolocation.')
          break
        case error.POSITION_UNAVAILABLE:
          alert('Location information is unavailable.')
          break
        case error.TIMEOUT:
          alert('The request to get user location timed out.')
          break
        case error.UNKNOWN_ERROR:
          alert('An unknown error occurred.')
          break
      }
    }
  }
}
</script>
