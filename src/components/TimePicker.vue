<template>
  <div class="timepicker">
    <label class="label">Enter a datetime ({{localTimezone}})</label>
    <input 
      class="dateinput"
      ref='input'
      type="datetime-local" 
      v-bind:value="value"
      v-on:input="updateValue($event.target.value)" />
  </div>
</template>

<script>
export default {
  name: 'TimePicker',
  props: ['value', 'localTimezone'],
  methods: {
    updateValue(value) {
      this.$emit('input', value);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  .timepicker {
    width: 100%;
    height: 80px;
    margin: 0;
    padding: 16px 32px;

    background: #FBFBFB;

    .dateinput {
      border: 0;
      padding: 0;
      margin-top: 4px;

      background-color: transparent;
      font-family: 'Roboto Mono', monospace;
      font-size: 16px;
    }
  }
</style>
