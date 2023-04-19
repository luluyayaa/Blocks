<template>
  <div class="v-weather" @click="getWeather"
       :style="`color: rgb(${convartedColor[0]}, ${convartedColor[1]}, ${convartedColor[2]})`">
      <span>{{ position.area }} / {{ weather.weather }} / {{ weather.temp }}℃</span>
    </div>
</template>

<script>
import axios from 'axios'
import Lottie from 'lottie-web'
import weatherIcon from './iconJson'

export default {
  name: "vMiniWeather",
  props: {
    size: {
      type: String,
      default: "small",
      validator: value => {
        return ['small', 'normal'].includes(value)
      }
    },
    type: {
      type: String,
      default: "oneline",
      validator: value => {
        return ['oneline', 'multiline'].includes(value)
      }
    },
    color: {
      type: String,
      default: "000000"
    },
    iconSize: {
      type: Number,
      default: 100
    },
    url: {
      type: String,
      default: 'https://apia.aidioute.cn/weather/index.php'
    }
  },
  data: () => ({
    convartedColor: [],
    location: null,   //定位功能获取的经纬度
    position: {},   //经纬度查询获得的位置信息
    weather: {},
    weatherIconAnimation: null,
    timer: null,
    errorMsg: ""
  }),
  watch: {
    color(data) {
      this.convartColor()
      Lottie.destroy()
      if (this.weather.hasOwnProperty('code')) {
        this.showIcon()
      } else {
        this.getLocation()
      }
    }
  },
  mounted() {
    this.convartColor()
    this.getLocation()
    this.timer = setInterval(() => {
      this.getLocation()
    }, 30 * 60 * 1000)
  },
  beforeDestroy() {
    clearInterval(this.timer)
  },
  methods: {
    convartColor() {
      const colorTemp = /([a-zA-Z0-9]{2})([a-zA-Z0-9]{2})([a-zA-Z0-9]{2})/.exec(this.color)
      this.convartedColor = [
        parseInt(`0x${colorTemp[1]}`),
        parseInt(`0x${colorTemp[2]}`),
        parseInt(`0x${colorTemp[3]}`)
      ]
    },
    getLocation() {
      if (typeof window !== "undefined" && window.navigator.geolocation) {
        window.navigator.geolocation.getCurrentPosition(
            (position) => {
              this.location = {
                latitude: position.coords.latitude.toFixed(6),
                longitude: position.coords.longitude.toFixed(6)
              }
              this.getWeather()
            },
        )
      }
    },
    async getWeather() {
      let url = ''
      if(this.location) {
        url = `${this.url}?location_type=1&lat=${this.location.latitude}&lng=${this.location.longitude}`
      } else {
        url = `${this.url}?location_type=0`
      }
      try {
        const weather = await axios.get(url)
        if (weather.status === 200 && weather.data.error === 0) {
          this.weather = weather.data.data.weather
          this.position = weather.data.data.location
          if (weather.data.data.location.error_msg !== '成功。') {
            message.warning(weather.data.data.location.error_msg)
          }
          this.showIcon()
        } else {
          message.error("网络错误!")
        }
      } catch (err) {
        console.log(err)
        message.error("未知错误!")
      }
    },
    showIcon() {
      const svgContainer = this.$refs.svgContainer
      if (this.weatherIconAnimation) {
        this.weatherIconAnimation.destroy()
      }
      this.weatherIconAnimation = Lottie.loadAnimation({
        wrapper: svgContainer,
        animType: 'svg',
        loop: true,
        animationData: weatherIcon[this.weather.weather](this.convartedColor[0] / 255, this.convartedColor[1] / 255, this.convartedColor[2] / 255)
      });
    }
  }
}
</script>

<!--<style scoped>-->
<!--p {-->
<!--  margin: 0;-->
<!--  padding: 0;-->
<!--}-->

<!--.v-weather {-->
<!--  display: inline-block;-->
<!--  cursor: pointer;-->
<!--  user-select: none;-->
<!--}-->


<!--.v-weather> .is-small>{-->
<!--  position: absolute;-->
<!--  top: 7px;-->
<!--  left: 10px;-->
<!--  width: 30px;-->
<!--  height: 30px;-->
<!--}-->


<!--.v-weather> .is-small.is-multiline> p {-->
<!--  height: 24px;-->
<!--  line-height: 24px;-->
<!--  font-size: 16px;-->
<!--}-->


<!--.v-weather> .is-normal.is-multiline> .map div {-->
<!--  display: inline-block;-->
<!--  width: 24px;-->
<!--  height: 24px;-->
<!--  margin-left: -14px;-->
<!--  vertical-align: middle;-->
<!--}-->

<!--.v-weather> .is-normal.is-multiline> p {-->
<!--  height: 20px;-->
<!--  line-height: 20px;-->
<!--  font-size: 16px;-->
<!--}-->
<!--</style>-->
