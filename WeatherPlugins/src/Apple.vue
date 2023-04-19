<template>
    <div>
      <v-mini-weather class="weather" size="small" type="oneline" color="000000"></v-mini-weather>
    </div>
</template>

<script>
import { vMiniWeather } from '../npm/index.umd.min'
import '../npm/index.css'

export default {
  name: 'Apple',
  components: {
    vMiniWeather
  }
}
</script>

