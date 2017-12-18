<template>
  <div class="add-timezone">
    <button v-on:click="addTimezone()">
      Add Timezone
    </button>
  </div>
</template>

<script>
export default {
  name: 'AddTimezone',
  methods: {
    addTimezone() {
      this.$emit('add-timezone');
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  .add-timezone {
    display: block;
    position: relative;
    width: 100%;
    height: 80px;
    margin: 0;
    padding: 0;

    background: #FBFBFB;
    text-align: center;
  
    #app[data-total-timezones='0'] & {
      position: absolute;
      bottom: 0;
      left: 0;
    } 

    button {
      margin-top: 24px;
      padding: 8px 32px;

      border-radius: 32px;
      border: 0;
      background-color: #FF56A9;
      color: white;

      font-family: 'Roboto Mono', monospace;
      font-size: 16px;
      
      transition: background-color 200ms linear;

      &:hover {
        background-color: #FF8BC4;
        cursor: pointer;
      }
    }
  }
</style>
