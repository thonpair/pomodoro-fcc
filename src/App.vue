<template>
  <div id="app">
    <div id="break">
    <label id="break-label">Break length</label>
    <button
      id="break-decrement"
      v-on:click="breakLen > 1 ? breakLen-- : breakLen"
    >
      -
    </button>
    <button
      id="break-increment"
      v-on:click="breakLen < 59 ? breakLen++ : breakLen"
    >
      +
    </button>
    <p id="break-length">{{ breakLen }}</p>
    </div>
    <div id="session">
    <label id="session-label">Session length</label>
    <button id="session-decrement" v-on:click="minusSessionLen">-</button>
    <button id="session-increment" v-on:click="plusSessionLen">+</button>
    <p id="session-length">{{ sessionLen }}</p>
    <label v-if="sessionRunning" id="timer-label">Session</label>
    <label v-else id="timer-label">Break</label>

    </div>
    <div id="countdown">
    <p id="time-left" :key="timeLeft">{{ timeLeft }}</p>

    </div>
    <div id="buttons">
    <button id="start_stop" v-on:click="startStop">start / stop</button>
    <button id="reset" v-on:click="reset">reset</button>

    </div>
    <audio
      id="beep"
      preload="auto"
      src="https://raw.githubusercontent.com/freeCodeCamp/cdn/master/build/testable-projects-fcc/audio/BeepSound.wav"
    />
  </div>
</template>

<script>
const DEF_BREAK = 5;
const DEF_SESSION = 25;
export default {
  name: "App",
  data() {
    return {
      breakLen: DEF_BREAK,
      sessionLen: DEF_SESSION,
      running: false,
      timeLeftSec: 600,
      sessionRunning: true,
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
  },
  mounted() {
    this.timeLeft = this.sessionLen;
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
