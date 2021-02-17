<template>
  <div class="hello">
    <h1>Timer app</h1>
    <div class="inputs">
      <p>Nowy licznik</p>
      <input type="datetime-local" v-model="date_input_value" />
      <br />
      <br />
      <button v-on:click="add_new_timer">+</button>
    </div>
    <div id="list">
      <ul>
        <li v-for="timer in dead_timers" :key="timer.value">
          {{ timer.time.seconds }} <button>restart</button>

          <button v-on:click="delete_timer(timer)">x</button>
        </li>
      </ul>
      <br />
      <ul>
        <li v-for="timer in running_timers" :key="timer.value">
          {{ timer.remaining.days }}::{{ timer.remaining.hours }}::{{
            timer.remaining.minutes
          }}::{{ timer.remaining.seconds }} :::: {{ timer.remaining.done }}

          <button v-on:click="delete_timer(timer)">x</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      date_now: "",
      date_input_value: "",
      timers_values: []
    };
  },
  mounted() {
    this.$options.timer = window.setTimeout(this.update_time, 1000);
    this.date_now = Date.now();
  },
  methods: {
    diff(timer_value) {
      var diff = Date.parse(timer_value) - this.date_now;
      let done = diff < 0;
      let d = Math.floor(diff / 86400000);
      diff = diff - d * 86400000;
      let h = Math.floor(diff / 3600000);
      diff = diff - h * 3600000;
      let m = Math.floor(diff / 60000);
      diff = diff - m * 60000;
      let s = Math.floor(diff / 1000);
      return { days: d, hours: h, minutes: m, seconds: s, done: done };
    },
    add_new_timer() {
      if (
        !this.timers_values.some(
          timer => timer.value == this.date_input_value
        ) &&
        Date.parse(this.date_input_value) - this.date_now > 0
      ) {
        this.timers_values.push({
          value: this.date_input_value,
          time: this.diff(this.date_input_value)
        });
      }
    },
    delete_timer(timer) {
      let index = this.timers_values.findIndex(x => x.value === timer.value);
      console.log(index);
      if (index > -1) {
        this.timers_values.splice(index, 1);
      }
    },
    update_time() {
      this.date_now = Date.now();
      this.$options.timer = window.setTimeout(this.update_time, 1000);
    }
  },
  computed: {
    now() {
      return Date.now();
    },
    timers() {
      return this.timers_values.map(x => {
        return { ...x, remaining: this.diff(x.value) };
      });
    },
    running_timers() {
      return this.timers.filter(x => {
        return x.remaining.done === false;
      });
    },
    dead_timers() {
      return this.timers.filter(x => {
        return x.remaining.done === true;
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
