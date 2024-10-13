<template>
  <div class="max-w-4xl mx-auto p-6 bg-gray-100 rounded-lg">
    <h1 class="text-3xl font-bold text-center mb-6 text-red-500">
      Admin Quiz Management
    </h1>

    <!-- Tab Navigation -->
    <div class="flex justify-center space-x-4 mb-6">
      <button
        :class="{
          'bg-blue-500 text-white': activeTab === 'questions',
          'text-blue-500': activeTab !== 'questions',
        }"
        @click="activeTab = 'questions'"
        class="px-4 py-2 rounded-md focus:outline-none"
      >
        Quiz Management
      </button>
      <button
        :class="{
          'bg-blue-500 text-white': activeTab === 'scores',
          'text-blue-500': activeTab !== 'scores',
        }"
        @click="activeTab = 'scores'"
        class="px-4 py-2 rounded-md focus:outline-none"
      >
        User Scores
      </button>
    </div>
    <!-- Form to create a new question -->
    <div v-if="activeTab === 'questions'">
      <form @submit.prevent="addQuestion" class="space-y-6">
        <div>
          <label class="block text-lg font-medium mb-2">Question:</label>
          <input
            v-model="newQuestion.question"
            type="text"
            required
            class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>

        <div>
          <label class="block text-lg font-medium mb-2">Answers:</label>
          <div
            v-for="(answer, index) in newQuestion.answers"
            :key="index"
            class="mb-2"
          >
            <input
              v-model="newQuestion.answers[index]"
              type="text"
              placeholder="Answer"
              required
              class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
        </div>

        <div>
          <label class="block text-lg font-medium mb-2"
            >Correct Answer Index (1-4):</label
          >
          <input
            v-model.number="newQuestion.correctAnswerIndex"
            type="number"
            min="1"
            max="4"
            required
            class="w-16 px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>

        <div>
          <label class="block text-lg font-medium mb-2">Category:</label>
          <input
            v-model="newQuestion.category"
            type="text"
            placeholder="Category"
            required
            class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>

        <button
          type="submit"
          class="px-6 py-2 bg-blue-500 text-white font-semibold rounded-md hover:bg-blue-600 transition-colors"
        >
          Add Question
        </button>
      </form>

      <hr class="my-6 border-t-2 border-gray-200" />

      <!-- List of existing questions with delete option -->
      <div v-if="questions.length">
        <div class="flex justify-between items-center mb-4">
          <h2 class="text-2xl font-bold">Existing Questions</h2>
          <select
            v-model="selectedCategory"
            @change="filterByCategory"
            class="px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="" selected>All Categories</option>
            <option
              v-for="(category, index) in uniqueCategories"
              :key="index"
              :value="category"
            >
              {{ category }}
            </option>
          </select>
        </div>
        <ul class="space-y-4">
          <li
            v-for="(question, index) in questions"
            :key="index"
            class="bg-white p-4 rounded-lg shadow-md"
          >
            <p class="text-lg font-semibold mb-2">{{ question.question }}</p>
            <ul class="list-disc pl-6 mb-4">
              <li
                v-for="(answer, i) in question.answers"
                :key="i"
                :class="{
                  'font-bold text-green-600': i + 1 === question.correct_answer,
                }"
              >
                {{ answer }}
              </li>
            </ul>
            <p class="mb-4">
              Correct answer:
              {{ question.answers[question.correct_answer - 1] }}
            </p>
            <!-- Displaying answer directly -->
            <div class="flex gap-2">
              <button
                @click="openUpdateModal(question)"
                class="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 transition-colors"
              >
                Update
              </button>
              <button
                @click="deleteQuestion(question.id)"
                class="px-4 py-2 bg-red-500 text-white rounded-md hover:bg-red-600 transition-colors"
              >
                Delete
              </button>
            </div>
          </li>
        </ul>
      </div>
      <div
        v-if="isModalOpen"
        class="fixed inset-0 bg-black bg-opacity-60 flex items-center justify-center"
      >
        <div class="bg-white p-8 rounded-lg shadow-xl max-w-md w-full">
          <h3 class="text-2xl font-bold mb-6 text-center">Update Question</h3>

          <input
            type="text"
            v-model="updatedQuestion.question"
            class="p-3 border border-gray-300 rounded-md w-full mb-4 focus:outline-none focus:ring-2 focus:ring-blue-500"
            placeholder="Question"
          />

          <div
            v-for="(answer, index) in updatedQuestion.answers"
            :key="index"
            class="mb-3"
          >
            <input
              type="text"
              v-model="updatedQuestion.answers[index]"
              class="p-3 border border-gray-300 rounded-md w-full focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Answer"
            />
          </div>

          <select
            v-model="updatedQuestion.correct_answer"
            class="p-3 border border-gray-300 rounded-md w-full mb-6 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option disabled value="">Select Correct Answer</option>
            <option
              v-for="(answer, index) in updatedQuestion.answers"
              :key="index"
              :value="index + 1"
            >
              {{ answer }}
            </option>
          </select>

          <div class="flex justify-between">
            <button
              @click="updateQuestionInDB"
              class="w-1/2 mr-2 px-4 py-2 bg-green-500 text-white rounded-md hover:bg-green-600 transition-colors"
            >
              Save
            </button>
            <button
              @click="closeModal"
              class="w-1/2 ml-2 px-4 py-2 bg-gray-300 text-black rounded-md hover:bg-gray-400 transition-colors"
            >
              Cancel
            </button>
          </div>
        </div>
      </div>
      <div v-else>
        <p class="text-gray-600 text-center">No questions available.</p>
      </div>
    </div>
    <!-- User Scores Section -->
    <div v-if="activeTab === 'scores'" class="mt-6">
      <h2 class="text-2xl font-bold mb-4">User Scores</h2>
      <div class="overflow-x-auto">
        <!-- Added overflow-x-auto for responsiveness -->
        <table class="min-w-full bg-white border border-gray-300">
          <thead>
            <tr class="bg-gray-200">
              <th class="py-2 px-4 border text-left">User</th>
              <th class="py-2 px-4 border text-left">Score</th>
              <th class="py-2 px-4 border text-left">Date</th>
              <th class="py-2 px-4 border text-left">Category</th>
              <!-- New Category Column -->
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(score, index) in userScores"
              :key="index"
              class="hover:bg-gray-100"
            >
              <!-- Added hover effect -->
              <td class="py-2 px-4 border">{{ score.userName }}</td>
              <td class="py-2 px-4 border">{{ score.score }}</td>
              <td class="py-2 px-4 border">{{ score.created_at }}</td>
              <td class="py-2 px-4 border">{{ score.category }}</td>
              <!-- Display Category -->
            </tr>
            <tr v-if="userScores.length === 0">
              <td
                colspan="4"
                class="py-2 px-4 border text-center text-gray-600"
              >
                No scores available.
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <!-- Display Average Scores -->
      <div class="mt-4">
        <h3 class="text-xl font-semibold mb-4">
          Average Scores and User Count by Category:
        </h3>
        <div class="bg-gray-100 p-4 rounded-lg shadow-md">
          <ul class="space-y-2">
            <li
              v-for="(avg, category) in categoryAverages"
              :key="category"
              class="flex justify-between p-2 border-b last:border-b-0"
            >
              <div>
                <span class="text-gray-700 font-medium">{{ category }}</span>
                <p class="text-gray-500 text-sm">Users: {{ avg.userCount }}</p>
              </div>
              <span class="font-bold text-blue-600">{{ avg.average }}</span>
            </li>
          </ul>
          <p
            v-if="Object.keys(categoryAverages).length === 0"
            class="text-center text-gray-600 mt-2"
          >
            No average scores or user counts available.
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { createClient } from "@supabase/supabase-js";
import dayjs from "dayjs";
export default {
  data() {
    const runtimeConfig = useRuntimeConfig();
    const supabase = createClient(
      runtimeConfig.public.SUPABASE_URL,
      runtimeConfig.public.SUPABASE_KEY
    );
    return {
      supabase,
      activeTab: "questions",
      newQuestion: {
        question: "",
        answers: ["", "", "", ""],
        correctAnswerIndex: 1,
      },
      allQuestions: [],
      questions: [],
      selectedCategory: "",
      uniqueCategories: [],
      isModalOpen: false,
      updatedQuestion: {
        id: null,
        question: "",
        answers: ["", "", "", ""],
        correct_answer: null,
      },
      userScores: [],
      categoryAverages: {},
    };
  },
  computed: {
    uniqueCategories() {
      return [...new Set(this.allQuestions.map((q) => q.category))];
    },
    categoryAverages() {
      const averages = {};

      // Calculate total score and user count for each category
      this.userScores.forEach((score) => {
        const category = score.category;
        if (!averages[category]) {
          averages[category] = { totalScore: 0, count: 0 };
        }
        averages[category].totalScore += score.score;
        averages[category].count += 1;
      });

      // Calculate the average score and add user count
      Object.keys(averages).forEach((category) => {
        averages[category].average = (
          averages[category].totalScore / averages[category].count
        ).toFixed(2);
        averages[category].userCount = averages[category].count;
      });

      return averages;
    },
  },
  async mounted() {
    this.fetchQuestions();
    this.fetchUserScores();
  },
  methods: {
    async fetchQuestions(category = "") {
      let query = this.supabase.from("question").select();
      if (category) {
        query = query.eq("category", category);
      }

      const { data, error } = await query;

      if (error) {
        console.error("Error fetching questions:", error);
      } else {
        this.allQuestions = data;

        this.questions = data;
      }
    },
    async addQuestion() {
      try {
        const questionToInsert = {
          created_at: new Date().toISOString(),
          question: this.newQuestion.question,
          answers: this.newQuestion.answers,
          correct_answer: this.newQuestion.correctAnswerIndex,
          category: this.newQuestion.category,
        };

        const { data, error } = await this.supabase
          .from("question")
          .insert([questionToInsert])
          .select();

        if (error) {
          console.error("Error adding question:", error);
        } else {
          console.log("Question added successfully:", data);
          this.questions.push({
            ...data[0],
            answers: this.newQuestion.answers,
          });
          this.resetForm();
        }
      } catch (error) {
        console.error("Unexpected error while adding question:", error);
      }
    },
    async filterByCategory() {
      this.questions = this.selectedCategory
        ? this.allQuestions.filter((q) => q.category === this.selectedCategory)
        : this.allQuestions;
    },
    async deleteQuestion(id) {
      const { error } = await this.supabase
        .from("question")
        .delete()
        .eq("id", id);

      if (error) {
        console.error("Error deleting question:", error);
      } else {
        this.questions = this.questions.filter((q) => q.id !== id);
      }
    },
    openUpdateModal(question) {
      this.updatedQuestion = { ...question };
      this.isModalOpen = true;
    },
    closeModal() {
      this.isModalOpen = false;
    },
    async updateQuestionInDB() {
      const { error } = await this.supabase
        .from("question")
        .update(this.updatedQuestion)
        .eq("id", this.updatedQuestion.id);

      if (error) {
        console.error("Error updating question:", error);
      } else {
        this.isModalOpen = false;
        await this.fetchQuestions(this.selectedCategory);
      }
    },

    async fetchUserScores() {
      const { data, error } = await this.supabase.from("users").select("*");

      if (error) {
        console.error("Error fetching questions:", error);
      } else {
        this.userScores = data.map((score) => ({
          ...score,
          created_at: dayjs(score.created_at).format("YYYY-MM-DD HH:mm:ss"), 
        }));
      }
    },
    resetForm() {
      this.newQuestion = {
        question: "",
        answers: ["", "", "", ""],
        correctAnswerIndex: 0,
      };
      this.correctAnswerIndex = 1;
    },
  },
};
</script>

<style scoped>
/* Your existing styles */
</style>
