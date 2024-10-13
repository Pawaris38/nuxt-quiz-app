<template>
  <div class="w-full max-w-[500px] mx-auto flex flex-col gap-4 p-4">
    <div class="flex flex-col mb-4">
      <label class="text-lg font-semibold mb-2" for="userName"
        >Enter your name:</label
      >
      <input
        type="text"
        id="userName"
        v-model="userName"
        class="p-2 border border-gray-300 rounded-md focus:outline-none focus:ring focus:ring-blue-500"
        placeholder="Your Name"
      />
    </div>
    <div
      class="w-full flex flex-row h-12 items-center justify-center text-white gap-3"
    >
      <span class="text-black font-semibold text-3xl"
        >Choose a Category for Quiz</span
      >
    </div>
    <div class="grid grid-cols-2 sm:grid-cols-3 gap-4">
      <button
        v-for="(category, index) in categories"
        :key="index"
        :to="`${category}`"
        @click.prevent="loadQuizByCategory(category)"
        class="bg-white shadow-md hover:shadow-lg transition-transform transform hover:scale-105 p-6 rounded-lg flex items-center justify-center text-center relative"
      >
        <div class="font-extrabold text-lg text-gray-800">
          {{ category }}
        </div>
        <span class="absolute inset-0 rounded-lg bg-blue-600 opacity-10"></span>
      </button>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.grid {
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
}
</style>

<script>
import { createClient } from "@supabase/supabase-js";
export default {
  data() {
    const runtimeConfig = useRuntimeConfig();
    const supabase = createClient(
      runtimeConfig.public.SUPABASE_URL,
      runtimeConfig.public.SUPABASE_KEY
    );
    return {
      supabase,
      categories: [],
      userName: "",
    };
  },
  async mounted() {
    await this.fetchCategories();
  },
  methods: {
    async fetchCategories() {
      const { data, error } = await this.supabase
        .from("question")
        .select("category");

      if (error) {
        console.error("Error fetching categories:", error);
      } else {
        const uniqueCategories = new Set(data.map((item) => item.category));
        this.categories = Array.from(uniqueCategories); 
      }
    },
    loadQuizByCategory(category) {
      if (!this.userName) {
        alert("Please enter your name before selecting a category.");
      } else {
        this.$router.push({
          path: `/category/${category}`,
          query: { userName: this.userName },
        });
      }
    },
  },
};
</script>
