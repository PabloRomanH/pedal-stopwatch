<template>
  <div>
    <h1>Stopwatch</h1>
    <div class="time">{{ time }}</div>
    <div>
      <button v-shortkey.once="['s']" @shortkey="start()" v-on:click="start()" :disabled="running">Start (S)</button>
      <button v-shortkey.once="['b']" @shortkey="newLap()" v-on:click="newLap()" :disabled="!running">Lap (B)</button>
      <button v-shortkey.once="['t']" @shortkey="stop()" v-on:click="stop()" :disabled="!running">Stop (T)</button>
      <button v-shortkey.once="['r']" @shortkey="reset()" v-on:click="reset()" :disabled="!finished">Reset (R)</button>
    </div>
    <ul>
      <li v-for="(lap, index) in laps">{{ displayLap(lap, index > 0 ? laps[index - 1] : null) }}</li>
      <li v-if="finished">End: {{ displayLap(end, laps[laps.length - 1]) }}</li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'Stopwatch',
  data () {
    return {
      laps: [],
      end: null,
      beg: null,
      running: false,
      finished: false,
      totalTime: 0,
      interval: null
    }
  },
  computed: {
    time() {
      if (!this.beg) return '0:00:00'

      let [hours, minutes, seconds] = this.calculateHMS(this.totalTime)
      return this.timeText(hours, minutes, seconds)
    }
  },
  methods: {
    start() {
      this.running = true
      this.beg = new Date()
      this.interval = setInterval(() => {
        let now = new Date()
        this.totalTime = Math.floor(now.valueOf() - this.beg.valueOf())
        }, 1000)
      this.finished = false
    },
    newLap() {
      this.laps.push(new Date())
    },
    stop() {
      this.running = false
      this.finished = true
      this.end = new Date()
      clearInterval(this.interval)
    },
    reset() {
      this.running = false
      this.finished = false
      this.beg = null
      this.end = null
      this.totalTime = 0
      this.laps = []
      clearInterval(this.interval)
    },
    displayLap(lapTime, prevLapTime) {
      let [hours, minutes, seconds] = this.calculateHMSinterval(this.beg.valueOf(), lapTime.valueOf())

      if (!prevLapTime) {
        let timeString = this.timeText(hours, minutes, seconds)
        return `Time: ${timeString} - Lap: ${timeString}`
      } else {
        let [laphours, lapminutes, lapseconds] = this.calculateHMSinterval(prevLapTime.valueOf(), lapTime.valueOf())

        return `Time: ${this.timeText(hours, minutes, seconds)} - Lap: ${this.timeText(laphours, lapminutes, lapseconds)}`
      }
    },
    calculateHMSinterval(first, second) {
      return this.calculateHMS(second.valueOf() - first.valueOf())
    },
    calculateHMS(ms) {
      let seconds = Math.floor(ms / 1000)

      let hours = Math.floor(seconds / 60 / 60)
      seconds = seconds - hours * 60 * 60
      let minutes = Math.floor(seconds / 60)
      seconds = seconds - minutes * 60

      return [hours, minutes, seconds]
    },
    timeText(hours, minutes, seconds) {
      return `${hours}:${this.padTime(minutes)}:${this.padTime(seconds)}`
    },
    padTime(num) {
      var s = num + "";
      if (s.length < 2) s = "0" + s;
      return s;
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
a {
  color: #42b983;
}

.time {
  margin-bottom: 20px;
  font-size: 30pt;
}

button {
  padding: 10px;
}
</style>
