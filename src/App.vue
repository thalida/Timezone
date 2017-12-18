<template>
  <div id="app" :data-total-timezones="totalTimezones">
    <TimePicker 
      v-model="datetime"
      v-bind:local-timezone="localTimezone" />
    <Timezone 
      v-for="(timezone, index) in timezones"
      :key="index"
      v-bind:moment-datetime="momentDatetime"
      v-bind:index="index"
      v-bind:timezone="timezone"
      v-on:delete-timezone="deleteTimezone"
      v-on:update-timezone="updateTimezone" />
    <AddTimezone 
      v-on:add-timezone="addTimezone" />
  </div>
</template>

<script>
import moment from 'moment-timezone';
import TimePicker from './components/TimePicker';
import Timezone from './components/Timezone';
import AddTimezone from './components/AddTimezone';

export default {
  name: 'app',
  components: {
    TimePicker,
    Timezone,
    AddTimezone,
  },
  data() {
    return {
      maxTimezones: 10,
      localTimezone: moment.tz.guess(),
      datetime: moment().format(moment.HTML5_FMT.DATETIME_LOCAL),
      timezones: [
        moment.tz.zone(moment.tz.guess()),
      ],
    };
  },
  computed: {
    momentDatetime() {
      return moment(this.datetime);
    },
    totalTimezones() {
      return this.timezones.length;
    },
  },
  methods: {
    addTimezone() {
      if (this.timezones.length >= this.maxTimezones) {
        return;
      }

      this.timezones.push(null);
    },
    deleteTimezone(res) {
      this.timezones.splice(res.index, 1);
    },
    updateTimezone(res) {
      this.timezones.splice(res.index, 1, moment.tz.zone(res.zoneName));
    },
  },
};
</script>

<style lang="scss">
html {
  box-sizing: border-box;
}
*, *:before, *:after {
  box-sizing: inherit;
}

body {
  margin: 0;
  padding: 0;

  background: white;
}

#app {
  display: block;
  position: relative;
  width: 100%;
  height: 100vh;

  background: rgba(0,0,0,0.01);
  font-family: 'Roboto Mono', monospace;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.label {
  display: block;
  font-size: 12px;
  color: rgba(0,0,0,0.60);
  text-transform: uppercase;
}
</style>
