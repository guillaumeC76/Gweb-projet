<template>
  <div class="home">
    <h2>Votre grand record est de : {{ record > 0 ? record : 0 }}</h2>
    <div class="theFlex">
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
        <button @click="newGame()">Recommencer</button>
      </div>
      <div ref="main" class="main">
        <div ref="box" class="box" @click="random()"></div>
      </div>

      <div class="resultat">
        <h3 v-for="resultat in resultats" :key="resultat" class="result">
          {{ resultat }}
        </h3>
      </div>
    </div>
  </div>
</template>

<script>
const FULL_DASH_ARRAY = 283
const WARNING_THRESHOLD = 10
const ALERT_THRESHOLD = 5

const COLOR_CODES = {
  info: {
    color: 'green',
  },
  warning: {
    color: 'orange',
    threshold: WARNING_THRESHOLD,
  },
  alert: {
    color: 'red',
    threshold: ALERT_THRESHOLD,
  },
}

const TIME_LIMIT = 40

export default {
  name: 'Accueil',

  data() {
    return {
      clique: 0,
      timePassed: 0,
      timerInterval: null,
      activate: false,
      record: localStorage.getItem("recordDeClique"),
      resultats: [],
    }
  },
  computed: {

    circleDasharray() {
      return `${(this.timeFraction * FULL_DASH_ARRAY).toFixed(0)} 283`
    },

    formattedTimeLeft() {
      const timeLeft = this.timeLeft
      const minutes = Math.floor(timeLeft / 60)
      let seconds = timeLeft % 60

      if (seconds < 10) {
        seconds = `0${seconds}`
      }

      return `${minutes}:${seconds}`
    },

    timeLeft() {
      return TIME_LIMIT - this.timePassed
    },

    timeFraction() {
      const rawTimeFraction = this.timeLeft / TIME_LIMIT
      return rawTimeFraction - (1 / TIME_LIMIT) * (1 - rawTimeFraction)
    },

    remainingPathColor() {
      const { alert, warning, info } = COLOR_CODES

      if (this.timeLeft <= alert.threshold) {
        return alert.color
      } else if (this.timeLeft <= warning.threshold) {
        return warning.color
      } else {
        return info.color
      }
    },
  },

  watch: {
    timeLeft(newValue) {
      if (newValue === 0) {
        this.onTimesUp()
      }
    },
  },
  mounted() {
    if (localStorage.getItem('resultat'))
      this.resultats = JSON.parse(localStorage.getItem('resultat'))
  },
  methods: {
    random() {
      const { clientWidth: mainWidth, clientHeight: mainHeight } =
        this.$refs.main

      if (this.activate === false) {
        this.startTimer()
        this.activate = true

        this.$refs.box.style.left =
          Math.random() * (mainWidth - this.$refs.box.clientWidth) + 'px'
        this.$refs.box.style.top =
          Math.random() * (mainHeight - this.$refs.box.clientHeight) + 'px'

        this.clique++
      } else {
        this.$refs.box.style.left =
          Math.random() * (mainWidth - this.$refs.box.clientWidth) + 'px'
        this.$refs.box.style.top =
          Math.random() * (mainHeight - this.$refs.box.clientHeight) + 'px'

        this.clique++
      }
    },
    onTimesUp() {
      clearInterval(this.timerInterval)
      this.resultats.unshift(this.clique)
      localStorage.setItem('resultat', JSON.stringify(this.resultats))
      if (this.clique > this.record) {
        window.alert(
          'Votre nombre de cliques est de : ' +
            this.clique +
            '. NOUVEAUX RECORD !!!!!!'
        )
        this.record = this.clique
        localStorage.setItem('recordDeClique', this.record)
      } else if (this.record === this.clique) {
        window.alert(
          'Votre nombre de cliques est de : ' +
            this.clique +
            '. Vous êtes à égalité avec le record.'
        )
      } else {
        window.alert(
          'Votre nombre de cliques est de : ' +
            this.clique +
            '. Dommage, vous pouvez recommencer.'
        )
      }
    },

    startTimer() {
      this.timerInterval = setInterval(() => (this.timePassed += 1), 1000)
    },

    newGame() {
      document.location.reload()
    },
  },
}
</script>

<style lang="css">
body {
  margin: 0;
  background-color: rgb(65, 184, 131);
  color: white;
}

.theFlex {
  display: flex;
  align-items: center;
  justify-content: space-evenly;
  padding: 20px 0 40px 0;
}

@media screen and (max-width: 890px) {
  .theFlex {
    flex-direction: column;
    padding: 20px 0 40px 0;
  }
}

.home {
  color: white;
  margin: 0 50px;
}

.resultat {
  background-color: rgb(65, 184, 131);
  color: white;
  overflow-y: hidden;
  width: 10%;
  height: 524px;
  border: 2px solid white;
  border-bottom: none;
}

.result {
  margin: 0;
  width: 100%;
  padding: 12px 0;
  border-bottom: 2px solid white;
  transition: 1s;
}

.main {
  width: 500px;
  height: 500px;
  border: 2px solid white;
  border-radius: 20px;
  margin: 0 auto;
  position: relative;
  cursor: crosshair;
}

.box {
  width: 20px;
  height: 20px;
  border-radius: 20px;
  background-color: red;
  position: absolute;
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
  color: rgb(204, 103, 44);
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
  height: 300px;
  top: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 48px;
}
</style>
