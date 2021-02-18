<template>
  <div v-bind:class="[mode]">
    <img alt="Vue logo" src="./../assets/logo.png" />
    <label class="switch">
      <input type="checkbox" v-model="dark" />
      <span class="slider"></span>
      <span>Dark mode</span>
    </label>
    <h1 id="title">Timer app</h1>
    <div class="inputs">
      <label for="date_input"
        >New Timer
        <input id="date_input" type="datetime-local" v-model="date_input_value"
      /></label>
      <button v-on:click="add_new_timer" class="positiv">+</button>
      <br />
      <br />
    </div>
    <div class="list">
      <ul id="dead_list">
        <h4>DONE</h4>
        <li v-for="timer in dead_timers" :key="timer.value">
          <div class="timer">
            <p>Countdown finished</p>
            <button v-on:click="restart_timer(timer)" class="positiv">
              restart
            </button>

            <button v-on:click="delete_timer(timer)" class="negativ">x</button>
          </div>
        </li>
      </ul>
      <br />
      <ul id="runing_list">
        <h4>RUNNING</h4>
        <li v-for="timer in running_timers" :key="timer.value">
          <div class="timer">
            <div class="row">
              <div class="cell"><p>Days</p></div>
              <div class="cell"><p>Hours</p></div>
              <div class="cell"><p>Min</p></div>
              <div class="cell"><p>Sec</p></div>
            </div>
            <div class="row">
              <div class="cell">
                <p>{{ timer.remaining.days }}</p>
              </div>

              <div class="cell">
                <p>{{ timer.remaining.hours }}</p>
              </div>

              <div class="cell">
                <p>{{ timer.remaining.minutes }}</p>
              </div>

              <div class="cell">
                <p>{{ timer.remaining.seconds }}</p>
              </div>
            </div>
            <button v-on:click="delete_timer(timer)" class="negativ">x</button>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      dark: false,
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
    },
    /**
     * Returns class name depend on mode selected.
     */
    mode() {
      if (this.dark === true) {
        return "app-dark";
      } else {
        return "app";
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less">
#colors() {
  primary: #ccedee;
  secondary: #faefae;
  positiv: #9bd985;
  positiv_h: saturate(spin(#9bd985, 15), 30%);
  negativ: #fe714a;
  negativ_h: saturate(spin(#fe714a, 15), 20%);
}
#colors_dark() {
  txt: rgb(173, 147, 75);
  primary: #517b7c;
  secondary: #474431;
  positiv: #335e24;
  positiv_h: saturate(spin(#335e24, 15), 30%);
  negativ: #b44a2c;
  negativ_h: saturate(spin(#b44a2c, 15), 20%);
}

.app {
  background: #colors[primary];
  .cell {
    border-color: black;
  }
  .timer {
    background: #colors[secondary];
    &:hover {
      border-color: brown;
    }
  }
  .positiv {
    background: #colors[positiv];
    &:hover {
      background: #colors[positiv_h];
    }
  }
  .negativ {
    background: #colors[negativ];
    &:hover {
      background: #colors[negativ_h];
    }
  }
  margin: 0;
  padding: 0;
}
.app-dark {
  background: #colors_dark[primary];
  color: #colors_dark[txt];
  .cell {
    border-color: rgba(114, 105, 105, 0.993);
  }
  .timer {
    background: #colors_dark[secondary];
    &:hover {
      border-color: rgba(165, 42, 42, 0.87);
    }
  }
  .positiv {
    color: #colors_dark[txt];
    background: #colors_dark[positiv];
    &:hover {
      background: #colors_dark[positiv_h];
    }
  }
  .negativ {
    background: #colors_dark[negativ];
    &:hover {
      background: #colors_dark[negativ_h];
    }
  }
}

.list {
  margin: 40px 0 0;
  width: auto;
  list-style-type: none;
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 10px;

  #dead_list {
    grid-column: 1;
    grid-row: 1;
    button:nth-child(2) {
      margin: 0px 20% 0px 0px;
      border-radius: 15px 15px 0px 0px;
    }
    button {
      border-radius: 0px 0px 15px 15px;
    }
  }
  #runing_list {
    grid-column: 2;
    grid-row: 1;
  }
}

ul {
  padding: 3px;
}
li {
  padding: 0px;
  display: block;
  margin: 0 2px;
}
.timer {
  padding: 0px;
  margin: 0px 5px 10px 5px;
  border: 2px solid black;
  border-radius: 15px;

  .row {
    text-align: center;
    font-weight: bold;
    display: block;
  }
  .cell {
    border-left: 1px solid;

    display: inline-block;
    width: 20%;
    padding: 5px;

    &:nth-child(1) {
      border-left: none;
    }
  }
  p {
    padding: 1%;
    margin: 1%;
  }
  button {
    width: 100%;
    border-radius: 15px;
    font-weight: bold;
    border: 2px solid black;
    margin-top: 2px;
  }
}
.switch {
  position: relative;
  display: block;
  width: 30px;
  height: 14px;
  input {
    opacity: 0;
    width: 0;
    height: 0;
    &:checked + .slider {
      background-color: #colors_dark[secondary];
    }
    &:checked + .slider:before {
      -webkit-transform: translateX(8px);
      -ms-transform: translateX(8px);
      transform: translateX(8px);
    }
  }
}
.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #colors[secondary];
  border-radius: 34px;
  -webkit-transition: 0.4s;
  transition: 0.4s;
  &:before {
    position: absolute;
    content: "";
    height: 12px;
    width: 12px;
    left: 5px;
    bottom: 1px;
    background-color: rgb(189, 186, 186);
    border-radius: 50%;
    -webkit-transition: 0.4s;
    transition: 0.4s;
  }
}
</style>
