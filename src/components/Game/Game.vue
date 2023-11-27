<template>
  <div class="simon-game">
    <h1>{{ title }}</h1>
    <div v-if="isGameOver" class="game-over">
      <h2>Game Over!</h2>
      <h2>Your score: {{ round }}</h2>
    </div>
    <div class="buttons">
      <button
          v-for="(btn, index) in buttons"
          :key="index"
          :class="[getButtonClass(btn), { active: btn === activeButton }]"
          @click="onHandleClickPlayer(btn)"
          :style="{
              '--animation-duration': animationDuration,
              'cursor': !isPlayingSequence && !isRoundOver ? 'pointer' : 'not-allowed',
              'pointer-events': !isPlayingSequence && canPlay ? 'auto' : 'none'}">
      </button>
    </div>
    <div class="player-ui">
      <ul v-if="!isGameBegin" class="game-options">
        <p>Сложность: </p>
        <li @click="setGameDifficult(0)" :class="{active: gameDifficult === 0}">легкая</li>
        <li @click="setGameDifficult(1)" :class="{active: gameDifficult === 1}">средняя</li>
        <li @click="setGameDifficult(2)" :class="{active: gameDifficult === 2}">трудная</li>
      </ul>

      <div class="game-stats">
        <h2 v-if="round !== 0 && !isGameOver">Round: {{ round }}</h2>
        <button v-if="round === 0 || isGameOver" @click="startGame">Start game</button>
        <button v-else-if="!isGameOver" @click="nextRound" :disabled="!isRoundOver || isPlayingSequence">Next round
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import './Game.sass';
import Red from "../../assets/sounds/red.mp3";
import Green from "../../assets/sounds/green.mp3";
import Blue from "../../assets/sounds/blue.mp3";
import Yellow from "../../assets/sounds/yellow.mp3";
import EndGame from "../../assets/sounds/end.mp3";

export default {
  name: 'SimonGame',
  data() {
    return {
      title: "Simon The Game",
      buttons: ["Red", "Green", "Blue", "Yellow"],
      sequence: [],
      playerSequence: [],
      activeButton: null,
      round: 0,
      isGameBegin: false,
      isGameOver: false,
      isRoundOver: true,
      canPlay: false,
      isPlayingSequence: false,
      gameDifficult: 0,
      animationDuration: '1.5s'
    };
  },
  methods: {
    async startGame() {
      this.isGameOver = false;
      this.isGameBegin = true;

      switch (this.gameDifficult) {
        case 0:
          this.animationDuration = '1.5s';
          break;
        case 1:
          this.animationDuration = '1s';
          break;
        case 2:
          this.animationDuration = '0.4s';
          break;
      }

      this.round = 0;
      await this.nextRound();
    },
    setGameDifficult(difficult) {
      this.gameDifficult = difficult
    },
    generateSequence() {
      for (let i = this.sequence.length; i < this.round; i++) {
        const randomIndex = Math.floor(Math.random() * this.buttons.length);
        this.sequence.push(this.buttons[randomIndex]);
      }
    },
    async playSequence() {
      this.isPlayingSequence = true;
      this.isRoundOver = false;

      for (const button of this.sequence) {
        await this.playButton(button).then(() => this.stopPlayButton());
        await new Promise(resolve => setTimeout(resolve, this.animationDuration.split("s")[0] * 1000 / 2));
      }
      this.canPlay = true
    },
    playButton(btn) {
      this.activeButton = btn;
      this.isPlayingSequence = true;
      this.playButtonSound(btn)
      return new Promise(resolve => {
        setTimeout(() => {
          this.stopPlayButton();
          resolve();
        }, this.animationDuration.split("s")[0] * 1000);
      });
    },

    playButtonSound(btn) {
      let audio = null
      switch (btn) {
        case "Red":
          audio = new Audio(Red);
          audio.play();
          break;
        case "Green":
          audio = new Audio(Green);
          audio.play();
          break;
        case "Blue":
          audio = new Audio(Blue);
          audio.play();
          break;
        case "Yellow":
          audio = new Audio(Yellow);
          audio.play();
          break;
      }
    },

    stopPlayButton() {
      this.activeButton = null;
      this.isPlayingSequence = false;
    },
    onHandleClickPlayer(btn) {
      console.log(this.isPlayingSequence)
      if (this.isPlayingSequence || !this.canPlay) {
        return
      }

      if (this.isRoundOver) {
        return
      }

      this.activeButton = null

      if (this.playerSequence.length < this.sequence.length) {
        this.playerSequence.push(btn)
        this.playButton(btn);

        if (this.isValidSequence()) {
          if (this.playerSequence.length === this.sequence.length) {
            this.isRoundOver = true
            this.playerSequence = []
          }
        } else {
          this.handleGameOver()
        }
      } else
        this.isRoundOver = true
    },
    async nextRound() {
      this.activeButton = null;
      this.canPlay = false
      await new Promise(resolve => setTimeout(resolve, 100));
      this.round++;
      this.generateSequence();
      await this.playSequence();
    },
    isValidSequence() {
      for (let i = 0; i < this.playerSequence.length; i++) {
        if (this.playerSequence[i] !== this.sequence[i]) {
          return false
        }
      }
      return true

    },
    handleGameOver() {
      this.playerSequence = []
      this.sequence = []
      this.isRoundOver = true
      this.activeButton = null
      this.isGameOver = true
      this.isGameBegin = false
      const sound = new Audio(EndGame);
      sound.play();
    },
    getButtonClass(btn) {
      return btn.toLowerCase();
    },
  },
};
</script>

<style lang="scss" scoped>
.buttons .active {
  animation: activeBtn var(--animation-duration) linear;
}

@keyframes activeBtn {
  0% {
    opacity: 1;
  }
  25% {
    opacity: 0.5;
  }
  50% {
    opacity: 0;
  }
  75% {
    opacity: 0.5;
  }
  100% {
    opacity: 1;
  }
}
</style>