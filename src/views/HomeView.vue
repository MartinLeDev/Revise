<script setup lang="ts">
import { ref, onMounted } from 'vue'
import type { Ref } from 'vue'

interface Question {
  name: string
  response: string
  fake_reponse: string[]
}

interface Chapter {
  name: string
  id: string
  questions: Question[]
}

interface Matiere {
  id: string
  name: string
  chapters: Chapter[]
}

const matieres: Ref<Matiere[]> = ref([])
const selectedMatiere: Ref<string | null> = ref(null)
const selectedChapter: Ref<string | null> = ref(null)
const questions: Ref<Question[]> = ref([])
const currentQuestionIndex: Ref<number> = ref(0)
const selectedAnswer: Ref<string | null> = ref(null)
const showResult: Ref<boolean> = ref(false)
const isCorrect: Ref<boolean> = ref(false)
const showQuiz: Ref<boolean> = ref(false)
const currentMatiere: Ref<Matiere | null> = ref(null)
const currentChapter: Ref<Chapter | null> = ref(null)
const currentShuffledAnswers: Ref<string[]> = ref([])
const currentQuestion: Ref<Question | null> = ref(null)
const currentQuestionChapter: Ref<Chapter | null> = ref(null)
const quizFinished = ref(false)

const shuffleQuestions = (array: Question[]) => {
  const shuffled = [...array]
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
  }
  return shuffled
}

const shuffleArray = <T>(array: T[]): T[] => {
  const shuffled = [...array]
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
  }
  return shuffled
}

onMounted(async () => {
  const response = await fetch('/example.json')
  const data = await response.json()
  matieres.value = data.matieres
})

const startQuiz = () => {
  let selectedQuestions: Question[] = []

  if (selectedMatiere.value === 'random') {
    currentMatiere.value = null
    currentChapter.value = null
    // Sélectionner des questions de toutes les matières
    matieres.value.forEach(matiere => {
      matiere.chapters.forEach(chapter => {
        selectedQuestions.push(...chapter.questions)
      })
    })
  } else {
    const matiere = matieres.value.find(m => m.id === selectedMatiere.value)
    if (!matiere) return
    currentMatiere.value = matiere

    if (selectedChapter.value === 'random') {
      currentChapter.value = null
      // Sélectionner des questions de tous les chapitres de la matière
      matiere.chapters.forEach(chapter => {
        selectedQuestions.push(...chapter.questions)
      })
    } else {
      // Sélectionner les questions du chapitre spécifique
      const chapter = matiere.chapters.find(c => c.name === selectedChapter.value)
      if (chapter) {
        currentChapter.value = chapter
        selectedQuestions = [...chapter.questions]
      }
    }
  }

  questions.value = shuffleArray(selectedQuestions)
  currentQuestionIndex.value = 0
  showQuiz.value = true
  selectedAnswer.value = null
  showResult.value = false
  shuffleCurrentAnswers()
  currentQuestionChapter.value = getCurrentQuestionChapter()
  quizFinished.value = false
}

const getCurrentQuestion = () => questions.value[currentQuestionIndex.value]

const shuffleCurrentAnswers = () => {
  const currentQuestion = getCurrentQuestion()
  if (!currentQuestion) {
    currentShuffledAnswers.value = []
    return
  }
  const answers = [...currentQuestion.fake_reponse, currentQuestion.response]
  currentShuffledAnswers.value = shuffleArray(answers)
}

const getShuffledAnswers = () => {
  const currentQuestion = getCurrentQuestion()
  if (!currentQuestion) return []

  const answers = [...currentQuestion.fake_reponse, currentQuestion.response]
  return answers.sort(() => Math.random() - 0.5)
}

const checkAnswer = (answer: string) => {
  selectedAnswer.value = answer
  isCorrect.value = answer === getCurrentQuestion().response
  showResult.value = true
  // Vérifie si c'est la dernière question
  if (currentQuestionIndex.value === questions.value.length - 1) {
    quizFinished.value = true
  }
}

const getCurrentQuestionChapter = () => {
  const question = getCurrentQuestion()
  if (!question) return null

  for (const matiere of matieres.value) {
    for (const chapter of matiere.chapters) {
      if (chapter.questions.some(q => q.name === question.name)) {
        return chapter
      }
    }
  }
  return null
}

const nextQuestion = () => {
  if (currentQuestionIndex.value >= questions.value.length - 1) {
    // Fin du quiz
    quizFinished.value = true
    return
  } else {
    currentQuestionIndex.value++
  }
  selectedAnswer.value = null
  showResult.value = false
  shuffleCurrentAnswers()
  currentQuestionChapter.value = getCurrentQuestionChapter()
}

const resetQuiz = () => {
  showQuiz.value = false
  selectedMatiere.value = null
  selectedChapter.value = null
  currentMatiere.value = null
  currentChapter.value = null
  questions.value = []
  currentQuestionIndex.value = 0
  selectedAnswer.value = null
  showResult.value = false
  currentQuestionChapter.value = null
  quizFinished.value = false
}

// Ajout d'une fonction utilitaire pour détecter le mobile
function isMobile() {
  return window.innerWidth <= 600;
}
</script>

