<template>
  <div
    class="flex flex-col gap-3 w-full text-white p-8 rounded-2xl bg-orange-300 bg-opacity-25 justify-center items-center"
  >
    <div class="grid grid-cols-5 w-full text-center">
      <div v-for="hour in firstHalf" :key="hour.time_epoch" class="text-sm">
        <p>{{ hour.label }}</p>
        <p><i class="fa-solid fa-cloud"></i> {{ hour.temp_c }}&#176;</p>
      </div>
    </div>
    <div class="border-b-2 border-white w-full p-2 mb-2"></div>
    <div class="grid grid-cols-5 w-full text-center">
      <div v-for="hour in secondHalf" :key="hour.time_epoch" class="text-sm">
        <p>{{ hour.label }}</p>
        <p><i class="fa-solid fa-cloud"></i> {{ hour.temp_c }}&#176;</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      hourlyForecast: [],
      firstHalf: [],
      secondHalf: []
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
      const url = `http://api.weatherapi.com/v1/forecast.json?key=${apiKey}&q=${lat},${lon}&hours=12&aqi=no`

      axios
        .get(url)
        .then((response) => {
          const weatherData = response.data
          this.location = `${weatherData.location.name}, ${weatherData.location.region}, ${weatherData.location.country}`
          this.temprature = weatherData.current.temp_c
          this.feelslike = weatherData.current.feelslike_c
          this.weatherCondition = weatherData.current.condition.text
          this.hourlyForecast = weatherData.forecast.forecastday[0].hour

          // current time
          const now = new Date()
          const nowHour = now.getHours()

          this.hourlyForecast = this.hourlyForecast.map((hour) => {
            const hourDate = new Date(hour.time)
            const hourFormatted = this.formatTime(hourDate)
            return { ...hour, label: hourFormatted }
          })

          // index of the current hour in the forecast data
          const currentIndex = this.hourlyForecast.findIndex((hour) => {
            const hourDate = new Date(hour.time)
            return hourDate.getHours() === nowHour
          })

          const upcomingHours = this.hourlyForecast.slice(currentIndex + 1)
          this.firstHalf = [{ label: 'Now', temp_c: this.temprature }].concat(
            upcomingHours.slice(0, 4)
          )
          if (this.firstHalf.length < 5) {
            this.firstHalf = this.firstHalf.concat(
              this.generateCurrentWeatherHours(5 - this.firstHalf.length)
            )
          }
          this.secondHalf = upcomingHours.slice(4, 9)
          if (this.secondHalf.length < 5) {
            this.secondHalf = this.secondHalf.concat(
              this.generateCurrentWeatherHours(5 - this.secondHalf.length)
            )
          }

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
    formatTime(date) {
      let hours = date.getHours()
      const period = hours >= 12 ? 'PM' : 'AM'
      hours = hours % 12 || 12
      return `${hours} ${period}`
    },
    generateCurrentWeatherHours(count) {
      const currentHourData = {
        label: this.formatTime(new Date()),
        temp_c: this.temprature
      }
      return Array(count).fill(currentHourData)
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
