# pedal-stopwatch

A simple stopwatch tool with lap support. Designed to be used with a USB pedal, but can be used also normally or with keyboard shortcuts.

If your pedal generates a key press other than 'x', you can change the key in the file `src/components/stopwatch.vue`:

    <button v-shortkey.once="['x']" @shortkey="newLap()" v-on:click="newLap()" :disabled="!running">Lap (X)</button>

Vue-ShortKey is used for the keyboard shortcuts. Check their [README](https://github.com/iFgR/vue-shortkey) for more options.


## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

