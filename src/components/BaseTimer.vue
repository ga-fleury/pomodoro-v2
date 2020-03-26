<template>
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
          :stroke-dasharray="circleDasharray"
          class="base-timer__path-remaining"
          :class="remainingPathColor"
          d="
            M 50, 50
            m -45, 0
            a 45,45 0 1,0 90,0
            a 45,45 0 1,0 -90,0
          "
        ></path>
      </g>
    </svg>
    <span class="base-timer__label">{{ formattedTimeLeft }}</span>
    <span class="base-timer__cycle">{{
      currentCycle[this.pomodoroTimes]
    }}</span>
  </div>
</template>

<script>
const FULL_DASH_ARRAY = 283;
const WARNING_THRESHOLD = 10;
const ALERT_THRESHOLD = 5;

const COLOR_CODES = {
  info: {
    color: "green"
  },
  warning: {
    color: "orange",
    threshold: WARNING_THRESHOLD
  },
  alert: {
    color: "red",
    threshold: ALERT_THRESHOLD
  }
};

//initial clock time
var TIME_LIMIT = 1500;

var endedYet;

export default {
  props: {
    currentRunState: Boolean,
    clockSwitch: Boolean,
    pomodoroTimes: Number
  },
  data() {
    return {
      timePassed: 0,
      timerInterval: null,
      currentCycle: [
        "1st Pomodoro",
        "Short Break",
        "2nd Pomodoro",
        "Short Break",
        "3rd Pomodoro",
        "Short Break",
        "4th Pomodoro",
        "Long Break"
      ]
    };
  },

  computed: {
    circleDasharray() {
      return `${(this.timeFraction * FULL_DASH_ARRAY).toFixed(0)} 283`;
    },

    formattedTimeLeft() {
      const minutes = Math.floor(this.timeLeft / 60);
      let seconds = this.timeLeft % 60;

      if (seconds < 10) {
        seconds = `0${seconds}`;
      }

      return `${minutes}:${seconds}`;
    },

    timeLeft() {
      return TIME_LIMIT - this.timePassed;
    },

    timeFraction() {
      const rawTimeFraction = this.timeLeft / TIME_LIMIT;
      return rawTimeFraction - (1 / TIME_LIMIT) * (1 - rawTimeFraction);
    },

    remainingPathColor() {
      const { alert, warning, info } = COLOR_CODES;

      if (this.timeLeft <= alert.threshold) {
        return alert.color;
      } else if (this.timeLeft <= warning.threshold) {
        return warning.color;
      } else {
        return info.color;
      }
    }
  },

  watch: {
    timeLeft(newValue) {
      if (newValue === 0) {
        endedYet = true;
        this.stopTimer();
        this.pomodoroReset();
        endedYet = false;
      }
    },
    currentRunState(newValue) {
      //watches for runstate change and triggers start/stop functions
      if (newValue == false) {
        this.stopTimer();
      } else if (newValue == true && endedYet != true) {
        this.startTimer();
      }
    },
    clockSwitch() {
      this.timePassed = 0; //resets timer
      clearInterval(this.timerInterval); //stops clock
      this.$emit("labelReset", false); //don't touch this. works in misterious ways (updates run state to control)
    }
  },

  methods: {
    pomodoroReset() {
      this.pomodoroCounter = this.pomodoroTimes;
      this.pomodoroCounter++;
      this.$emit("pomodoroFinished");
      this.pomodoroCounter % 2
        ? (this.timePassed = 1200)
        : (this.timePassed = 0);
      if (this.pomodoroCounter % 7 == 0) {
        this.timePassed = 300;
      } else if (this.pomodoroCounter % 8 == 0) {
        this.$emit("pomodoroReset", 0);
      }
      this.$emit("labelReset", false);
    },
    startTimer() {
      this.timerInterval = setInterval(() => (this.timePassed += 1), 1000);
    },
    stopTimer() {
      clearInterval(this.timerInterval);
    }
  }
};
</script>

<style scoped lang="scss">
.base-timer {
  position: relative;
  width: 300px;
  height: 300px;
  margin: 0 auto;

  &__svg {
    transform: scaleX(-1);
  }

  &__circle {
    fill: none;
    stroke: none;
  }

  &__path-elapsed {
    stroke-width: 7px;
    stroke: grey;
  }

  &__path-remaining {
    stroke-width: 7px;
    stroke-linecap: round;
    transform: rotate(90deg);
    transform-origin: center;
    transition: 1s linear all;
    fill-rule: nonzero;
    stroke: currentColor;

    &.green {
      color: rgb(65, 184, 131);
    }

    &.orange {
      color: orange;
    }

    &.red {
      color: red;
    }
  }

  &__label {
    position: absolute;
    width: 300px;
    height: 300px;
    top: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 48px;
  }
  &__cycle {
    position: absolute;
    width: 300px;
    height: 220px;
    top: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 16px;
  }
}
</style>
