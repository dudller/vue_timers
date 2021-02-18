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
          {{ timer.time.seconds }}
          <button v-on:click="restart_timer(timer)">restart</button>

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
    if (localStorage.getItem("timers_values")) {
      try {
        this.timers_values = JSON.parse(localStorage.getItem("timers_values"));
      } catch (e) {
        localStorage.removeItem("timers_values");
      }
    }
  },
  beforeDestroy() {
    this.save_data();
  },
  methods: {
    /**
     * Takes value of time and counts the difference betwen it and current time.
     * Returns a dictionary with days, hours, minutes, seconds, and done - bool value if time is greather than current date.
     * @param {int} timer_value - value of time in ms.
     */
    diff(timer_value) {
      var diff = timer_value - this.date_now;
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
    /**
     * Adds new timer with value of variable data_input_value
     */
    add_new_timer() {
      let value = Date.parse(this.date_input_value);
      let time = value - this.date_now;
      if (!this.timers_values.some(timer => timer.value == value) && time > 0) {
        this.timers_values.push({
          value: value,
          time: time
        });
        this.save_data();
      }
    },
    /**
     * Removes provided timer from a list of timers.
     * @param {object} timer - timer object to remove.
     */
    delete_timer(timer) {
      let index = this.timers_values.findIndex(x => x.value === timer.value);
      if (index > -1) {
        this.timers_values.splice(index, 1);
        this.save_data();
      }
    },
    /**
     * Resets timer time by setting it's value as sum of it's time property to current time.
     * @param {object} timer - timer object to reset.
     */
    restart_timer(timer) {
      let index = this.timers_values.findIndex(x => x.value === timer.value);
      if (index > -1) {
        this.timers_values[index].value = this.date_now + timer.time;
      }
    },
    /**
     * Every second updates value of current time.
     */
    update_time() {
      this.date_now = Date.now();
      this.$options.timer = window.setTimeout(this.update_time, 1000);
    },
    /**
     * Saves list of timers values to local storage.
     */
    save_data() {
      const parsed = JSON.stringify(this.timers_values);
      localStorage.setItem("timers_values", parsed);
    }
  },
  computed: {
    /**
     * Returns list of timers with additional remaining property provided by diff method.
     */
    timers() {
      return this.timers_values.map(x => {
        return { ...x, remaining: this.diff(x.value) };
      });
    },
    /**
     * Returns list of timers which are still running.
     */
    running_timers() {
      return this.timers.filter(x => {
        return x.remaining.done === false;
      });
    },
    /**
     * Returns list of timers which value is lower than current time.
     */
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
