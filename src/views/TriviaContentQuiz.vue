<template>
  <title>
    Trivia Management System | Elfrique – Complete Event Management System
  </title>
  <elfrique-header />

  <section class="voting-content trivia-content">
    <div class="container header-cont">
      <div class="row">
        <div class="col-md-4">
          <div class="img-area">
            <img :src="trivia.image" />
          </div>
        </div>
        <div class="col-md-6">
          <div class="text-title-area">
            <h1>{{ trivia.title }}</h1>
            <small
              >Organised by :
              <span
                >{{ trivia.adminuser.profile.firstname }}
                {{ trivia.adminuser.profile.lastname }}</span
              ></small
            >
            <div class="details-header">
              <h5>Price</h5>
              <p><i class="bi bi-credit-card-fill"></i> : {{ trivia.type }}</p>
              <h5>Duration</h5>
              <p><i class="bi bi-alarm"></i> : {{ trivia.duration }} Minutes</p>
            </div>
            <div class="details-social">
              <h5>Share on:</h5>
              <ShareNetwork
                network="facebook"
                :url="currentUrl"
                :title="trivia.title"
                :description="trivia.details"
                :quote="trivia.title"
                :hashtags="'Elfrique, Trivia, Quiz,' + trivia.title"
              >
                <img src="@/assets/images/share-facebook.png" />
              </ShareNetwork>
              <ShareNetwork
                network="whatsapp"
                :url="currentUrl"
                :title="trivia.title"
                :description="trivia.details"
              >
                <img src="@/assets/images/share-whatsapp.png" />
              </ShareNetwork>
              <ShareNetwork
                network="telegram"
                :url="currentUrl"
                :title="trivia.title"
                :description="trivia.details"
              >
                <img src="@/assets/images/share-telegram.png" />
              </ShareNetwork>
              <ShareNetwork
                network="twitter"
                :url="currentUrl"
                :title="trivia.title"
                twitter-user="@elfrique"
                :hashtags="'Elfrique, Trivia, Quiz,' + trivia.title"
              >
                <img src="@/assets/images/share-twitter.png" />
              </ShareNetwork>
              <ShareNetwork
                network="email"
                :url="currentUrl"
                :title="trivia.title"
                :description="trivia.details"
              >
                <img src="@/assets/images/share-email.png" />
              </ShareNetwork>
            </div>
          </div>
        </div>
        <div class="col-md-2 justify-content-center text-center">
          <div class="counter-div">
              <div class="icon">
                  <i class="bi bi-alarm-fill"></i>
              </div>
              <!-- <div class="boxes days">
                  <span class="title">Days</span> <br>
                  <span id="days"></span>
              </div>
              <div class="boxes hours">
                  <span class="title">Hours</span> <br>
                  <span id="hours"></span>
              </div> -->
              <div class="boxes minutes">
                  <span class="title">Min</span> <br>
                  <span id="minutes"></span>
              </div>
              <div class="boxes seconds">
                  <span class="title">Sec</span> <br>
                  <span id="seconds"></span>
              </div>
              <div class="clear"></div>
              <p id="timeUpText"></p>
          </div>
        </div>
      </div>
    </div>

    <div class="container">
      <div class="row">
        <div  class="col-lg-12 mt-3 quiz-box">
          <span style="color: red;text-align:center">{{errMessage}}</span>
          <form>
            <div  class="question mb-4">
              <div v-for="(question, index) in questions" :key="question.id">
              <div v-show="index === questionIndex">
                <h1>{{ question.question }}</h1>
                <div v-for="opt in question.questionOptions" :key="opt.id">
                  <input v-model="check" @change="selectOption(opt)" type="radio" v-bind:name="index" :value="opt"/>
                  <span>{{ opt.option }}</span>
                </div>
              </div>
              </div>
            </div>
          </form>
            <div>
              <button v-if="questionIndex > 0" @click="prev">
                  Previous
                </button>
                
                <button style="margin-left: 20px" v-if="questionIndex < (questions.length - 1)" @click="next">
                  Next
                </button>
              <button v-if="questionIndex == (questions.length - 1)" style="margin-left: 30px" @click="submitQuiz" type="submit">Submit Quiz</button>
            </div>
          
        </div>
      </div>
    </div>
  </section>

  <elfrique-footer />
</template>

<script>
import Header from "./elfrique-header.vue";
import Footer from "./elfrique-footer.vue";
import TriviaService from "../service/trivia.service";
export default {
  name: "Elfrique",
  components: {
    "elfrique-header": Header,
    "elfrique-footer": Footer,
  },
  data() {
    return {
      trivia: "",
      loading: false,
      questions: "",
      answer: {
        "playerEmail": this.$store.state.vote.player.email,
        "trivia_answer": []
      },
      questionIndex: 0,
      check: '',
      errMessage: '',
      ended:false,
      endDate: "",
      countdown: {
        months: 0,
        days: 0,
        hours: 0,
        minutes: 0,
        seconds: 0,
      },
    };
  },
  computed: {
    player() {
      return this.$store.state.vote.player;
    },
    currentUrl() {
      return window.location.href;
    },
  },
  created() {
    TriviaService.getSingleTrivia(this.$route.params.id)
      .then((response) => {
        this.trivia = response.data.trivia;
        this.questions = response.data.trivia.questions;
      })
      .then(() => {
        if (this.player.id == null) {
          this.$router.push("/trivia-content/" + this.trivia.id);
        }
      });
  },
  methods: {
    startQuiz() {
      this.$router.push("/trivia-content-quiz/" + this.trivia.id);
    },
    next() {
      this.questionIndex++;
    },
    // Go to previous question
    prev() {
      this.questionIndex--;
    },
    selectOption(index){
      
      const elementIndex = this.answer.trivia_answer.findIndex((obj => obj.questionId === index.questionId));
      if(elementIndex == -1){
        this.answer.trivia_answer.push({
          questionId: index.questionId, 
          answer: index.option
        })
      }else{
        this.answer.trivia_answer[elementIndex].answer = index.option;
      }
      
    },
    submitQuiz(){
      this.loading = true
      if (this.check == "") {
        this.errMessage = "Please answer the questions before submitting quiz"
        setTimeout(() => {
          this.errMessage = ""
        }, 4000);
      } else {
        TriviaService.answerTrivia( this.trivia.id, this.answer).then(res => {
          this.loading = false
          let data = res.data.data
          this.$router.push({ name: 'TriviaResult', params:{ data: JSON.stringify(data) } })
        })
      }
      
    }
  },
  mounted() {
    window.scrollTo(0, 0);
  },
};
</script>