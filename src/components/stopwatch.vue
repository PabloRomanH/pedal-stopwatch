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
      <li v-for="(lap, index) in laps" :key="index">{{ displayLap(lap, index > 0 ? laps[index - 1] : null) }}</li>
      <li v-if="finished">End: {{ displayLap(end, laps[laps.length - 1]) }}</li>
    </ul>
    <button v-if="finished" v-on:click="download($event)">Download .csv</button>
  </div>
</template>

<script>
import { saveAs } from 'file-saver';

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
  created () {
    const dataString = window.localStorage.getItem('stopwatch')

    if (dataString === null) {
      return
    }

    const data = JSON.parse(dataString)
    
    this.laps = data.laps.map(lap => new Date(lap))
    this.end = data.end && new Date(data.end)
    this.beg = data.beg && new Date(data.beg)
    this.running = data.running
    this.finished = data.finished

    let now = new Date()
    this.totalTime = Math.floor(now.valueOf() - this.beg.valueOf())

    if (this.running) {
      this.interval = setInterval(() => {
        let now = new Date()
        this.totalTime = Math.floor(now.valueOf() - this.beg.valueOf())
      }, 1000)
    }
  },
  methods: {
    saveState() {
      const data = {
        laps: this.laps.map(lap => lap.valueOf()),
        end: this.end && this.end.valueOf(),
        beg: this.beg && this.beg.valueOf(),
        running: this.running,
        finished: this.finished
      }

      const dataString = JSON.stringify(data)

      window.localStorage.setItem('stopwatch', dataString)
    },
    start() {
      this.running = true
      if (this.finished === false) {
        this.beg = new Date()
      } else {
        let now = new Date()
        this.beg = new Date(now.valueOf() - (this.end.valueOf() - this.beg.valueOf()))
        this.laps = this.laps.map(lap => new Date(now.valueOf() - (this.end.valueOf() - lap.valueOf())))
      }
      this.interval = setInterval(() => {
        let now = new Date()
        this.totalTime = Math.floor(now.valueOf() - this.beg.valueOf())
        }, 1000)
      this.finished = false
      
      this.saveState()
    },
    newLap() {
      this.laps.push(new Date())

      this.saveState()
    },
    stop() {
      this.running = false
      this.finished = true
      this.end = new Date()
      clearInterval(this.interval)

      this.saveState()
    },
    reset() {
      this.running = false
      this.finished = false
      this.beg = null
      this.end = null
      this.totalTime = 0
      this.laps = []
      clearInterval(this.interval)

      window.localStorage.removeItem('stopwatch')
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
    },
    download() {
      let text = `"Screenlight Premiere Pro Marker Export 1.0",,,,
"Name:","Time:","Duration:","Marker Color:","Comment:"
`

      for (let i = 0; i < this.laps.length; i++) {
        const ms = this.laps[i].valueOf() - this.beg.valueOf()

        const [hours, minutes, seconds] = this.calculateHMS(ms)

        let timeString = this.timeText(hours, minutes, seconds)

        text = text + `"whatever",${ms / 1000},0,#718637,"${timeString}"\n`
      }
      
      let blob = new Blob([text], {type: "text/plain;charset=utf-8"})
      saveAs(blob, "timestamps.csv")
    },
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
