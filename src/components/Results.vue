<script setup>
import { computed } from "vue";

const { userAnswers } = defineProps(["userAnswers"]);
const emit = defineEmits(["restartQuiz"]);

const correctAnswerCount = computed(() => {
  return userAnswers.filter(
    (answer) => answer.userAnswer === answer.question.correct_answer
  ).length;
});
</script>

<template>
  <section class="result-screen container">
    <h1 v-if="correctAnswerCount === userAnswers.length">
      &#127881; Congratulations!
    </h1>
    <h1>
      You have answered {{ correctAnswerCount }} out of
      {{ userAnswers.length }} questions correctly.
    </h1>
    <ul>
      <li
        v-for="(answer, index) in userAnswers"
        :key="index"
        :class="
          answer.question.correct_answer === answer.userAnswer
            ? 'correct'
            : 'incorrect'
        "
      >
        <p
          v-html="
            answer.question.correct_answer === answer.userAnswer
              ? '&#x2714;'
              : '&#x2716;'
          "
        ></p>
        <b>
          {{ answer.question.question }}
        </b>
        <p>Your answer: {{ answer.userAnswer }}</p>
        <p>Correct answer: {{ answer.question.correct_answer }}</p>
      </li>
    </ul>

    <button @click="$emit('restartQuiz')">Create a new Quiz</button>
  </section>
</template>
