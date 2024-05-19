<template>
  <div>
    <div v-if="questions.length">
      <div v-for="(question, index) in questions" :key="index" class="question">
        <p>{{ question.question }}</p>
        <div v-for="(answer, i) in question.answers" :key="i">
          <input
            type="radio"
            :id="'answer-' + index + '-' + i"
            :name="'question-' + index"
            :value="answer.answer"
            v-model="userAnswers[index]"
            :disabled="isSubmitted"
          />
          <label :for="'answer-' + index + '-' + i" :class="getAnswerClass(index, answer.answer)">
            {{ answer.answer }}
          </label>
        </div>
      </div>
      <button @click="submitAnswers" :disabled="isSubmitted">Submit</button>
    </div>
    <div v-if="results !== null">
      <p>Ваш результат: {{ results }} з {{ questions.length }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'

const questions = ref([])
const userAnswers = reactive([])
const results = ref(null)
const isSubmitted = ref(false)

const fetchQuestions = async () => {
  const response = await fetch('/testData.json')
  const data = await response.json()
  questions.value = data.questions
  userAnswers.length = questions.value.length
  userAnswers.fill(null)
}

onMounted(fetchQuestions)

const submitAnswers = () => {
  let score = 0
  questions.value.forEach((question, index) => {
    const correctAnswer = question.answers.find((ans) => ans.isCorrect).answer
    if (userAnswers[index] === correctAnswer) {
      score++
    }
  })
  results.value = score
  isSubmitted.value = true
}

const getAnswerClass = (questionIndex, answer) => {
  if (!isSubmitted.value) return ''
  const question = questions.value[questionIndex]
  const correctAnswer = question.answers.find((ans) => ans.isCorrect).answer
  if (answer === correctAnswer) {
    return 'correct'
  }
  if (answer === userAnswers[questionIndex]) {
    return 'incorrect'
  }
  return ''
}
</script>

<style scoped>
.question {
  margin-bottom: 20px;
}
.correct {
  color: green;
}
.incorrect {
  color: red;
}
</style>
