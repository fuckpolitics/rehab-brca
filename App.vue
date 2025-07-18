<!-- Vue 3 Single File Component (App.vue) -->
<template>
  <div class="max-w-2xl mx-auto p-4 text-gray-800">
    <h1 class="text-2xl font-bold mb-4">Подбор физической тренировки</h1>

    <div v-if="step < questions.length" class="mb-6">
      <p class="font-medium mb-2">Вопрос {{ step + 1 }} из {{ questions.length }}</p>
      <p class="mb-4">{{ questions[step].text }}</p>

      <div v-for="(opt, i) in questions[step].options" :key="i" class="mb-2">
        <label class="flex items-center gap-2">
          <input type="radio" :name="'q' + step" :value="i" v-model="answers[step]" />
          {{ opt.label }}
        </label>
      </div>

      <button @click="next" :disabled="answers[step] === null" class="mt-4 px-4 py-2 bg-blue-600 text-white rounded">
        Далее
      </button>
    </div>

    <div v-else>
      <h2 class="text-xl font-bold mb-2">Результат:</h2>
      <p class="mb-4"><strong>Оптимальный режим:</strong> {{ getProgram(primary) }}</p>
      <p><strong>Альтернативный режим:</strong> {{ getProgram(secondary) }}</p>
      <button @click="reset" class="mt-6 px-4 py-2 bg-gray-600 text-white rounded">Пройти заново</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { questions } from './questions.js'

const step = ref(0)
const answers = ref(Array(questions.length).fill(null))
const scores = ref({ A: 0, B: 0, C: 0, D: 0 })

const next = () => {
  const selected = questions[step.value].options[answers.value[step.value]].values
  scores.value.A += selected[0]
  scores.value.B += selected[1]
  scores.value.C += selected[2]
  scores.value.D += selected[3]
  step.value++
}

const reset = () => {
  step.value = 0
  answers.value = Array(questions.length).fill(null)
  scores.value = { A: 0, B: 0, C: 0, D: 0 }
}

const sortedKeys = computed(() => {
  return Object.entries(scores.value)
      .sort(([, a], [, b]) => b - a)
      .map(([key]) => key)
})

const primary = computed(() => sortedKeys.value[0])
const secondary = computed(() => sortedKeys.value[1])

function getProgram(letter) {
  const map = {
    A: 'Аэробная тренировка умеренной интенсивности, 3× в неделю по 30+ мин. Всего 150 мин в неделю.',
    B: 'Комбинированные тренировки: аэробные (2× в неделю, 80–90 мин) + силовые (2× в неделю, 60–70 мин).',
    C: 'Силовые тренировки/сопротивление, 2× в неделю, 2 подхода по 8–15 повторов, умеренная/низкая интенсивность.',
    D: 'Высокоинтенсивные интервальные тренировки 3× в неделю, 20–30 мин с разминкой и заминкой.'
  }
  return map[letter]
}
</script>

<style>
body {
  font-family: system-ui, sans-serif;
  background-color: #f9fafb;
}
</style>
