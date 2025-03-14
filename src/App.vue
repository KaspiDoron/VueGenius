<script setup>
import { reactive, ref } from "vue";
import StartScreen from "./components/StartScreen.vue";
import { GoogleGenerativeAI, SchemaType } from "@google/generative-ai";
import Quiz from "./components/Quiz.vue";
import Loader from "./components/Loader.vue";
import Results from "./components/Results.vue";
const apiKey = import.meta.env.VITE_GEMINI_API_KEY;

const questions = ref("");
const status = ref("start");
const userAnswers = ref([]);
const errorMessage = ref("");

const restartQuiz = () => {
  questions.value = "";
  status.value = "start";
  userAnswers.value = [];
  errorMessage.value = "";
};

const storeAnswer = (answer) => {
  userAnswers.value.push(answer);
};

const startQuiz = async (topic) => {
  status.value = "loading";
  const genAI = new GoogleGenerativeAI(apiKey);

  const schema = {
    description: "Quiz questions schema",
    type: SchemaType.OBJECT,
    properties: {
      response_code: {
        type: SchemaType.NUMBER,
        description: "Response code indicating the status of the request",
        nullable: false,
      },
      results: {
        type: SchemaType.ARRAY,
        description: "List of quiz questions",
        items: {
          type: SchemaType.OBJECT,
          properties: {
            type: {
              type: SchemaType.STRING,
              description:
                "Question type (e.g., multiple choice or true/false)",
              nullable: false,
            },
            difficulty: {
              type: SchemaType.STRING,
              description: "Difficulty level (easy, medium, hard)",
              nullable: false,
            },
            category: {
              type: SchemaType.STRING,
              description: "Category of the question",
              nullable: false,
            },
            question: {
              type: SchemaType.STRING,
              description: "The quiz question text",
              nullable: false,
            },
            correct_answer: {
              type: SchemaType.STRING,
              description: "The correct answer to the question",
              nullable: false,
            },
            incorrect_answers: {
              type: SchemaType.ARRAY,
              description: "List of incorrect answers",
              items: {
                type: SchemaType.STRING,
              },
              nullable: false,
            },
          },
          required: [
            "type",
            "difficulty",
            "category",
            "question",
            "correct_answer",
            "incorrect_answers",
          ],
        },
      },
    },
    required: ["response_code", "results"],
  };

  const model = genAI.getGenerativeModel({
    model: "gemini-1.5-pro",
    generationConfig: {
      responseMimeType: "application/json",
      responseSchema: schema,
    },
  });

  try {
    const result = await model.generateContent(
      `
    Create 5 quiz questions about ${topic}
    Difficulty: Easy to Medium
    Type: Multiple Choice
    `
    );

    questions.value = JSON.parse(result.response.text());
    status.value = "ready";
  } catch (error) {
    errorMessage.value = error.message;
    status.value = "start";
  }
};
</script>

<template>
  <div id="app">
    <header>
      <div class="container">
        <img class="logo" src="./assets/logo.png" alt="App Logo" />
        <h1>VueGenius Quiz</h1>
      </div>
    </header>

    <StartScreen
      v-if="status === 'start'"
      @startQuiz="startQuiz"
      :errorMessage="errorMessage"
    />
    <Loader v-if="status === 'loading'" />
    <Quiz
      v-if="status === 'ready'"
      @storeAnswer="storeAnswer"
      @endQuiz="status = 'finished'"
      :questions="questions"
    />
    <Results
      v-if="status === 'finished'"
      :userAnswers="userAnswers"
      @restartQuiz="restartQuiz"
    />
  </div>
</template>

<style scoped></style>
