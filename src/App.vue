<template>
  <div class="container">
    <h1 class="title">Подбор физической тренировки</h1>

    <transition name="fade" mode="out-in" appear>
      <div v-if="showIntro" key="intro" class="card intro-card">
        <p class="intro-text">
          Программа для персонализированного подбора<br />
          вида, интенсивности и продолжительности физической тренировки<br />
          для пациентки, страдающей раком молочной железы.
        </p>
        <button @click="start" class="btn">Начать</button>
      </div>

      <div v-else-if="step < questions.length" key="question" class="card">
        <p class="progress">Вопрос {{ step + 1 }} из {{ questions.length }}</p>
        <p class="question-text">{{ questions[step].text }}</p>

        <form class="options">
          <label
              v-for="(opt, i) in questions[step].options"
              :key="i"
              class="option-label"
              :class="{ checked: answers[step] === i }"
          >
            <input
                type="radio"
                :name="'q' + step"
                :value="i"
                v-model="answers[step]"
                class="radio-input"
            />
            <span class="custom-radio"></span>
            {{ opt.label }}
          </label>
        </form>

        <button
            @click="next"
            :disabled="answers[step] === null"
            class="btn"
            :class="{ disabled: answers[step] === null }"
        >
          Далее
        </button>
      </div>

      <div v-else key="result" class="card result-card">
        <h2 class="result-title">Результат:</h2>
        <p><strong>Оптимальный режим:</strong> {{ getProgram(primary) }}</p>
        <p><strong>Альтернативный режим:</strong> {{ getProgram(secondary) }}</p>
        <button @click="reset" class="btn reset-btn">Пройти заново</button>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { questions } from './questions.js'

const showIntro = ref(true)
const step = ref(0)
const answers = ref(Array(questions.length).fill(null))
const scores = ref({ A: 0, B: 0, C: 0, D: 0 })

const start = () => {
  showIntro.value = false
}

const next = () => {
  const selected = questions[step.value].options[answers.value[step.value]].values
  scores.value.A += selected[0]
  scores.value.B += selected[1]
  scores.value.C += selected[2]
  scores.value.D += selected[3]
  step.value++
}

const reset = () => {
  showIntro.value = true
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

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');

html, body {
  height: 100%;
  width: 100%;
  margin: 0;
  padding: 0;
  background: #f9fafb;
  font-family: 'Inter', system-ui, sans-serif;
  box-sizing: border-box;
}

body {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  width: 100vw;
  overflow: auto;
  user-select: none;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  transition: background-color 0.3s ease;
}

.container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 680px;
  max-width: 95vw;
  background: #fff;
  border-radius: 16px;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.12), 0 8px 32px rgba(0, 0, 0, 0.06);
  color: #222;
  display: flex;
  flex-direction: column;
  padding: 3.5rem 3rem;
  margin: auto;
  flex-shrink: 0;
}

.title {
  text-align: center;
  font-weight: 800;
  font-size: 3.4rem;
  margin-bottom: 2.5rem;
  color: #111827;
  user-select: text;
}

.card {
  flex: 1 1 auto;
  overflow: visible;
  padding-right: 0;
  transition: all 0.5s ease;
}

.progress {
  font-weight: 700;
  color: #4b5563;
  margin-bottom: 1rem;
  font-size: 1.3rem;
}

.question-text {
  font-size: 1.8rem;
  margin-bottom: 2rem;
  line-height: 1.5;
  font-weight: 600;
  user-select: text;
}

.option-label {
  position: relative;
  display: flex;
  align-items: center;
  gap: 20px;
  font-size: 1.4rem;
  cursor: pointer;
  padding: 14px 18px;
  border-radius: 12px;
  transition: background-color 0.3s ease;
  user-select: none;
  font-weight: 500;
}

.option-label:hover {
  background-color: #f0f9ff;
}

.option-label.checked {
  background-color: #bae6fd;
  border: 2px solid #3b82f6;
}

.radio-input {
  opacity: 0;
  position: absolute;
  pointer-events: none;
}

.custom-radio {
  width: 24px;
  height: 24px;
  border: 3px solid #9ca3af;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  transition: border-color 0.3s ease;
}

.option-label.checked .custom-radio {
  border-color: #2563eb;
}

.option-label.checked .custom-radio::after {
  content: "";
  width: 12px;
  height: 12px;
  background-color: #2563eb;
  border-radius: 50%;
}

.btn {
  display: inline-block;
  padding: 1rem 2.5rem;
  background-color: #2563eb;
  color: #fff;
  font-weight: 700;
  font-size: 1.4rem;
  border: none;
  border-radius: 14px;
  cursor: pointer;
  user-select: none;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
  box-shadow: 0 6px 16px rgba(37, 99, 235, 0.4);
  margin-top: 1rem;
  align-self: flex-start;
}

.btn:hover:not(.disabled) {
  background-color: #1d4ed8;
  box-shadow: 0 8px 20px rgba(29, 78, 216, 0.5);
}

.btn.disabled,
.btn:disabled {
  cursor: not-allowed;
  background-color: #93c5fd;
  box-shadow: none;
  color: #dbeafe;
}

.reset-btn {
  margin-top: 3.5rem;
  align-self: center;
}

.result-card {
  text-align: center;
  font-size: 1.6rem;
}

.result-title {
  font-size: 2rem;
  font-weight: 700;
  margin-bottom: 1.5rem;
  color: #111827;
  user-select: text;
}

/* Интро */
.intro-card {
  text-align: center;
  font-size: 1.6rem;
}

.intro-text {
  font-size: 1.6rem;
  line-height: 1.6;
  font-weight: 500;
  color: #374151;
  margin-bottom: 0.8rem;
  user-select: text;
}

.start-btn {
  align-self: center;
  padding: 1.2rem 3rem;
  font-size: 1.5rem;
  background-color: #10b981;
  box-shadow: 0 6px 16px rgba(16, 185, 129, 0.4);
}

.start-btn:hover {
  background-color: #059669;
  box-shadow: 0 8px 20px rgba(5, 150, 105, 0.5);
}

/* Анимация */
.fade-enter-active,
.fade-leave-active {
  transition: all 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(20px);
}

.fade-enter-to,
.fade-leave-from {
  opacity: 1;
  transform: translateY(0);
}

/* Адаптив */
@media (max-width: 720px), (max-height: 650px) {
  .container {
    width: 90vw;
    padding: 2.5rem 2rem;
  }

  .title {
    font-size: 2.4rem;
    margin-bottom: 1.8rem;
  }

  .progress {
    font-size: 1.1rem;
  }

  .question-text {
    font-size: 1.4rem;
    margin-bottom: 1.5rem;
  }

  .option-label {
    font-size: 1.2rem;
    padding: 10px 14px;
    gap: 14px;
  }

  .custom-radio {
    width: 20px;
    height: 20px;
    border-width: 2.5px;
  }

  .option-label.checked .custom-radio::after {
    width: 10px;
    height: 10px;
  }

  .btn {
    font-size: 1.1rem;
    padding: 0.75rem 2rem;
    margin-top: 0.8rem;
  }

  .reset-btn {
    margin-top: 2rem;
  }

  .result-title {
    font-size: 1.6rem;
    margin-bottom: 1rem;
  }
}

* {
  transition: all 0.3s ease;
}
</style>