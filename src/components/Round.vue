<template>
  <div class="round">
    <h3 v-if="!gameFailed" class="round__subtitle">Round: {{roundNumber}}</h3>
    <p v-if="gameFailed">Вы проиграли на {{roundNumber}} раунде</p>
    <button class="round__btn" v-on:click="startGame">Start</button>
    <h3 class="round__subtitle">Levels:</h3>
    <ul class="round__levels levels">
      <li>
        <label>
          <input
            type="radio"
            name="level"
            id="level-easy"
            v-on:change="$emit('change-level', level)"
            value="easy"
            v-model="level"
          />
          Easy
        </label>
      </li>
      <li>
        <label>
          <input
            type="radio"
            name="level"
            id="level-medium"
            v-on:change="$emit('change-level', level)"
            value="medium"
            v-model="level"
          />
          Medium
        </label>
      </li>
      <li>
        <label>
          <input
            type="radio"
            name="level"
            id="level-hard"
            v-on:change="$emit('change-level', level)"
            value="hard"
            v-model="level"
          />
          Hard
        </label>
      </li>
    </ul>
  </div>
</template>


<script>
export default {
  name: "Round",
  props: {
    roundNumber: Number,
    gameFailed: Boolean
  },
  data() {
    return {
      level: "easy"
    };
  },

  watch: {
    roundNumber: function() {
      //this.startGame();
    }
  },

  methods: {
    randomNumber(min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min);
    },

    startGame() {
      //генерируем последовательность кнопок
      let sequence = [];

      for (let i = 0; i < this.roundNumber; i++) {
        sequence.push(this.randomNumber(1, 4));
      }

      //передаем последовательность в родительский компонент Game
      this.$emit("show-sequence", sequence);
    }
  }
};
</script>


<style scoped>
.round {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.round__subtitle {
  margin-bottom: 0.5rem;
  font-weight: 700;
  font-size: 1.4rem;
}

.round__btn {
  padding: 0.3rem 0.6rem;
  border: none;
  border-radius: 10px;
  width: 5em;

  background-color: #6dabe8;
  font-weight: 500;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  font-size: 1.4rem;
}

.round__btn:hover {
  filter: brightness(110%);
}

.round__btn:active {
  filter: brightness(95%);
}

.round__btn:focus {
  outline: none;
}

.round__levels {
  margin: 0;
  padding: 0;
}

.levels {
  list-style: none;
  text-align: left;
  font-weight: 500;
}
</style>