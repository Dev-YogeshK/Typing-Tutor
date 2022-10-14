<template>
    <div>
      <h1 id="game_slogan" v-if="!state.display" style="display: block">
        How fancy are your fingers?
      </h1>
      <p><em>Note: </em> This project works only on physical keyboard, not touch screen.</p>
      <div class="gameBar">
        <div class="timer" v-if="state.display" style="opacity: 1">
          {{ `Timer: ${state.count} sec` }}
        </div>
        <p
          v-if="state.display"
          style="
            opacity: 1;
            background-color: yellow;
            color: #000;
            padding: 1rem;
            font-weight: 900;
          "
        >
          Press enter to submit progress
        </p>
        <div class="score">Score: {{ score }}</div>
      </div>
      <div v-if="state.display" style="display: block">
        <span
          v-bind:class="{
            correct: word.correct,
            wrong: word.wrong,
            pending: word.pending,
          }"
          :key="word.position"
          v-for="word in state.fetchedText"
          >{{ word.text }} {{ "" }}</span
        >
      </div>
      <textarea
        name=""
        id=""
        cols="30"
        rows="10"
        @keyup.space="processInput($event)"
        @keyup.enter="submitProgress($event)"
        v-if="state.display"
        style="display: block"
      ></textarea>
      <button
        @click="fetchData(), runTimer()"
        id="startButton"
        v-if="!state.display"
        style="display: inline"
      >
        Start game
      </button>
      <button
        @click="stopTimer(), (state.index = 0)"
        id="stopButton"
        v-if="state.display"
        style="display: inline"
      >
        Quit
      </button>
    </div>
  </template>
  
  <script>
  import { reactive, computed } from "vue";
  export default {
    setup() {
      const state = reactive({
        inputValue: "",
        index: 0,
        display: false,
        count: 0,
        fetchedText: [],
        poems: [
          "my_name_in_love_language.txt",
          "she_loves_me.txt",
          "when_I_find_you.txt",
        ],
      });
      let score = computed(() => {
        return `${state.fetchedText.filter((word) => word.correct).length} / ${
          state.fetchedText.length
        }`;
      });
      function randPoem() {
        let r = Math.floor(Math.random() * state.poems.length);
        return r;
      }
      async function fetchData() {
        let response = await fetch(`text/${state.poems[randPoem()]}`);
        let data = await response.text();
        let obj = data
          .trim()
          // Include non space characters including \r or \n
          .split(/\s+/)
          .map((word, index) => {
            return {
              position: index,
              text: word,
              correct: false,
              wrong: false,
              pending: true,
              myTimer: "",
            };
          });
        return (state.fetchedText = obj);
      }
      function runTimer() {
        state.display = true;
        state.index = 0;
        state.myTimer = setInterval(() => {
          ++state.count;
        }, 1000);
      }
      function stopTimer() {
        state.count = 0;
        state.display = false;
        clearInterval(state.myTimer);
      }
      function submitProgress(event) {
        event.preventDefault();
        // Include non space characters including \r or \n
        state.inputValue = event.target.value.trim().split(/\s+/);
        this.stopTimer()
        //To avoid index error
        if (state.index >= state.fetchedText.length) {
          state.display = false;
          return;
        }
        if (
          state.fetchedText[state.index].text ===
          state.inputValue[state.inputValue.length - 1]
        ) {
          //correct answer
          state.fetchedText[state.index].correct = true;
          state.fetchedText[state.index].wrong = false;
          state.fetchedText[state.index].pending = false;
        } else {
          //wrong answer
          state.fetchedText[state.index].correct = false;
          state.fetchedText[state.index].wrong = true;
          state.fetchedText[state.index].pending = false;
        }
        ++state.index;
      }
      function processInput(event) {
        event.preventDefault();
        // Include non space characters including \r or \n
        state.inputValue = event.target.value.trim().split(/\s+/);
        if (state.inputValue === "") {
          return;
        }
        //To avoid index error
        if (state.index >= state.fetchedText.length) {
          state.display = false;
          return;
        }
        if (
          state.fetchedText[state.index].text ===
          state.inputValue[state.inputValue.length - 1]
        ) {
          //correct answer
          state.fetchedText[state.index].correct = true;
          state.fetchedText[state.index].wrong = false;
          state.fetchedText[state.index].pending = false;
        } else {
          //wrong answer
          state.fetchedText[state.index].correct = false;
          state.fetchedText[state.index].wrong = true;
          state.fetchedText[state.index].pending = false;
        }
        ++state.index;
      }
      return {
        state,
        score,
        randPoem,
        fetchData,
        runTimer,
        stopTimer,
        submitProgress,
        processInput,
      };
    },
  };
  </script>
  
  <!-- Using sass -->
  <style lang="scss">
  body {
    user-select: none;
    background-color: #666f70;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  #app {
    background-color: #f6fdff;
    max-width: 50%;
    border-radius: 8px;
    padding: 1rem;
    box-shadow: 0px 0px 10px #929292;
    button {
      font-weight: bold;
      color: #fff;
      cursor: pointer;
      background-color: #663399;
    }
    #game_slogan {
      text-align: center;
      margin: 0px;
      display: none;
    }
    .gameBar {
      display: flex;
      justify-content: space-between;
      margin-bottom: 4px;
      .timer {
        opacity: 0;
      }
      .score,
      .timer {
        background-color: #111;
        color: #fefefe;
        min-width: 20px;
        max-width: 100px;
        height: 100%;
        padding: 4px;
        border-radius: 4px;
      }
    }
    .pending {
      font-weight: bold;
    }
    .correct {
      font-weight: bold;
      color: blue;
    }
    .wrong {
      font-weight: bold;
      border-bottom: 2px dotted #ff0000;
    }
  }
  textarea {
    width: 99%;
    max-width: 800px;
    border-radius: 8px;
    display: none;
    &:focus {
      outline: 2px solid #03bcdd;
      border-radius: 0px;
    }
    &::placeholder {
      padding: 1rem;
      font-size: larger;
      color: #757272;
      margin: auto;
    }
    #startButton {
      display: none;
    }
    #stopButton {
      display: none;
    }
  }
  </style>