<script setup>
import { computed, ref } from "vue";

const { questions } = defineProps(["questions"]);
const emit = defineEmits(["storeAnswer", "endQuiz"]);
const currentQuestion = ref(0);
const selectedOption = ref(null);

const shuffledOptions = computed(() => {
  let options = [...questions.results[currentQuestion.value].incorrect_answers];
  options.splice(
    Math.round(Math.random() * options.length),
    0,
    questions.results[currentQuestion.value].correct_answer
  );

  return options;
});

const submitAnswer = () => {
  emit("storeAnswer", {
    question: questions.results[currentQuestion.value],
    userAnswer: selectedOption.value,
  });
  selectedOption.value = null;

  if (currentQuestion.value === questions.results.length - 1) {
    currentQuestion.value = 0;
    emit("endQuiz");
  } else {
    currentQuestion.value++;
  }
};

console.log(questions.results);
</script>

<template>
  <section class="container quiz">
    <div class="header">
      <h2>Quiz About {{ questions.results[0].category }}</h2>
      <p>
        Question {{ currentQuestion + 1 }} of {{ questions.results.length }}
      </p>
    </div>
    <progress
      max="100"
      :value="((currentQuestion + 1) / questions.results.length) * 100"
    ></progress>

    <div class="questions">
      <h3>
        {{ questions.results[currentQuestion].question }}
      </h3>
    </div>

    <div class="answers">
      <button
        class="answer"
        :class="{ active: answer === selectedOption }"
        v-for="answer in shuffledOptions"
        :key="answer"
        @click="selectedOption = answer"
      >
        {{ answer }}
      </button>
    </div>

    <button @click="submitAnswer" :disabled="!selectedOption">Send</button>
  </section>
</template>
