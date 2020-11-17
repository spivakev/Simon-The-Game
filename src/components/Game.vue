<template>
  <div id="game">
    <h1 class="game__title">Simon Says</h1>
    <div class="container">
      <div class="game__circle circle">
        <div class="row">
          <button
            class="btn btn--blue"
            v-bind:class="{'btn--active': buttons[0].isActive}"
            name="blue"
            id="1"
            v-on:click="colorClicked(1,'blue', $event)"
          />
          <button
            class="btn btn--red"
            v-bind:class="{'btn--active': buttons[1].isActive}"
            name="red"
            id="2"
            v-on:click="colorClicked(2,'red', $event)"
          />
        </div>
        <div class="row">
          <button
            class="btn btn--yellow"
            v-bind:class="{'btn--active': buttons[2].isActive}"
            name="yellow"
            id="3"
            v-on:click="colorClicked(3, 'yellow', $event)"
          />
          <button
            class="btn btn--green"
            v-bind:class="{'btn--active': buttons[3].isActive}"
            name="green"
            id="4"
            v-on:click="colorClicked(4, 'green', $event)"
          />
        </div>
      </div>

      <div class="game__round round">
        <h3 v-if="!gameFailed" class="round__subtitle">Round: {{roundNumber}}</h3>
        <p class="round__message" v-if="gameFailed">Game lost on Round {{roundNumber}}</p>
        <button class="round__btn" v-on:click="restartGame">Start</button>
        <h3 class="round__subtitle">Levels:</h3>
        <ul class="round__levels levels">
          <li>
            <label>
              <input type="radio" name="level" id="level-easy" value="easy" v-model="level" />
              Easy
            </label>
          </li>
          <li>
            <label>
              <input type="radio" name="level" id="level-medium" value="medium" v-model="level" />
              Medium
            </label>
          </li>
          <li>
            <label>
              <input type="radio" name="level" id="level-hard" value="hard" v-model="level" />
              Hard
            </label>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

 

<script>
export default {
  name: "Game",
  data() {
    return {
      level: "medium",
      delayPeriod: 1000,
      roundNumber: 0,
      roundSequence: [],
      gameStarted: false,
      gameFailed: false,
      listenSequence: false,

      clickedButtons: [],
      checkingIndex: 0,

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
      },
      buttons: [
        {
          id: 1,
          name: "blue",
          isActive: false
        },
        {
          id: 2,
          name: "red",
          isActive: false
        },
        {
          id: 3,
          name: "yellow",
          isActive: false
        },
        {
          id: 4,
          name: "green",
          isActive: false
        }
      ],
      sounds: {
        blue: "1",
        red: "2",
        yellow: "3",
        green: "4"
      }
    };
  },

  watch: {
    level: function() {
      this.roundNumber = 0;
      this.gameFailed = false;
      this.delayPeriod = this.levels[this.level].delayPeriod;
    },

    roundNumber: function() {
      if (this.roundNumber) {
        this.generateSequence();
      }
    },

    clickedButtons: function() {
      if (this.clickedButtons.length > 0) {
        if (
          this.clickedButtons.length > 0 &&
          this.clickedButtons[this.checkingIndex] ==
            this.roundSequence[this.checkingIndex]
        ) {
          this.checkingIndex++;

          if (this.checkingIndex === this.roundSequence.length) {
            this.clearRoundInfo();
            this.nextRound();
          }
        } else {
          this.clearRoundInfo();
          this.finishGame();
        }
      }
    }
  },

  methods: {
    restartGame() {
      this.roundNumber = 0;
      this.roundSequence = [];
      this.clickedButtons = [];
      this.gameFailed = false;

      this.nextRound();
      this.gameStarted = true;
    },

    generateSequence() {
      this.roundSequence = [];

      for (let i = 0; i < this.roundNumber; i++) {
        this.roundSequence.push(this.randomNumber(1, 4));
      }

      this.showSequence();
    },

    randomNumber(min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min);
    },

    showSequence() {
      let buttons = document.querySelectorAll(".btn");
      let event = new Event("click");

      //создаем массив кнопок в порядке их нажатия
      let arrayToClick = [];

      for (let i = 0; i < this.roundSequence.length; i++) {
        for (let j = 0; j < buttons.length; j++) {
          if (buttons[j].id == this.roundSequence[i]) {
            arrayToClick.push(buttons[j]);
            continue;
          }
        }
      }

      let clickAndWait = btn => {
        return () => {
          return new Promise(resolve => {
            btn.dispatchEvent(event);
            setTimeout(() => {
              resolve();
            }, this.delayPeriod);
          });
        };
      };

      let p = Promise.resolve();

      for (let btn of arrayToClick) {
        p = p.then(clickAndWait(btn));
      }
      p.then(() => {
        this.clickedButtons = [];
        this.listenSequence = true;
      });
    },

    playSound(sound) {
      if (sound) {
        let audio = new Audio(require(`@/sounds/${sound}.mp3`));
        audio.play();
      }
    },

    colorClicked(id, buttonName, event) {
      if (this.gameStarted) {
        //если пользователь пытается нажать на кнопку до того, как проиграна вся последовательность, воспроизведения звука и изменения стиля этой кнопки не произойдет
        if (!event.isTrusted || (event.isTrusted && this.listenSequence)) {
          this.buttons[id - 1].isActive = true;

          let sound = this.sounds[buttonName];

          this.playSound(sound);
          setTimeout(() => {
            this.buttons[id - 1].isActive = false;
          }, 150);

          if (event.isTrusted && event.type == "click") {
            this.clickedButtons.push(id);
          }
        }
      }
    },

    clearRoundInfo() {
      this.clickedButtons = [];
      this.roundSequence = [];
      this.checkingIndex = 0;
    },

    nextRound() {
      this.listenSequence = false;
      setTimeout(() => {
        this.roundNumber++;
      }, this.delayPeriod * 1.5);
    },

    finishGame() {
      this.gameStarted = false;
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
  width: 750px;
  padding-left: 300px;
  margin: 0 auto;
  display: flex;
  justify-content: center;
}

/* Круг с кнопками */

.game__circle {
  margin-right: 40px;
}

.circle {
  width: 300px;
  height: 300px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 50%;
  box-shadow: rgb(170, 170, 170) 2px 1px 12px 0px;
}

.row {
  display: flex;
  justify-content: center;
  align-items: center;
}

.btn {
  width: 150px;
  height: 150px;
  opacity: 0.6;
  border: 2px solid white;
  cursor: pointer;
}

.btn:hover {
  border-color: #494949;
}

.btn--active {
  opacity: 1;
}

.btn:focus {
  outline: 0;
}

.btn--blue {
  background-color: dodgerblue;
  border-radius: 100% 0 0 0;
  border-right: none;
  border-bottom: none;
}

.btn--red {
  background-color: #ff5643;
  border-radius: 0 100% 0 0;
  border-left: none;
  border-bottom: none;
}

.btn--yellow {
  background-color: #feef33;
  border-radius: 0 0 0 100%;
  border-top: none;
  border-right: none;
}

.btn--green {
  background-color: #bede15;
  border-radius: 0 0 100% 0;
  border-left: none;
  border-top: none;
}

/* Настройки игры */
.game__round {
  margin-top: 20px;
}

.round {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 400px;
}

.round__subtitle,
.round__message {
  margin-bottom: 0.5rem;
  font-weight: 700;
  font-size: 1.4rem;
}

.round__message {
  /* padding: 0;*/
  text-align: left;
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
