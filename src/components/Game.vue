<template>
  <div id="game">
    <h1 class="game__title">Simon Says</h1>
    <div class="container">
      <div class="game__circle">
        <Simon
          v-bind:delayPeriod="delayPeriod"
          v-bind:roundSequence="roundSequence"
          v-bind:startShowing="startShowing"
          v-bind:gameFailed="gameFailed"
          v-on:round-finished="nextRound"
          v-on:game-failed="finishGame"
        />
      </div>
      <div class="game__round">
        <Round
          v-bind:roundNumber="roundNumber"
          v-on:change-level="changeLevel"
          v-on:show-sequence="showSequence"
        />
      </div>
    </div>
  </div>
</template>

 

<script>
import Simon from "@/components/Simon";
import Round from "@/components/Round";
export default {
  name: "Game",
  components: {
    Simon,
    Round
  },
  data() {
    return {
      level: "easy",
      delayPeriod: 1500,
      roundNumber: 0,
      roundSequence: null,
      startShowing: false,
      gameFailed: false,

      levels: {
        easy: {
          label: "Легкий",
          delayPeriod: 1500
        },
        medium: {
          label: "Средний",
          delayPeriod: 1000
        },
        hard: {
          label: "Сложный",
          delayPeriod: 400
        }
      }
    };
  },

  methods: {
    startGame() {
      this.roundNumber = 1;
    },

    changeLevel(level) {
      this.level = level;
      this.delayPeriod = this.levels[level].delayPeriod;
      console.log("Новый уровень сложности : ", this.level);
      console.log("Новая задержка между нажатиями : ", this.delayPeriod);
    },

    showSequence(sequence) {
      this.startShowing = true;
      this.roundSequence = sequence.slice();
    },

    nextRound() {
      this.startShowing = false;
      this.roundNumber++;
    },

    finishGame() {
      this.gameFailed = true;
    }
  }
};
</script>

<style>
.game__title {
  margin-bottom: 30px;
}

.container {
  width: 450px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
}

.game__round {
  margin-top: 20px;
}
</style>