<template>
  <div class="quiz-container">
    <h1>Quiz infini</h1>
    <div v-if="!showQuiz" class="selection-container">
      <div class="selection-group">
        <label>Choisissez une matière:</label>
        <select v-model="selectedMatiere">
          <option value="random">Aléatoire</option>
          <option v-for="matiere in matieres" :key="matiere.id" :value="matiere.id">
            {{ matiere.name }}
          </option>
        </select>
      </div>
      <div class="selection-group" v-if="selectedMatiere && selectedMatiere !== 'random'">
        <label>Choisissez un chapitre:</label>
        <select v-model="selectedChapter">
          <option value="random">Aléatoire</option>
          <option
            v-for="chapter in matieres.find(m => m.id === selectedMatiere)?.chapters"
            :key="chapter.name"
            :value="chapter.name"
          >
            {{ chapter.name }}
          </option>
        </select>
      </div>
      <button @click="startQuiz" class="start-btn" :disabled="!selectedMatiere">
        Commencer le quiz
      </button>
    </div>
    <div v-else>
      <button
        v-if="showQuiz"
        class="home-btn top-left"
        @click="resetQuiz"
        :class="{ 'mobile-bottom': isMobile() }"
        style="left: 50%; transform: translateX(-50%); right: auto;"
      >
        Retour à l'accueil
      </button>
      <div v-if="questions.length > 0" class="question-card">
        <div class="quiz-header">
          <h2 v-if="currentMatiere">
            {{ currentMatiere.name }}
            <span v-if="currentChapter"> - {{ currentChapter.name }}</span>
          </h2>
          <h2 v-else>
            Toutes les matières
            <span v-if="currentQuestionChapter"> - {{ currentQuestionChapter.name }}</span>
          </h2>
          <p class="question-count">Question {{ currentQuestionIndex + 1 }}/{{ questions.length }}</p>
        </div>

        <p class="question">{{ getCurrentQuestion().name }}</p>

        <div class="answers">
          <button
            v-for="answer in currentShuffledAnswers"
            :key="answer"
            :class="[
              'answer-btn',
              {
                'selected': selectedAnswer === answer,
                'correct': showResult && answer === getCurrentQuestion().response,
                'incorrect': showResult && selectedAnswer === answer && answer !== getCurrentQuestion().response
              }
            ]"
            @click="checkAnswer(answer)"
            :disabled="showResult || quizFinished"
          >
            {{ answer }}
          </button>
        </div>

        <div class="progress-bar-container">
          <div class="progress-bar" :style="{ width: ((currentQuestionIndex + (quizFinished ? 1 : 0)) / questions.length * 100) + '%' }"></div>
        </div>
        <div v-if="showResult" class="result">
          <p :class="{ 'correct-text': isCorrect, 'incorrect-text': !isCorrect }">
            {{ isCorrect ? 'Correct !' : 'Incorrect !' }}
          </p>
          <div class="button-group">
            <button v-if="!quizFinished" @click="nextQuestion" class="action-btn next-btn">
              Question suivante
            </button>
            <button v-if="quizFinished" @click="startQuiz" class="action-btn restart-btn">
              Recommencer
            </button>
            <button v-if="quizFinished" @click="resetQuiz" class="action-btn home-btn">
              Retour à l'accueil
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.selection-container {
  background: var(--slate2);
  border-radius: 16px;
  padding: 24px;
  margin-top: 20px;
  box-shadow: 12px 12px 24px var(--slate4),
              -12px -12px 24px var(--slate1);
}

.selection-group {
  margin-bottom: 20px;
}

.selection-group label {
  display: block;
  margin-bottom: 8px;
  color: var(--slate12);
  font-weight: 500;
}

.selection-group select {
  width: 100%;
  padding: 12px;
  border: 2px solid var(--slate6);
  border-radius: 8px;
  background: var(--slate1);
  color: var(--slate12);
  font-family: 'Poppins', sans-serif;
  font-size: 1em;
}

.start-btn {
  width: 100%;
  padding: 14px;
  margin-top: 20px;
  background: var(--slate2);
  border: none;
  border-radius: 8px;
  color: var(--slate12);
  font-family: 'Poppins', sans-serif;
  font-weight: 600;
  font-size: 1.1em;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 8px 8px 16px var(--slate4),
              -8px -8px 16px var(--slate1);
}

.start-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 12px 12px 24px var(--slate4),
              -12px -12px 24px var(--slate1);
}

.start-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.quiz-header {
  margin-bottom: 16px;
}

.question-count {
  font-size: 0.9em;
  color: var(--slate11);
  margin-top: 4px;
}

.button-group {
  display: flex;
  gap: 10px;
  margin-top: 20px;
  flex-wrap: wrap;
  justify-content: center;
}

.action-btn {
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  font-family: 'Poppins', sans-serif;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.next-btn {
  background: var(--slate3);
  color: var(--slate12);
}

.restart-btn {
  background: var(--slate4);
  color: var(--slate12);
}

.home-btn {
  background: var(--slate5);
  color: var(--slate12);
}

.home-btn.top-left {
  position: absolute;
  top: 24px;
  left: 24px;
  z-index: 10;
  background: var(--slate5);
  color: var(--slate12);
  border-radius: 8px;
  padding: 10px 18px;
  font-size: 1em;
  font-family: 'Poppins', sans-serif;
  font-weight: 600;
  border: none;
  cursor: pointer;
  box-shadow: 4px 4px 12px var(--slate4), -4px -4px 12px var(--slate1);
}

.home-btn.mobile-bottom {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  top: auto;
  width: 100vw;
  border-radius: 0;
  box-shadow: 0 -2px 12px var(--slate4);
  padding: 18px 0;
  font-size: 1.1em;
  text-align: center;
}

.progress-bar-container {
  width: 100%;
  height: 12px;
  background: var(--slate3);
  border-radius: 8px;
  overflow: hidden;
}

.progress-bar {
  height: 100%;
  background: var(--blue9);
  transition: width 0.4s cubic-bezier(.4,2,.6,1);
}

@media (max-width: 600px) {
  .home-btn.top-left {
    display: none;
  }
  .home-btn.mobile-bottom {
    display: block;
  }
}
</style>
