<template lang='pug'>
  #app
    div
      transition(name='fade')
        div(v-show='!appRunning')#first-div
          .text-center
            h1.md-title VueTrivia
            .md-subhead Made with Vue and Material Design
          .text-center
            md-button.md-raised.md-primary(v-on:click.native="startTrivia") GO
          div(v-if='showEndScreen').text-center
            p Your final score is {{ triviaCounter }} / {{ nOfQuestions }}.


    transition(name='fade')
      md-layout.main-div(v-if='appRunning' md-align='center')#second-div
        md-layout(md-flex='50')
          div(style='width: 100%')
            h3 {{ this.currentQuestion + 1 }} / 5
          p#question {{ question }}
          md-layout(md-flex='100' md-align='center').answer-container
            button.answer-box#answer1(@click='chooseAnswer(answerArray[0], "answer1")' v-bind:class='[{ correct: is1Correct }, { false: is1False }]')
              p {{ answerArray[0] }}
          md-layout(md-flex='100' md-align='center').answer-container
            button.answer-box#answer2(@click='chooseAnswer(answerArray[1], "answer2")' v-bind:class='[{ correct: is2Correct }, { false: is2False }]')
              p {{ answerArray[1] }}
          md-layout(md-flex='100' md-align='center').answer-container
            button.answer-box#answer3(@click='chooseAnswer(answerArray[2], "answer3")' v-bind:class='[{ correct: is3Correct }, { false: is3False }]')
              p {{ answerArray[2] }}
          md-layout(md-flex='100' md-align='center').answer-container
            button.answer-box#answer4(@click='chooseAnswer(answerArray[3], "answer4")' v-bind:class='[{ correct: is4Correct }, { false: is4False }]')
              p {{ answerArray[3] }}
          div(v-show='showNext' @click='nextQuestion').md-48#next-btn
            i.material-icons arrow_forward
          <!-- h3#trivia-count Correct: {{ triviaCounter }} -->







</template>

<script>
import axios from 'axios'
import shuffle from 'lodash/shuffle'
import he from 'he'

export default {
  name: 'app',
  components: {
  },
  data () {
    return {
      nOfQuestions: 5,
      appRunning: false,
      triviaCounter: 0,
      triviaArray: '',
      currentQuestion: 0,
      question: 'Default question here.',
      answerArray: [],
      correctAnswer: '',
      is1Correct: false,
      is2Correct: false,
      is3Correct: false,
      is4Correct: false,
      is1False: false,
      is2False: false,
      is3False: false,
      is4False: false,
      showNext: false,
      showEndScreen: false
    }
  },
  methods: {
    startTrivia () {
      if (!this.appRunning) {
        axios('https://opentdb.com/api.php?amount=' + this.nOfQuestions + '&type=multiple')
        .then((response) => {
          // Resetting Vue data to default
          this.currentQuestion = 0
          this.triviaCounter = 0
          this.answerArray = []
          this.is1Correct = false
          this.is2Correct = false
          this.is3Correct = false
          this.is4Correct = false
          this.is1False = false
          this.is2False = false
          this.is3False = false
          this.is4False = false
          this.showNext = false
          this.triviaArray = response.data
          this.appRunning = true
          this.question = he.decode(this.triviaArray.results[0].question)
          this.getCurrentQuestion
        })
        .catch(function (error) {
          console.log('Error!: ', error)
        })
      }
    },
    shuffleArray: function () {
      this.answerArray = shuffle(this.answerArray)
    },
    chooseAnswer: function (answer, answerId) {
      if (answer === this.correctAnswer) {
        this.triviaCounter += 1
        switch (answerId) {
          case 'answer1':
            this.is1Correct = true
            break
          case 'answer2':
            this.is2Correct = true
            break
          case 'answer3':
            this.is3Correct = true
            break
          case 'answer4':
            this.is4Correct = true
            break
        }
      } else {
        switch (answerId) {
          case 'answer1':
            this.is1False = true
            break
          case 'answer2':
            this.is2False = true
            break
          case 'answer3':
            this.is3False = true
            break
          case 'answer4':
            this.is4False = true
            break
        }
        if (document.getElementById('answer1').textContent === this.correctAnswer) {
          this.is1Correct = true
        } else if (document.getElementById('answer2').textContent === this.correctAnswer) {
          this.is2Correct = true
        } else if (document.getElementById('answer3').textContent === this.correctAnswer) {
          this.is3Correct = true
        } else {
          this.is4Correct = true
        }
      }
      this.showNext = true
      this.disableElements()
    },
    nextQuestion () {
      if (this.currentQuestion < this.nOfQuestions - 1) {
        this.answerArray = []
        this.currentQuestion += 1
        this.question = he.decode(this.triviaArray.results[this.currentQuestion].question)
        this.is1Correct = false
        this.is2Correct = false
        this.is3Correct = false
        this.is4Correct = false
        this.is1False = false
        this.is2False = false
        this.is3False = false
        this.is4False = false
        this.showNext = false
        this.getCurrentQuestion
        this.enableElements()
      } else {
        this.showEndScreen = true
        this.appRunning = false
      }
    },
    disableElements () {
      var elements = document.getElementsByClassName('answer-box')
      for (let i = 0; i < elements.length; i++) {
        elements[i].disabled = true
      }
    },
    enableElements () {
      var elements = document.getElementsByClassName('answer-box')
      for (let i = 0; i < elements.length; i++) {
        elements[i].disabled = false
      }
    }
  },
  computed: {
    getCurrentQuestion: function () {
      this.answerArray.push(he.decode(this.triviaArray.results[this.currentQuestion].correct_answer))
      this.correctAnswer = this.triviaArray.results[this.currentQuestion].correct_answer
      this.triviaArray.results[this.currentQuestion].incorrect_answers.map(q => {
        q = he.decode(q)
        this.answerArray.push(q)
      })
      this.shuffleArray()

      return this.triviaArray.results[this.currentQuestion]
    },
    getCurrentCategory: function () {
      return this.triviaArray.results[this.currentQuestion].category
    }
  }
}
</script>

<style lang='scss'>
@import url('https://fonts.googleapis.com/css?family=Righteous');
$lblue: #2196f3;
$oj: #ff5722;
body {
  padding: 10px;
}
.md-title {
  font-family: 'Righteous', cursive;
  font-size: 3.5em !important;
}
.text-center {
  text-align: center
}
.main-card {
  padding: 30px;
}
#first-div {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  button {
    margin: 16px;
  }
}
#second-div {
  width: 100%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
#trivia-count {
  margin: 16px;
}
#question {
  font-size: 2em;
  line-height: 1.3;
}
#next-btn {
  position: absolute;
  bottom: 50%;
  right: 0;
  margin: 24px;

  font-size: 48px;

  cursor: pointer;
  cursor: hand;

}

.main-div {
  margin-top: 24px;
}
.correct {
  background-color: $lblue;
  color: white;
}
.false {
  background-color: $oj;
  color: white;
}
#diff-icon {
  /*text-align: right;*/
}
.difficulty-easy {
  color: blue;
}
/*.answer-container {
  :hover {
    background-color: lighten($lblue, 30);
  }
}*/
.answer-box {
  display: inline;
  border: 2px solid #333;
  border-radius: 24px;
  width: 100%;
  text-align: center;
  margin: 5px;

  color: #333;
  font-size: 1.3em;

  /*:hover {
    background-color: $lblue;
    color: white;
  }*/
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s
}
.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
  opacity: 0
}
</style>
