<template>
  <div class="container jumbotron">
    <h1 class="display-4">RunTime Terrors</h1>
    <p class="lead">
      Code-A-Thon</p>
    <hr class="my-4">
  

    <div>
      <div v-if="lmode" class="card">
        <div class="card-body">
          <span v-for="(word,key) in words.filter((data,index)=>index<20)" :key="key" v-bind:class="key!=0 || writingWordControl " class="words ml-2">
            {{word}}
          </span>
        </div>
      </div>
      <div v-if="pmode" class="col-sm-12 scroll">
      <div class="card">
      
         
          <span v-for="(word,key) in words.filter((data,index)=>index<20)" :key="key" v-bind:class="key!=0 || writingWordControl " class="words ml-2">
            <pre style="font-family:arial;color:191970;font-size:10px;line-height:.1;">{{word}}</pre> 
          </span>
      </div>
    </div>
      <div class="card">
        <div class="card-body bg-secondary">
          <div class="input-group input-group-lg">
            <input type="text" class="form-control" v-model="writingWord">
            <div class="input-group-append" id="button-addon4">
              <button class="btn btn-light" disabled type="button">{{timer}} sec</button>
              <button class="btn btn-light" type="button" @click="modeChange">Mode Change</button>
            </div>
          </div>
         
        </div>
      </div>
    </div>
    <div class="alert alert-primary">
      <h3>Progress</h3>
      <h3><p class="display-4">Attempted {{dks}} Words </p></h3>
      <h3>Accuracy : {{truePercent}} % </h3><br>
      <h3>Correct: {{trueCount}} </h3><br>
      <h3>Incorrect : {{falseCount}} </h3><br>
      <h3>WPM : {{wpm}}</h3><br>
      <h3>Error Rate : {{errorRate}}% </h3><br>
      <h3>Raw Speed : {{falseCount}} </h3><br>
      <h3>Key Speed : {{falseCount}} </h3><br>
      <h3>Completed Words : {{falseCount}} </h3><br>
      <h3>Total time: {{timer}} </h3><br>
      <button @click="newGame" class="btn btn-success mt-5 btn-lg btn-block">Start New</button>
    </div>
  </div>
</template>

<script>

import wordList from '@/assets/words.json'
import test1 from '@/assets/test.json'
export default {

  data () {
    return {
      words: [],
      writingWord: null,
      isTrue: true,
      trueCount: 0,
      falseCount: 0,
      total: 0,
      wpm: 0.0,
      timer: 30,
      errorRate :0.0,
      interval: false,
      isRunning: false,
      isFinish: false,
      wordList: wordList,
      test1:test1,
      pmode :false,
      lmode: true
    }
  },

  watch: {
    writingWord (val) {
      if (!val || val === ' ') {
        this.writingWord = ''
        return
      }
      if (!this.isRunning) this.toggleTimer()

      const word = this.words[0].slice(0, val.length).toUpperCase()
      const userWord = val.replace(' ', '').toUpperCase()
      // this.ftime = this.timer
      this.isTrue = word === userWord

      if (val.indexOf(' ') !== -1) {
        this.isTrue ? this.trueCount++ : this.falseCount++
        this.isTrue ? this.total++ : this.total++
        this.words.splice(0, 1)
        this.writingWord = ''
        this.isTrue = true
        this.wpm = this.total / (30/60)
        this.errorRate = ((this.falseCount / this.total ) * 100).toFixed(2) 
      }
    }
  },

  computed: {
    writingWordControl () {
      return this.isTrue ? 'writing-word' : 'writing-word bg-danger'
    },
    dks () {
      return 300 - this.words.length
    },
    truePercent () {
      const percent = (100 / this.dks)
      const val = (percent * this.trueCount)
      return isNaN(val) ? 0 : val.toFixed(2)
    },
    test () {
      return 300 - this.words.length
    }
  },

  // this.wpm = (this.timer / 60)
  mounted () {
    this.getWords()
  },

  methods: {
    newGame () {
      this.getWords()
      this.isFinish = false
      this.timer = 30
      this.isTrue = true
      this.isRunning = false
      this.writingWord = ''
      window.location.reload()
    },

    getWords () {
      this.words = this.wordList.sort(() => Math.random() - 0.5).splice(0, 300)
    },
    modeChange () {
      // if (this.lmode==false) {
      //   this.pmode=false
      //   this.lmode=true
      // }
      if(this.pmode==false){
        this.lmode=false
        this.pmode=true
      }else {
        this.pmode=false
        this.lmode=true
      }
      
    },

    toggleTimer () {
      this.isRunning = true
      this.interval = setInterval(this.timeProcess, 1000)
    },
    timeProcess () {
      if (this.timer === 0) {
        clearInterval(this.interval)
        this.isFinish = true
        return
      }
      this.timer--
    }
  }

}

</script>

<style>
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
  .words {
    font-size: 25px;
    font-weight: 400;
  }

  .writing-word {
    background-color: slategrey;
    color: white;
    padding: 5px;
    border-radius: 5px;
  }
  .scroll {
    max-height: 100px;
    overflow-y: auto;
}

</style>
