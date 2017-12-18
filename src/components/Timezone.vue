<template>
  <section class="timezone" :class="[textColor]" :style="{backgroundColor: timeColor.asString}">
    <div v-if="timezone">
      <div v-if="formattedDates">
        <h3 class="label">{{timezone.name}} <span>({{formattedDates.abbr}})</span></h3>
        <p class="time-human">{{formattedDates.humanReadable}}</p>
        <p class="time-iso">{{formattedDates.iso}}</p> 
      </div>
      <div v-else>
        <h3>{{timezone.name}}</h3>
      </div>
      <button class="btn-trash" v-on:click="deleteTimezone()">x</button>
    </div>
    <div v-else>
      <p>Select a timezone</p>
      <select
        ref="select"
        v-model="selectedTimezone" 
        v-on:change="updateTimezone()">
        <option v-for="tz in timezonesList" :value="tz">
        {{ tz }}
        </option>
      </select>
      <button class="btn-trash" v-on:click="deleteTimezone()">x</button>
    </div>
  </section>
</template>

<script>
import moment from 'moment-timezone';

export default {
  name: 'Timezone',
  props: ['momentDatetime', 'index', 'timezone', 'totalTimezones'],
  data() {
    return {
      footimeColor: 'rgb(255,159,116)',
      selectedTimezone: null,
      timezonesList: moment.tz.names(),
      colorGroups: [
        { startHour: 0, color: { r: 50, g: 60, b: 100 } },
        { startHour: 4, color: { r: 139, g: 152, b: 206 } },
        { startHour: 8, color: { r: 86, g: 216, b: 255 } },
        { startHour: 12, color: { r: 255, g: 216, b: 116 } },
        { startHour: 15, color: { r: 255, g: 183, b: 116 } },
        { startHour: 18, color: { r: 255, g: 135, b: 116 } },
        { startHour: 21, color: { r: 40, g: 75, b: 215 } },
      ],
    };
  },
  mounted() {
    this.$refs.select.focus();
  },
  computed: {
    totalColorGroups() {
      return this.colorGroups.length;
    },
    timezoneDatetime() {
      if (!this.timezone) {
        return null;
      }

      const timezoneDate = moment.tz(this.momentDatetime, this.timezone.name);
      return (timezoneDate.isValid()) ? timezoneDate : null;
    },
    formattedDates() {
      if (!this.timezoneDatetime) {
        return null;
      }

      return {
        humanReadable: this.timezoneDatetime.clone().format('llll'),
        iso: this.timezoneDatetime.clone().format(),
        abbr: this.timezoneDatetime.clone().format('z'),
      };
    },
    timeColor() {
      if (!this.timezoneDatetime) {
        return {};
      }

      return this.getTimeColor(this.timezoneDatetime);
    },
    textColor() {
      if (!this.timeColor || !this.timeColor.asString) {
        return '';
      }

      return this.getTextColor(this.timeColor);
    },
  },
  methods: {
    getRange(time) {
      const groups = [];

      // Current hour + minutes in military time
      const hour = parseInt(time.format('H'), 10);
      const minute = parseInt(time.format('m'), 10);

      for (let i = 0; i < this.totalColorGroups; i += 1) {
        const nextIdx = (i + 1 < this.totalColorGroups) ? i + 1 : 0;
        const currGroup = this.colorGroups[i];
        const nextGroup = this.colorGroups[nextIdx];

        // Check if we have found the correct color range:
        //    current hour >= the currGroups's start time
        //  AND current hour is < nextGroup's start time
        //  OR  nextGroup's start time is midnight
        //    meaning that the currGroup's index is the last in the arr
        if (hour >= currGroup.startHour
           && (hour < nextGroup.startHour || nextGroup.startHour === 0)
        ) {
          groups[0] = currGroup;
          groups[1] = nextGroup;
          break;
        }
      }

      return {
        time: { hour, minute },
        groups,
      };
    },
    getTimeColor(time) {
      // Get the start + end colors - as well as the time used
      const range = this.getRange(time);

      const interval = {};
      const distance = {};

      const endRangeTime = (range.groups[1].startHour === 0) ? 24 : range.groups[1].startHour;
      const numHrsInRange = Math.abs(endRangeTime - range.groups[0].startHour);
      const timeSinceRangeBegin = Math.abs(range.time.hour - range.groups[0].startHour);

      // Get the total # of hours b/w the two groups
      // Split the transition distance (1) to pieces for each hour mark
      interval.hour = +(1 / numHrsInRange).toFixed(3);

      // Split the hour interval into 60 pieces (1 for each minute)
      interval.minute = +(interval.hour / 60).toFixed(3);

      // Calculate the current hour + minute values using the intervals
      distance.hour = +(interval.hour * timeSinceRangeBegin).toFixed(3);
      distance.minute = +(interval.minute * range.time.minute).toFixed(3);
      distance.total = +(distance.hour + distance.minute).toFixed(3);

      const colorParts = ['r', 'g', 'b'];
      const newColor = [];

      for (let i = 0; i < colorParts.length; i += 1) {
        const part = colorParts[i];
        const startColor = range.groups[0].color[part];
        const endColor = range.groups[1].color[part];
        const transitionColor = Math.round(endColor + ((startColor - endColor) * distance.total));
        newColor.push(transitionColor);
      }

      return {
        asString: `rgb(${newColor.join(',')})`,
        asArray: newColor,
      };
    },
    // https://stackoverflow.com/a/1855903
    getTextColor(timeColor) {
      const color = timeColor.asArray;

      // Counting the perceptive luminance - human eye favors green color..
      const r = 0.299 * color[0];
      const g = 0.587 * color[1];
      const b = 0.114 * color[2];
      const a = 1 - ((r + g + b) / 255);

      return (a < 0.4) ? 'black' : 'white';
    },
    deleteTimezone() {
      this.$emit('delete-timezone', { index: this.index });
    },
    updateTimezone() {
      this.$emit('update-timezone', {
        index: this.index,
        zoneName: this.selectedTimezone,
      });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  $max_timezones: 10;

  .timezone {
    display: flex;
    align-items: center;
    justify-content: center;

    position: relative;
    margin: 0;
    padding: 5% 32px;
    width: 100%;

    text-align: center;
  
    @for $i from 1 through $max_timezones {
      #app[data-total-timezones='#{$i}'] & {
        min-height: calc((100% - 160px) / #{$i});
      } 
    }

    &.black,
    &.black .label,
    &.black .btn-trash {
      color: black;
    }

    &.white,
    &.white .label,
    &.white .btn-trash {
      color: white;
    }

    .label {
      padding: 0;
      margin: 0 0 16px 0;
      align-self: flex-start;
    }

    .time-human {
      margin: 0;
      padding: 8px 0;
      font-size: 18px;
    }

    .time-iso {
      margin: 0;
      padding: 8px 0;
      font-size: 14px;
      font-weight: 100;
    }

    .btn-trash {
      position: absolute;
      top: 0;
      right: 0;
      border: 0;
      padding: 3% 3%;

      background-color: transparent;
      text-transform: uppercase;
      font-weight: 100;
      font-size: 16px;
      
      cursor: pointer;
      transition: opacity 200ms linear;

      &:hover {
        opacity: 0.5;
      }
    }
  }

</style>
