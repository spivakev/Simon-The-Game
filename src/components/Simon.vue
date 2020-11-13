<template>
  <div class>
    <div class="simon">
      <div class="row">
        <button
          class="btn btn--blue"
          v-bind:class="{'btn--active': buttons[0].isActive}"
          name="blue"
          id="1"
          v-on:click="clickHandler(1,'blue')"
        />
        <button
          class="btn btn--red"
          v-bind:class="{'btn--active': buttons[1].isActive}"
          name="red"
          id="2"
          v-on:click="clickHandler(2,'red')"
        />
      </div>
      <div class="row">
        <button
          class="btn btn--yellow"
          v-bind:class="{'btn--active': buttons[2].isActive}"
          name="yellow"
          id="3"
          v-on:click="clickHandler(3, 'yellow')"
        />
        <button
          class="btn btn--green"
          v-bind:class="{'btn--active': buttons[3].isActive}"
          name="green"
          id="4"
          v-on:click="clickHandler(4, 'green')"
        />
      </div>
    </div>
    <br />
    <br />
    <button class="showSequence" v-on:click="showSequence">Show Sequence</button>
  </div>
</template>

<script>
export default {
  name: "Simon",
  props: {
    roundSequence: Array,
    startShowing: Boolean,
    delayPeriod: Number
  },
  data() {
    return {
      clickedButtons: [],
      listenClicks: false,
      gameFailed: false,
      checkingIndex: 0,
      roundFinished: false,
      sequence: [],
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
    roundSequence: function() {
      this.sequence = this.roundSequence.slice();
    },

    startShowing: function(val) {
      if (val) {
        this.showSequence();
        this.$emit("round-finished");
      }
    },
    clickedButtons: function(val) {
      if (this.listenClicks && !this.roundFinished) {
        console.log(" СЛУШАЕМ ");
        console.log("последовательность:", this.sequence);
        console.log("в watch: clickedButtons значение", val);

        if (this.checkingIndex === this.sequence.length - 1) {
          this.listenClicks = false;
          this.roundFinished = true;
          this.clickedButtons = [];
          this.sequence = [];
          this.checkingIndex = 0;
          console.log('Раунд закончен успешно! :)')

          /*Отправить в главный компонент сигнал о том, что раунд завершен*/
          this.$emit('round-finished');
        } 
        else if (
          this.clickedButtons.length > 0 &&
          this.clickedButtons[this.checkingIndex] ==
            this.sequence[this.checkingIndex]
        ) {
          // this.sequence.shift();
          // this.clickedButtons.shift();
          console.log(
            "Кнопка с индексом",
            this.checkingIndex,
            " угадана правильно"
          );
          this.checkingIndex++;
        } else { 
          console.log("Ты проиграла");
          this.gameFailed = true;
          this.listenClicks = false;
          this.clickedButtons = [];
          this.sequence = [];
          this.checkingIndex = 0;
          this.$emit('game-failed');
        }
      }
    }
  },

  methods: {
    clickHandler(id, buttonName) {
      //TODO: не делать задержку, если это пользователь тыкнул:!!!
      this.buttons[id - 1].isActive = true;

      let sound = this.sounds[buttonName];

      this.playSound(sound);
      setTimeout(() => {
        this.buttons[id - 1].isActive = false;
      }, 150); //TODO: сделать тут задержу такую же как между звуками (?)

      if (this.listenClicks) {
        this.clickedButtons.push(id);
      }

    },

    playSound(sound) {
      if (sound) {
        let audio = new Audio(require(`@/sounds/${sound}.mp3`));
        audio.play();
      }
    },

    showSequence() {
      console.log("последовательность:", this.sequence);
      let sequence = this.roundSequence.slice();

      let buttons = document.querySelectorAll(".btn");
      let event = new Event("click");

      //создаем массив кнопок в порядке их нажатия
      let arrayToClick = [];

      for (let i = 0; i < sequence.length; i++) {
        for (let j = 0; j < buttons.length; j++) {
          if (buttons[j].id == sequence[i]) {
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
        this.listenClicks = true;
      });

      /*  setTimeout(() => {
        this.listenClicks = true;
      }, this.delayPeriod * 3);*/
    }
    // ловим каждое нажатие кнопки
    //сразу сравниваем с последовательностью
    // если ошибка, то сразу прерываем и возвращаем false в компонент раунда
    //если все ок, возвращаем true в компонент раунда
  }
};
</script>

<style scoped>
.simon {
  width: 300px;
  height: 300px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 50%;
  /*overflow: hidden;*/
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

.btn:active,
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

.btn--yellow:hover {
}

.btn--green {
  background-color: #bede15;
  border-radius: 0 0 100% 0;
  border-left: none;
  border-top: none;
}

.btn--green:hover {
}

.showSequence {
  /* margin-top: 0.5rem;*/
  padding: 0.3rem 0.6rem;
  border: none;
  border-radius: 10px;
  min-width: 5em;
  /*height: 35px;*/

  background-color: #ff5643;
  font-weight: 500;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  font-size: 1.4rem;
}
</style>