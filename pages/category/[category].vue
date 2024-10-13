<template>
  <div class="max-w-2xl mx-auto py-10 px-4">
    <h1 v-if="!quizFinished" class="text-3xl font-bold text-center mb-8">Nuxt Quiz App</h1>

    <div v-if="loading" class="text-center">Loading...</div>

    <div v-if="!quizFinished && !loading" class="space-y-6">
      <div v-if="currentQuestion">
        <h3 class="text-xl md:text-2xl font-semibold mb-2">
          {{ currentQuestion.question }}
        </h3>
        <ul class="space-y-4">
          <li v-for="(answer, index) in currentQuestion.answers" :key="index">
            <button
              @click="checkAnswer(index)"
              :class="[
                {
                  'bg-blue-500 hover:bg-blue-600': userSelectedAnswer === null,
                  'bg-green-500':
                    userSelectedAnswer === index &&
                    index === currentQuestion.correctAnswerIndex,
                  'bg-red-500':
                    userSelectedAnswer === index &&
                    index !== currentQuestion.correctAnswerIndex,
                },
                'text-white py-2 px-4 rounded-md transition-colors w-full',
              ]"
              :disabled="userSelectedAnswer !== null"
            >
              {{ answer }}
            </button>
          </li>
        </ul>
      </div>
      <p class="text-gray-600">
        Question {{ currentQuestionIndex + 1 }} of {{ questions.length }}
      </p>
    </div>

    <div v-else-if="quizFinished" class="text-center space-y-4">
      <h2 class="text-2xl md:text-3xl font-bold">Quiz Finished</h2>
      <p class="text-lg">Your score: {{ score }}/{{ questions.length }}</p>
      <div class="mt-4">
        <h3 class="text-lg font-semibold">Review Your Answers:</h3>
        <ul class="space-y-2">
          <li
            v-for="(question, index) in questions"
            :key="index"
            class="bg-gray-100 p-4 rounded-md shadow-sm"
          >
            <p class="font-bold">{{ index + 1 }}. {{ question.question }}</p>
            <p class="text-gray-700">
              Your answer:
              <span
                :class="{
                  'text-red-500':
                    userAnswers[index] !== question.correctAnswerIndex,
                  'text-green-500':
                    userAnswers[index] === question.correctAnswerIndex,
                }"
              >
                {{ question.answers[userAnswers[index]] }}
              </span>
            </p>
            <p class="text-gray-500">
              Correct answer:
              {{ question.answers[question.correctAnswerIndex] }}
            </p>
          </li>
        </ul>
      </div>

      <button
        @click="restartQuiz"
        class="bg-green-500 hover:bg-green-600 text-white py-2 px-4 rounded-md transition-colors"
      >
        Restart Quiz
      </button>
    </div>
  </div>
</template>

<script>
import { createClient } from "@supabase/supabase-js";
import { useRoute } from "vue-router";
export default {
  data() {
    const runtimeConfig = useRuntimeConfig();
    const supabase = createClient(
      runtimeConfig.public.SUPABASE_URL,
      runtimeConfig.public.SUPABASE_KEY
    );
    return {
      supabase,
      questions: [],
      currentQuestionIndex: 0,
      score: 0,
      quizFinished: false,
      userSelectedAnswer: null,
      loading: true,
      quiz_category: "",
      userName: "",
      userAnswers: [],
    };
  },
  computed: {
    currentQuestion() {
      return this.questions[this.currentQuestionIndex];
    },
  },
  async mounted() {
    const route = useRoute();
    const category = route.params.category;
    this.quiz_category = category;
    this.userName = route.query.userName;
    const { data, error } = await this.supabase
      .from("question")
      .select("*")
      .eq("category", category);

    if (error) {
      console.error("Error fetching questions:", error);
    } else {
      this.questions = data.map((item) => ({
        question: item.question,
        answers:
          typeof item.answers === "string"
            ? JSON.parse(item.answers)
            : item.answers,
        correctAnswerIndex: item.correct_answer - 1,
      }));
    }

    this.loading = false;
  },
  methods: {
    checkAnswer(index) {
      this.userSelectedAnswer = index;
      this.userAnswers[this.currentQuestionIndex] = index;
      if (index === this.currentQuestion.correctAnswerIndex) {
        this.score++;
      }

      setTimeout(async () => {
        if (this.currentQuestionIndex < this.questions.length - 1) {
          this.currentQuestionIndex++;
          this.userSelectedAnswer = null;
        } else {
          const userToInsert = {
            created_at: new Date().toISOString(),
            score: this.score,
            category: this.quiz_category,
            userName: this.userName,
          };
          const { data, error } = await this.supabase
            .from("users")
            .insert([userToInsert])
            .select();
          if (error) {
            console.error("Error save result:", error);
          } else {
            console.log("Result saved successfully:", data);
          }
          this.quizFinished = true;
        }
      }, 1000);
    },
    restartQuiz() {
      this.currentQuestionIndex = 0;
      this.score = 0;
      this.quizFinished = false;
      this.userSelectedAnswer = null;
      this.$router.push({ path: `/` });
    },
  },
};
</script>

<style scoped></style>
