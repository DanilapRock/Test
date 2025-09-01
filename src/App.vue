<template>
  <div class="min-h-screen bg-neutral-50 text-neutral-900 antialiased p-6">
    <div class="max-w-2xl mx-auto space-y-8">
      <header class="flex items-center justify-between">
        <h1 class="text-3xl font-semibold tracking-tight">Дневник тренировок</h1>
      </header>

    <!-- Текущая тренировка -->
    <section class="space-y-4">
      <div class="flex items-center justify-between">
        <h2 class="text-xl font-semibold tracking-tight">Текущая тренировка</h2>
        <button
          v-if="!currentWorkout"
          @click="startWorkout"
          class="inline-flex items-center rounded-lg bg-black px-4 py-2 text-white hover:bg-neutral-800 active:bg-neutral-900 transition"
        >
          Начать тренировку
        </button>
      </div>

      <div v-if="currentWorkout" class="rounded-xl border border-neutral-200 bg-white/70 backdrop-blur-sm shadow-sm p-5">
        <div class="flex items-center justify-between mb-5">
          <div>
            <p class="text-sm text-neutral-600">Дата: {{ currentWorkout.date }}</p>
            <p class="text-sm text-neutral-600">Подходов: {{ currentWorkout.sets.length }}</p>
          </div>
          <div class="space-x-2">
            <button
              @click="clearCurrent"
              class="inline-flex items-center rounded-lg border border-neutral-300 px-3 py-2 text-neutral-700 hover:bg-neutral-50 transition"
            >
              Очистить
            </button>
            <button
              @click="endWorkout"
              class="inline-flex items-center rounded-lg bg-red-600 px-4 py-2 text-white hover:bg-red-700 active:bg-red-800 transition"
            >
              Завершить
            </button>
          </div>
        </div>

        <!-- Добавить подход -->
        <form @submit.prevent="addSet" class="grid grid-cols-1 md:grid-cols-4 gap-3 mb-5">
          <select v-model="newSet.exerciseId" required class="border border-neutral-300 rounded-lg px-3 py-2 bg-white focus:outline-none focus:ring-2 focus:ring-black/10 focus:border-black/30">
            <option disabled value="">Выберите упражнение</option>
            <option v-for="ex in exerciseDatabase" :key="ex.id" :value="ex.id">{{ ex.name }}</option>
          </select>
          <input v-model.number="newSet.reps" type="number" min="1" placeholder="Повторения" required class="border border-neutral-300 rounded-lg px-3 py-2 bg-white placeholder:text-neutral-400 focus:outline-none focus:ring-2 focus:ring-black/10 focus:border-black/30" />
          <input v-model.number="newSet.weight" type="number" min="0" step="0.5" placeholder="Вес (кг)" required class="border border-neutral-300 rounded-lg px-3 py-2 bg-white placeholder:text-neutral-400 focus:outline-none focus:ring-2 focus:ring-black/10 focus:border-black/30" />
          <button type="submit" class="inline-flex items-center justify-center rounded-lg bg-black px-4 py-2 text-white hover:bg-neutral-800 active:bg-neutral-900 transition">Добавить подход</button>
        </form>

        <!-- Таблица подходов -->
        <div v-if="currentWorkout.sets.length" class="overflow-hidden rounded-lg border border-neutral-200">
          <table class="w-full text-left text-sm">
            <thead>
              <tr class="text-neutral-600 border-b bg-neutral-50">
                <th class="py-2 px-3">#</th>
                <th class="py-2 px-3">Упражнение</th>
                <th class="py-2 px-3">Повторения</th>
                <th class="py-2 px-3">Вес (кг)</th>
                <th class="py-2 px-3 text-right"></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(s, i) in currentWorkout.sets" :key="s.id" class="border-b">
                <td class="py-2 px-3">{{ i + 1 }}</td>
                <td class="py-2 px-3">{{ exerciseName(s.exerciseId) }}</td>
                <td class="py-2 px-3">{{ s.reps }}</td>
                <td class="py-2 px-3">{{ s.weight }}</td>
                <td class="py-2 px-3 text-right">
                  <button @click="removeSet(i)" class="text-red-600 hover:text-red-700 hover:underline">Удалить</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </section>

    <!-- База упражнений -->
    <section class="space-y-3">
      <h2 class="text-xl font-semibold tracking-tight">Упражнения</h2>
      <div class="rounded-xl border border-neutral-200 bg-white/70 backdrop-blur-sm shadow-sm p-5">
        <form @submit.prevent="addExercise" class="flex gap-3 mb-4">
          <input v-model="newExercise" placeholder="Новое упражнение" class="border border-neutral-300 rounded-lg px-3 py-2 flex-1 bg-white placeholder:text-neutral-400 focus:outline-none focus:ring-2 focus:ring-black/10 focus:border-black/30" />
          <button type="submit" class="inline-flex items-center rounded-lg bg-black px-4 py-2 text-white hover:bg-neutral-800 active:bg-neutral-900 transition">Добавить</button>
        </form>
        <div v-if="exerciseDatabase.length" class="flex flex-wrap gap-2">
          <span v-for="ex in exerciseDatabase" :key="ex.id" class="px-2 py-1 rounded-full border border-neutral-300 text-sm bg-white text-neutral-800">
            {{ ex.name }}
          </span>
        </div>
        <p v-else class="text-sm text-neutral-500">Пока нет упражнений</p>
      </div>
    </section>

    <!-- История тренировок -->
    <section class="space-y-3">
      <h2 class="text-xl font-semibold tracking-tight">История</h2>
      <div v-if="allWorkouts.length" class="space-y-3">
        <div v-for="w in allWorkouts" :key="w.id" class="rounded-xl border border-neutral-200 bg-white/70 backdrop-blur-sm shadow-sm p-4">
          <div class="flex items-center justify-between">
            <div>
              <p class="font-medium">{{ w.date }}</p>
              <p class="text-sm text-neutral-600">Подходов: {{ w.sets.length }}</p>
            </div>
            <button @click="deleteWorkout(w.id)" class="text-red-600 hover:text-red-700 hover:underline">Удалить</button>
          </div>
          <details class="mt-2">
            <summary class="cursor-pointer text-sm text-neutral-600">Показать детали</summary>
            <ul class="mt-2 list-disc pl-5 text-sm text-neutral-800">
              <li v-for="(s, i) in w.sets" :key="s.id">
                {{ i + 1 }}. {{ s.reps }}× {{ exerciseName(s.exerciseId) }} — {{ s.weight }} кг
              </li>
            </ul>
          </details>
        </div>
      </div>
      <p v-else class="text-sm text-neutral-500">История пуста</p>
    </section>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      exerciseDatabase: JSON.parse(localStorage.getItem('exerciseDatabase')) || [
        { id: 1, name: 'Жим лежа' },
        { id: 2, name: 'Приседания' },
        { id: 3, name: 'Тяга' },
      ],
      allWorkouts: JSON.parse(localStorage.getItem('allWorkouts')) || [],
      currentWorkout: JSON.parse(localStorage.getItem('currentWorkout')) || null,
      newExercise: '',
      newSet: { exerciseId: '', reps: null, weight: null },
      tg: null,
    }
  },
  mounted() {
    const tg = window?.Telegram?.WebApp
    if (tg) {
      this.tg = tg
      try {
        tg.ready()
        tg.expand()
        tg.MainButton.onClick(this.onMainClick)
        this.updateMainButton()
      } catch (e) {
        console.warn('Telegram WebApp init failed:', e)
      }
    }
  },
  beforeUnmount() {
    if (this.tg) {
      try { this.tg.MainButton.offClick(this.onMainClick) } catch {}
    }
  },
  watch: {
    exerciseDatabase: {
      deep: true,
      handler(v) { localStorage.setItem('exerciseDatabase', JSON.stringify(v)) }
    },
    allWorkouts: {
      deep: true,
      handler(v) { localStorage.setItem('allWorkouts', JSON.stringify(v)) }
    },
    currentWorkout: {
      deep: true,
      handler(v) {
        localStorage.setItem('currentWorkout', JSON.stringify(v))
        this.updateMainButton()
      }
    },
  },
  methods: {
    exerciseName(id) {
      const ex = this.exerciseDatabase.find(e => e.id === id)
      return ex ? ex.name : '—'
    },
    addExercise() {
      const name = this.newExercise?.trim()
      if (!name) return
      this.exerciseDatabase.push({ id: Date.now(), name })
      this.newExercise = ''
    },
    startWorkout() {
      this.currentWorkout = { id: Date.now(), date: new Date().toLocaleDateString(), sets: [] }
    },
    clearCurrent() {
      if (this.currentWorkout) this.currentWorkout.sets = []
    },
    addSet() {
      const { exerciseId, reps, weight } = this.newSet
      if (!exerciseId || !reps || weight === null || weight === undefined) return
      this.currentWorkout.sets.push({ id: Date.now(), exerciseId, reps, weight })
      this.newSet = { exerciseId: '', reps: null, weight: null }
    },
    removeSet(index) {
      this.currentWorkout.sets.splice(index, 1)
    },
    endWorkout() {
      if (!this.currentWorkout || !this.currentWorkout.sets.length) return null
      const finished = this.currentWorkout
      this.allWorkouts.unshift(finished)
      this.currentWorkout = null
      return finished
    },
    deleteWorkout(id) {
      this.allWorkouts = this.allWorkouts.filter(w => w.id !== id)
    },
    updateMainButton() {
      if (!this.tg) return
      const mb = this.tg.MainButton
      if (this.currentWorkout && this.currentWorkout.sets?.length > 0) {
        mb.setText('Завершить тренировку')
        mb.show()
        mb.enable()
      } else {
        mb.hide()
      }
    },
    onMainClick() {
      const finished = this.endWorkout()
      if (finished && this.tg) {
        try {
          this.tg.sendData(JSON.stringify({ type: 'workout_completed', workout: finished }))
        } catch (e) {
          console.warn('sendData failed', e)
        }
      }
      this.updateMainButton()
    },
  },
}
</script>

<style scoped>
/* Дополнительные мелкие правки при желании */
</style>
