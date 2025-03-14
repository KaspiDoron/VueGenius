<script setup>
import StartScreen from "./components/StartScreen.vue";
import { GoogleGenerativeAI } from "@google/generative-ai";
const apiKey = import.meta.env.VITE_GEMINI_API_KEY;

const startQuiz = async (topic) => {
  const genAI = new GoogleGenerativeAI(GEMINI_API_KEY);

  const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });

  const prompt = "Explain how AI works";

  const result = await model.generateContent(prompt);
  console.log(result.response.text());
};
</script>

<template>
  <div id="app">
    <header>
      <div class="container">
        <img class="logo" src="./assets/logo.png" alt="App Logo" />
        <h1>VueGenius</h1>
      </div>
    </header>
  </div>

  <StartScreen @startQuiz="startQuiz" />
</template>

<style scoped></style>
