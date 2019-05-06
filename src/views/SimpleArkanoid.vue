<template>
  <div class="game-container">
    <div class="play-area"></div>
    <div class="balls">
      <div
        class="ball"
        v-for="(item, index) in fBalls"
        :key="index"
        :style="`margin-left: ${item.x}px; margin-top: ${item.y}px`"
      ></div>
    </div>
    <label for="player-input" class="input-label"></label>
    <div class="player" ref="player"></div>
    <input
      id="player-input"
      type="range"
      min="0"
      max="1000"
      @keyup="barMove($event.key)"
    />
    <div class="result-container">
      <div class="level" v-text="`Level : ${level}`"></div>
      <div class="point" v-text="`Score : ${point}`"></div>
      <div
        class="time"
        v-text="`Time : ${parseInt(time / 60)} : ${time % 60}`"
      ></div>
    </div>
  </div>
</template>

<script>
import Swal from "sweetalert2";

export default {
  data() {
    return {
      width: window.innerWidth,
      height: window.innerHeight,
      point: 0,
      level: 1,
      time: 0,
      ballTimer: null,
      gameTimer: null,
      pPosition: 0,
      barWith: 300,
      initPRange: {
        left: 0,
        right: 0,
        top: window.innerHeight - 150
      },
      pRange: {
        left: 0,
        right: 0,
        top: window.innerHeight - 150
      },
      initBalls: [
        {
          x: window.innerWidth / 2 - 25,
          y: 0,
          vecX: -2,
          vecY: 4,
          alive: true
        }
      ],
      balls: [
        {
          x: window.innerWidth / 2 - 25,
          y: 0,
          vecX: -2,
          vecY: 4,
          alive: true
        }
      ]
    };
  },
  watch: {
    pPosition: function() {
      this.pRange.left = ((this.width - this.barWidth) / 1000) * this.pPosition;
      this.pRange.right = this.pRange.left + this.barWidth;
      this.$refs["player"].style.marginLeft = `${this.pRange.left}px`;
    },
    fBalls: function() {
      if (this.fBalls.length < 1) {
        clearInterval(this.gameTimer);
        clearInterval(this.ballTimer);
        Swal.fire("Your score!", `${this.point}`).then(() => {
          this.balls = this.initBalls;
          this.pRange = this.initPRange;
          this.point = 0;
          this.level = 1;
          this.time = 0;
          Swal.fire("Press button to start").then(() => {
            this.gameTimer = setInterval(this.timeCount, 1000);
            this.ballTimer = setInterval(this.ballMove, 10);
          });
        });
      }
    }
  },
  computed: {
    fBalls: function() {
      return this.balls.filter(ba => ba.alive);
    }
  },
  created() {
    window.addEventListener("resize", this.handleResize);
  },
  mounted() {
    this.barWidth =
      parseInt(this.width / 3) < 400 ? parseInt(this.width / 3) : 400;
    this.$refs["player"].style.width = `${this.barWidth}px`;
    Swal.fire("Press button to start").then(() => {
      this.gameTimer = setInterval(this.timeCount, 1000);
      this.ballTimer = setInterval(this.ballMove, 10);
    });
  },
  methods: {
    handleResize: function() {
      this.width = window.innerWidth;
      this.height = window.innerHeight;
      this.barWidth =
        parseInt(this.width / 3) < 400 ? parseInt(this.width / 3) : 400;
      this.$refs["player"].style.width = `${this.barWidth}px`;
      this.pRange.top = this.height - 150;
    },
    barMove: function(key) {
      if (key === "ArrowLeft") {
        this.pPosition = this.pPosition - 100 > -1 ? this.pPosition - 100 : 0;
      } else if (key === "ArrowRight") {
        this.pPosition =
          this.pPosition + 100 < 1001 ? this.pPosition + 100 : 1000;
      }
    },
    ballMove: function() {
      this.balls.forEach(ball => {
        const asX = ball.x + ball.vecX;
        const asY = ball.y + ball.vecY;
        if (0 > asX || asX + 50 > this.width) ball.vecX *= -1;
        if (asY < 0) ball.vecY *= -1;
        if (
          ball.x <= this.pRange.right &&
          ball.x + 50 >= this.pRange.left &&
          asY > this.pRange.top
        ) {
          ball.vecY *= -1;
          let diff = asX - this.pRange.left;
          if (diff >= 250) {
            ball.vecX *= 1 + (this.barWidth - diff) / 50;
          }
          if (diff < 50) {
            ball.vecX *= 1 + diff / 50;
          }
        }
        ball.vecX = ball.vecX > 5 * this.level ? 5 * this.level : ball.vecX;
        ball.x += ball.vecX;
        ball.y += ball.vecY;
        if (ball.y > this.pRange.top) ball.alive = false;
      });
    },
    timeCount: function() {
      this.time += 1;
      if (parseInt(this.time / 30) + 1 > this.level) {
        this.balls.forEach(ball => {
          ball.vecX *= 1.2;
          ball.vecY *= 1.2;
        });
      }
      this.level = parseInt(this.time / 30) + 1;
      this.point += this.level * 10;
    }
  }
};
</script>

<style lang="scss" scoped>
@mixin ball($color) {
  position: absolute;
  height: 50px;
  width: 50px;
  background-color: $color;
}

.game-container {
  .play-area {
    position: absolute;
    height: calc(100vh - 50px);
    width: 100vw;
    background-color: whitesmoke;
  }

  .player {
    position: absolute;
    height: 50px;
    width: 300px;
    background-color: rgb(188, 199, 36);
    left: 0;
    bottom: 50px;
    transition: all 0.2s ease-in-out;
  }
  .ball {
    @include ball(grey);
  }

  input {
    position: absolute;
    height: 0px;
    width: 0px;
    bottom: 0;
    left: 0;
    margin-left: -100px;
  }

  .input-label {
    position: absolute;
    height: 100vh;
    width: 100vw;
    top: 0;
    left: 0;
    z-index: 100px;
  }

  .result-container {
    position: absolute;
    display: flex;
    justify-content: space-evenly;
    width: 100vw;
    height: 50px;
    bottom: 0;
    font-size: 15px;
    font-weight: bold;

    .level {
      width: 30vw;
      height: 50px;
      line-height: 50px;
      text-align: center;
    }
    .point {
      width: 30vw;
      height: 50px;
      line-height: 50px;
      text-align: center;
    }
    .time {
      width: 30vw;
      height: 50px;
      line-height: 50px;
      text-align: center;
    }
  }
}
</style>
