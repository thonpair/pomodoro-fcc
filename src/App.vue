<template>
  <div id="app">
    <h1>Pomodoro clock</h1>
    <div id="settings">
      <div id="break">
        <label id="break-label">Break length</label>
        <button
          id="break-decrement"
          v-on:click="breakLen > 1 ? breakLen-- : breakLen"
        >
          -
        </button>
        <input type="number" id="break-length" min="1" max="59" v-model="breakLen" />
        <button
          id="break-increment"
          v-on:click="breakLen < 59 ? breakLen++ : breakLen"
        >
          +
        </button>
      </div>
      <div id="session">
        <label id="session-label">Session length</label>
        <button id="session-decrement" v-on:click="minusSessionLen">-</button>
        <input type="number" id="session-length" min="1" max="59" v-model="sessionLen" />
        <button id="session-increment" v-on:click="plusSessionLen">+</button>
      </div>
      <div id="buttons">
        <button id="start_stop" v-on:click="startStop">start / stop</button>
        <button id="reset" v-on:click="reset">reset</button>
      </div>
    </div>
    <div id="clock">
      <div class="base-timer">
        <svg
          class="base-timer__svg"
          viewBox="0 0 100 100"
          xmlns="http://www.w3.org/2000/svg"
        >
          <g class="base-timer__circle">
            <circle
              class="base-timer__path-elapsed"
              cx="50"
              cy="50"
              r="45"
            ></circle>
            <path
              id="base-timer-path-remaining"
              class="base-timer__path-remaining"
              v-bind:class="remainingPathColor"
              :stroke-dasharray="circleDasharray"
              d="
          M 50, 50
          m -45, 0
          a 45,45 0 1,0 90,0
          a 45,45 0 1,0 -90,0
        "
            ></path>
          </g>
        </svg>
        <span v-if="sessionRunning" id="timer-label" class="base-timer__title"
          >Session</span
        >
        <span v-else id="timer-label" class="base-timer__title">Break</span>
        <span id="base-timer-label" class="base-timer__label">
          {{ timeLeft }}</span
        >
      </div>
    </div>
    <audio
      id="beep"
      preload="auto"
      src="https://raw.githubusercontent.com/freeCodeCamp/cdn/master/build/testable-projects-fcc/audio/BeepSound.wav"
    />
  </div>
</template>

<script>
const DEF_BREAK = 2;
const DEF_SESSION = 1;

const FULL_DASH_ARRAY = 283;
const WARNING_THRESHOLD = 30;
const ALERT_THRESHOLD = 10;

const COLOR_CODES = {
  info: {
    color: "green",
  },
  warning: {
    color: "orange",
    threshold: WARNING_THRESHOLD,
  },
  alert: {
    color: "red",
    threshold: ALERT_THRESHOLD,
  },
};

export default {
  name: "App",
  data() {
    return {
      breakLen: DEF_BREAK,
      sessionLen: DEF_SESSION,
      running: false,
      timeLeftSec: 600,
      sessionRunning: true,
      remainingPathColor: COLOR_CODES.info.color,
      circleDasharray: FULL_DASH_ARRAY,
    };
  },
  computed: {
    timeLeft: {
      get: function () {
        let minutes = Math.floor(this.timeLeftSec / 60, 0);
        minutes = minutes < 10 ? `0${minutes}` : minutes;
        let secondes = this.timeLeftSec % 60;
        secondes = secondes < 10 ? `0${secondes}` : secondes;
        return `${minutes}:${secondes}`;
      },
      set: function (newTime) {
        this.timeLeftSec = newTime * 60;
      },
    },
  },
  methods: {
    playSound() {
      let audioPlay = document.getElementById("beep");
      audioPlay.play();
    },
    updateTimeLeft() {
      this.timeLeftSec--;
      this.setCircleDasharray();
      this.setRemainingPathColor(this.timeLeftSec);
      if (this.timeLeftSec < 0) {
        this.playSound();
        if (this.sessionRunning) {
          this.timeLeft = this.breakLen;
        } else {
          this.timeLeft = this.sessionLen;
        }
        this.sessionRunning = !this.sessionRunning;
      }
      this.timeOut = window.setTimeout(this.updateTimeLeft, 1000);
    },
    plusSessionLen() {
      if (this.sessionLen < 59) {
        this.sessionLen++;
        this.timeLeft = this.sessionLen;
      }
    },
    minusSessionLen() {
      if (this.sessionLen > 1) {
        this.sessionLen--;
        this.timeLeft = this.sessionLen;
      }
    },
    startStop() {
      if (this.running) {
        clearTimeout(this.timeOut);
      } else {
        this.updateTimeLeft();
      }
      this.running = !this.running;
    },
    reset() {
      if (this.running) {
        clearTimeout(this.timeOut);
      }
      this.breakLen = DEF_BREAK;
      this.sessionLen = DEF_SESSION;
      this.timeLeft = this.sessionLen;
    },
    setRemainingPathColor(timeLeft) {
      const { alert, warning, info } = COLOR_CODES;
      if (timeLeft <= alert.threshold) {
        this.remainingPathColor = alert.color;
      } else if (timeLeft <= warning.threshold) {
        this.remainingPathColor = warning.color;
      } else {
        this.remainingPathColor = info.color;
      }
    },

    calculateTimeFraction() {
      const fullLen = this.sessionRunning
        ? this.sessionLen * 60
        : this.breakLen * 60;
      const rawTimeFraction = this.timeLeftSec / fullLen;
      return rawTimeFraction - (1 / fullLen) * (1 - rawTimeFraction);
    },

    setCircleDasharray() {
      this.circleDasharray = `${(
        this.calculateTimeFraction() * FULL_DASH_ARRAY
      ).toFixed(0)} 283`;
    },
  },
  mounted() {
    this.timeLeft = this.sessionLen;
  },
};
</script>

<style>
body {
  font-family: sans-serif;
  height: 100vh;
}
h1{
  text-align: center;
}

#clock {
  display: grid;
  place-items: center;
}

.base-timer {
  position: relative;
  width: 300px;
  height: 300px;
}

.base-timer__svg {
  transform: scaleX(-1);
}

.base-timer__circle {
  fill: none;
  stroke: none;
}

.base-timer__path-elapsed {
  stroke-width: 7px;
  stroke: grey;
}

.base-timer__path-remaining {
  stroke-width: 7px;
  stroke-linecap: round;
  transform: rotate(90deg);
  transform-origin: center;
  transition: 1s linear all;
  fill-rule: nonzero;
  stroke: currentColor;
}

.base-timer__path-remaining.green {
  color: rgb(65, 184, 131);
}

.base-timer__path-remaining.orange {
  color: orange;
}

.base-timer__path-remaining.red {
  color: red;
}

.base-timer__label {
  position: absolute;
  width: 300px;
  height: 350px;
  top: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 48px;
}

.base-timer__title {
  position: absolute;
  width: 300px;
  height: 200px;
  top: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 48px;
}
</style>
