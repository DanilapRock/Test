<template>
  <div class="min-h-screen bg-neutral-50 text-neutral-900 antialiased p-6">
    <div class="max-w-5xl mx-auto space-y-8">
      <header class="flex items-center justify-between">
        <h1 class="text-3xl font-semibold tracking-tight">Дневник тренировок</h1>
      </header>

    <!-- Текущая тренировка -->
    <!-- DASHBOARD -->
    <section class="grid gap-6 md:grid-cols-3">
      <div class="rounded-xl border border-neutral-200 bg-white/70 shadow-sm p-5">
        <h3 class="font-semibold mb-3">Эта неделя</h3>
        <ul class="text-sm text-neutral-700 space-y-1">
          <li>Тренировок: <strong>{{ weekly.workouts }}</strong></li>
          <li>Подходов: <strong>{{ weekly.sets }}</strong></li>
          <li>Общий тоннаж: <strong>{{ weekly.volume }} кг</strong></li>
        </ul>
      </div>
      <div class="rounded-xl border border-neutral-200 bg-white/70 shadow-sm p-5">
        <h3 class="font-semibold mb-3">Следующая тренировка</h3>
        <div v-if="nextPlanned" class="text-sm">
          <p class="text-neutral-700">{{ formatDate(nextPlanned.date) }} — {{ nextPlanned.name || 'План тренировки' }}</p>
          <button @click="startFromPlan(nextPlanned.id)" class="mt-3 inline-flex items-center rounded-lg bg-black px-3 py-2 text-white hover:bg-neutral-800">Запустить</button>
        </div>
        <p v-else class="text-sm text-neutral-500">Нет запланированных — создайте ниже</p>
      </div>
      <div class="rounded-xl border border-neutral-200 bg-white/70 shadow-sm p-5">
        <h3 class="font-semibold mb-3">Мотивация</h3>
        <p class="text-sm text-neutral-700">“{{ currentQuote }}”</p>
        <button @click="shuffleQuote" class="mt-3 text-xs text-neutral-600 underline">Обновить</button>
      </div>
    </section>

    <!-- RUNNER -->
    <section class="space-y-4">
      <div class="flex items-center justify-between">
        <h2 class="text-xl font-semibold tracking-tight">Текущая тренировка</h2>
        <div class="space-x-2" v-if="!currentWorkout">
          <button @click="startWorkout" class="inline-flex items-center rounded-lg bg-black px-4 py-2 text-white hover:bg-neutral-800">Начать с нуля</button>
          <button @click="openPlanPicker=true" class="inline-flex items-center rounded-lg border border-neutral-300 px-4 py-2 text-neutral-800 hover:bg-neutral-50">Запустить из плана</button>
        </div>
      </div>

      <div v-if="currentWorkout" class="rounded-xl border border-neutral-200 bg-white/70 backdrop-blur-sm shadow-sm p-5">
        <div class="flex items-center justify-between mb-5">
          <div>
            <p class="text-sm text-neutral-600">Дата: {{ currentWorkout.date }}</p>
            <p class="text-sm text-neutral-600">Подходов: {{ currentWorkout.sets.length }}</p>
          </div>
          <div class="space-x-2">
            <button @click="clearCurrent" class="inline-flex items-center rounded-lg border border-neutral-300 px-3 py-2 text-neutral-700 hover:bg-neutral-50 transition">Очистить</button>
            <button @click="endWorkout" class="inline-flex items-center rounded-lg bg-red-600 px-4 py-2 text-white hover:bg-red-700">Завершить</button>
          </div>
        </div>

        <!-- Hints for planned sequence -->
        <div v-if="currentWorkout.planExercises?.length" class="mb-4 text-sm text-neutral-700">
          <p class="font-medium mb-2">План на сегодня:</p>
          <ul class="list-disc pl-5 space-y-1">
            <li v-for="e in currentWorkout.planExercises" :key="e.id">
              {{ exerciseName(e.exerciseId) }} — {{ e.sets }}×{{ e.reps }}<span v-if="e.weight"> @ {{ e.weight }} кг</span>
              <span v-if="lastPerformance(e.exerciseId)" class="text-neutral-500"> (прошлый раз: {{ lastPerformance(e.exerciseId) }})</span>
            </li>
          </ul>
        </div>

        <form @submit.prevent="addSet" class="grid grid-cols-1 md:grid-cols-4 gap-3 mb-5">
          <select v-model="newSet.exerciseId" required class="border border-neutral-300 rounded-lg px-3 py-2 bg-white focus:outline-none focus:ring-2 focus:ring-black/10 focus:border-black/30">
            <option disabled value="">Выберите упражнение</option>
            <option v-for="ex in exerciseDatabase" :key="ex.id" :value="ex.id">{{ ex.name }}</option>
          </select>
          <input v-model.number="newSet.reps" type="number" min="1" placeholder="Повторения" required class="border border-neutral-300 rounded-lg px-3 py-2 bg-white placeholder:text-neutral-400 focus:outline-none focus:ring-2 focus:ring-black/10 focus:border-black/30" />
          <input v-model.number="newSet.weight" type="number" min="0" step="0.5" placeholder="Вес (кг)" required class="border border-neutral-300 rounded-lg px-3 py-2 bg-white placeholder:text-neutral-400 focus:outline-none focus:ring-2 focus:ring-black/10 focus:border-black/30" />
          <button type="submit" class="inline-flex items-center justify-center rounded-lg bg-black px-4 py-2 text-white hover:bg-neutral-800">Добавить подход</button>
        </form>

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

      <!-- Plan picker modal -->
      <div v-if="openPlanPicker" class="fixed inset-0 bg-black/30 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="w-full max-w-lg rounded-xl bg-white p-5 shadow-lg">
          <div class="flex items-center justify-between mb-3">
            <h3 class="font-semibold">Выберите план</h3>
            <button @click="openPlanPicker=false" class="text-neutral-500">✕</button>
          </div>
          <div v-if="plannedWorkouts.length" class="space-y-2 max-h-72 overflow-auto">
            <div v-for="p in plannedWorkoutsSorted" :key="p.id" class="rounded-lg border p-3 flex items-center justify-between">
              <div>
                <p class="font-medium">{{ p.name || 'План' }}</p>
                <p class="text-sm text-neutral-600">{{ formatDate(p.date) }} · {{ p.exercises.length }} упражн.</p>
              </div>
              <button @click="startFromPlan(p.id); openPlanPicker=false" class="rounded-lg bg-black px-3 py-2 text-white">Запустить</button>
            </div>
          </div>
          <p v-else class="text-sm text-neutral-600">Нет запланированных тренировок</p>
        </div>
      </div>
    </section>

    <!-- PLANNER -->
    <section class="space-y-3">
      <h2 class="text-xl font-semibold tracking-tight">Запланировать тренировку</h2>
      <div class="rounded-xl border border-neutral-200 bg-white/70 shadow-sm p-5">
        <form @submit.prevent="createPlan" class="grid md:grid-cols-4 gap-3 mb-4">
          <input v-model="planForm.name" placeholder="Название" class="border border-neutral-300 rounded-lg px-3 py-2" />
          <input v-model="planForm.date" type="date" required class="border border-neutral-300 rounded-lg px-3 py-2" />
          <select v-model="planExercise.exerciseId" class="border border-neutral-300 rounded-lg px-3 py-2">
            <option disabled value="">Упражнение</option>
            <option v-for="ex in exerciseDatabase" :key="ex.id" :value="ex.id">{{ ex.name }}</option>
          </select>
          <div class="grid grid-cols-3 gap-2 md:col-span-4">
            <input v-model.number="planExercise.sets" type="number" min="1" placeholder="Подходы" class="border border-neutral-300 rounded-lg px-3 py-2" />
            <input v-model.number="planExercise.reps" type="number" min="1" placeholder="Повторы" class="border border-neutral-300 rounded-lg px-3 py-2" />
            <input v-model.number="planExercise.weight" type="number" min="0" step="0.5" placeholder="Вес (кг)" class="border border-neutral-300 rounded-lg px-3 py-2" />
          </div>
          <div class="md:col-span-4 flex gap-2">
            <button type="button" @click="addExerciseToPlan" class="rounded-lg border border-neutral-300 px-3 py-2">Добавить упражнение</button>
            <button type="submit" class="rounded-lg bg-black px-4 py-2 text-white">Сохранить план</button>
          </div>
        </form>
        <div v-if="planPreview.length" class="text-sm">
          <p class="font-medium mb-2">Будут добавлены:</p>
          <ul class="list-disc pl-5 space-y-1">
            <li v-for="(e,i) in planPreview" :key="i">{{ exerciseName(e.exerciseId) }} — {{ e.sets }}×{{ e.reps }} <span v-if="e.weight">@ {{ e.weight }} кг</span></li>
          </ul>
        </div>
      </div>
    </section>

    <!-- CALENDAR -->
    <section class="space-y-3">
      <h2 class="text-xl font-semibold tracking-tight">Календарь</h2>
      <div class="rounded-xl border border-neutral-200 bg-white/70 shadow-sm p-5">
        <div class="flex items-center justify-between mb-3">
          <button class="px-2 py-1 border rounded" @click="prevMonth">←</button>
          <div class="font-medium">{{ monthLabel }}</div>
          <button class="px-2 py-1 border rounded" @click="nextMonth">→</button>
        </div>
        <div class="grid grid-cols-7 gap-1 text-center text-xs text-neutral-600 mb-1">
          <div v-for="d in ['Пн','Вт','Ср','Чт','Пт','Сб','Вс']" :key="d">{{ d }}</div>
        </div>
        <div class="grid grid-cols-7 gap-1">
          <div v-for="(cell, idx) in calendarCells" :key="idx" class="aspect-square">
            <button v-if="cell"
              @click="selectDate(cell.date)"
              :class="['w-full h-full rounded-lg border text-sm flex flex-col items-center justify-center', selectedDateISO===cell.date ? 'border-black' : 'border-neutral-200']">
              <span>{{ cell.day }}</span>
              <span v-if="cell.marks.completed" class="mt-1 inline-block h-1 w-1 rounded-full bg-green-600"></span>
              <span v-if="cell.marks.planned" class="mt-1 inline-block h-1 w-1 rounded-full bg-blue-600"></span>
            </button>
          </div>
        </div>
        <div class="mt-4 space-y-2">
          <div v-for="w in workoutsOnSelected" :key="w.id" class="rounded-lg border p-3 flex items-center justify-between">
            <div>
              <p class="font-medium">{{ w.name || (w.status==='planned'?'Запланировано':'Тренировка') }}</p>
              <p class="text-sm text-neutral-600">{{ w.exercises?.length || 0 }} упражн. · {{ w.sets?.length || 0 }} подходов</p>
            </div>
            <div class="space-x-2">
              <button v-if="w.status==='planned'" @click="startFromPlan(w.id)" class="rounded-lg bg-black px-3 py-2 text-white">Запустить</button>
              <button v-else @click="openWorkoutDetails(w)" class="rounded-lg border px-3 py-2">Открыть</button>
            </div>
          </div>
          <p v-if="!workoutsOnSelected.length" class="text-sm text-neutral-500">Нет тренировок в этот день</p>
        </div>
      </div>
    </section>

    <!-- EXERCISES -->
    <section class="space-y-3">
      <h2 class="text-xl font-semibold tracking-tight">База упражнений</h2>
      <div class="rounded-xl border border-neutral-200 bg-white/70 shadow-sm p-5 space-y-4">
        <form @submit.prevent="addExercise" class="grid md:grid-cols-3 gap-3">
          <input v-model="newExerciseName" placeholder="Название упражнения" class="border border-neutral-300 rounded-lg px-3 py-2" />
          <input v-model="newExerciseTip" placeholder="Краткая техника/подсказка" class="border border-neutral-300 rounded-lg px-3 py-2" />
          <button type="submit" class="rounded-lg bg-black px-4 py-2 text-white">Добавить</button>
        </form>
        <div class="grid md:grid-cols-2 gap-3">
          <div v-for="ex in exerciseDatabase" :key="ex.id" class="rounded-lg border p-3">
            <div class="flex items-center justify-between">
              <p class="font-medium">{{ ex.name }}</p>
              <button @click="editExercise(ex)" class="text-sm text-neutral-600 underline">Редактировать</button>
            </div>
            <p class="text-sm text-neutral-600 mt-1">{{ ex.tip }}</p>
          </div>
        </div>
      </div>
    </section>

    <!-- ANALYTICS -->
    <section class="space-y-3">
      <h2 class="text-xl font-semibold tracking-tight">Аналитика и рекорды</h2>
      <div class="rounded-xl border border-neutral-200 bg-white/70 shadow-sm p-5 space-y-4">
        <div class="flex flex-wrap gap-3 items-end">
          <div class="flex-1 min-w-[220px]">
            <label class="block text-xs text-neutral-500 mb-1">Упражнение</label>
            <select v-model="analyticsExerciseId" class="w-full border border-neutral-300 rounded-lg px-3 py-2">
              <option v-for="ex in exerciseDatabase" :key="ex.id" :value="ex.id">{{ ex.name }}</option>
            </select>
          </div>
          <div>
            <p class="text-sm text-neutral-700">Текущий рекорд: <strong>{{ prFor(analyticsExerciseId) || '—' }} кг</strong></p>
          </div>
        </div>
        <div class="w-full h-40 border rounded-lg p-2 bg-white">
          <svg viewBox="0 0 100 40" preserveAspectRatio="none" class="w-full h-full">
            <polyline :points="analyticsPoints" fill="none" stroke="#0ea5e9" stroke-width="2" />
          </svg>
          <p class="text-[10px] text-neutral-500 mt-1">Динамика максимального веса по датам</p>
        </div>
      </div>
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
        { id: 1, name: 'Жим лежа', tip: 'Лопатки сведены, стопы прижаты, контролируйте амплитуду.' },
        { id: 2, name: 'Приседания', tip: 'Нейтральная спина, колени по направлению носков.' },
        { id: 3, name: 'Становая тяга', tip: 'Спина ровная, штанга вдоль ног, толкайте пол.' },
        { id: 4, name: 'Жим стоя', tip: 'Корпус напряжен, не переразгибайте поясницу.' },
        { id: 5, name: 'Подтягивания', tip: 'Плечи вниз-назад, без рывков.' },
        { id: 6, name: 'Тяга в наклоне', tip: 'Тяните локтями, спина ровная.' },
        { id: 7, name: 'Разгибания ног', tip: 'Контроль траектории, не бросайте вес.' },
        { id: 8, name: 'Сгибания ног', tip: 'Без отрыва таза, плавно.' },
        { id: 9, name: 'Жим гантелей', tip: 'Локти ~45°, контролируйте траекторию.' },
        { id: 10, name: 'Разводки', tip: 'Малый вес, растяжение грудных.' },
        { id: 11, name: 'Французский жим', tip: 'Локти фиксированы, работайте трицепсом.' },
        { id: 12, name: 'Подъемы на бицепс', tip: 'Без раскачки, локти прижаты.' },
      ],
      allWorkouts: JSON.parse(localStorage.getItem('allWorkouts')) || [],
      plannedWorkouts: JSON.parse(localStorage.getItem('plannedWorkouts')) || [],
      currentWorkout: JSON.parse(localStorage.getItem('currentWorkout')) || null,
      newSet: { exerciseId: '', reps: null, weight: null },
      openPlanPicker: false,
      planForm: { name: '', date: new Date().toISOString().slice(0,10) },
      planExercise: { exerciseId: '', sets: null, reps: null, weight: null },
      planPreview: [],
      // Calendar state
      calYear: new Date().getFullYear(),
      calMonth: new Date().getMonth(),
      selectedDateISO: new Date().toISOString().slice(0,10),
      // Analytics
      analyticsExerciseId: 1,
      // Quotes
      quotes: [
        'Сегодня лучше, чем вчера. Завтра — лучше, чем сегодня.',
        'Дисциплина сильнее мотивации.',
        'Маленькие шаги ведут к большим результатам.',
        'Сила растет, когда ты думаешь, что больше не можешь.',
      ],
      quoteIndex: 0,
      tg: null,
      newExerciseName: '',
      newExerciseTip: '',
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
    plannedWorkouts: {
      deep: true,
      handler(v) { localStorage.setItem('plannedWorkouts', JSON.stringify(v)) }
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
      const name = this.newExerciseName?.trim()
      const tip = this.newExerciseTip?.trim() || ''
      if (!name) return
      this.exerciseDatabase.push({ id: Date.now(), name, tip })
      this.newExerciseName = ''
      this.newExerciseTip = ''
    },
    editExercise(ex) {
      const name = prompt('Название упражнения:', ex.name)
      if (name) ex.name = name
      const tip = prompt('Подсказка по технике:', ex.tip || '')
      if (tip !== null) ex.tip = tip
    },
    startWorkout() {
      this.currentWorkout = { id: Date.now(), date: this.todayISO(), sets: [], fromPlanId: null, planExercises: [] }
    },
    startFromPlan(planId) {
      const p = this.plannedWorkouts.find(x => x.id === planId)
      if (!p) return
      this.currentWorkout = { id: Date.now(), date: p.date, sets: [], fromPlanId: p.id, planExercises: p.exercises || [] }
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
      const finished = { ...this.currentWorkout, status: 'completed' }
      this.allWorkouts.unshift(finished)
      if (finished.fromPlanId) {
        this.plannedWorkouts = this.plannedWorkouts.filter(p => p.id !== finished.fromPlanId)
      }
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
    // Planning
    addExerciseToPlan() {
      const { exerciseId, sets, reps, weight } = this.planExercise
      if (!exerciseId || !sets || !reps) return
      this.planPreview.push({ exerciseId, sets, reps, weight })
      this.planExercise = { exerciseId: '', sets: null, reps: null, weight: null }
    },
    createPlan() {
      if (!this.planForm.date || !this.planPreview.length) return
      const plan = {
        id: Date.now(),
        name: this.planForm.name?.trim() || 'Тренировка',
        date: this.planForm.date,
        exercises: this.planPreview.slice(),
        status: 'planned',
      }
      this.plannedWorkouts.push(plan)
      this.planForm = { name: '', date: this.todayISO() }
      this.planPreview = []
    },
    // Helpers
    todayISO() { return new Date().toISOString().slice(0,10) },
    formatDate(iso) { try { return new Date(iso).toLocaleDateString() } catch { return iso } },
    lastPerformance(exId) {
      for (const w of this.allWorkouts) {
        if (w.sets && w.sets.length) {
          const s = [...w.sets].reverse().find(x => x.exerciseId === exId)
          if (s) return `${s.weight} кг × ${s.reps}`
        }
      }
      return ''
    },
    // Calendar
    firstDayOfMonth(year, month) { return new Date(Date.UTC(year, month, 1)) },
    daysInMonth(year, month) { return new Date(Date.UTC(year, month+1, 0)).getUTCDate() },
    selectDate(iso) { this.selectedDateISO = iso },
    prevMonth() { if (this.calMonth===0) { this.calMonth=11; this.calYear-- } else { this.calMonth-- } },
    nextMonth() { if (this.calMonth===11) { this.calMonth=0; this.calYear++ } else { this.calMonth++ } },
    // Quotes
    shuffleQuote() { this.quoteIndex = (this.quoteIndex + 1) % this.quotes.length },
    // Analytics
    prFor(exId) {
      let max = 0
      for (const w of this.allWorkouts) {
        for (const s of (w.sets || [])) {
          if (s.exerciseId === exId) max = Math.max(max, Number(s.weight)||0)
        }
      }
      return max || 0
    },
    openWorkoutDetails(w) { alert(`${w.date}: ${w.sets?.length||0} подходов`) },
  },
  computed: {
    weekly() {
      const now = new Date()
      const start = new Date(now)
      start.setDate(now.getDate() - 6)
      start.setHours(0,0,0,0)
      let workouts = 0, sets = 0, volume = 0
      for (const w of this.allWorkouts) {
        const d = new Date(w.date)
        if (d >= start && d <= now) {
          workouts++
          for (const s of (w.sets || [])) {
            sets++
            volume += (Number(s.weight)||0) * (Number(s.reps)||0)
          }
        }
      }
      return { workouts, sets, volume }
    },
    nextPlanned() {
      const today = this.todayISO()
      return [...this.plannedWorkouts]
        .filter(p => p.date >= today)
        .sort((a,b) => a.date.localeCompare(b.date))[0] || null
    },
    plannedWorkoutsSorted() {
      return [...this.plannedWorkouts].sort((a,b)=>a.date.localeCompare(b.date))
    },
    monthLabel() {
      const d = new Date(Date.UTC(this.calYear, this.calMonth, 1))
      return d.toLocaleString(undefined, { month: 'long', year: 'numeric' })
    },
    calendarCells() {
      const y=this.calYear, m=this.calMonth
      const first = this.firstDayOfMonth(y,m)
      const days = this.daysInMonth(y,m)
      let startIdx = (first.getUTCDay()+6)%7
      const cells = []
      for (let i=0;i<startIdx;i++) cells.push(null)
      for (let d=1; d<=days; d++) {
        const iso = new Date(Date.UTC(y,m,d)).toISOString().slice(0,10)
        const marks = {
          planned: this.plannedWorkouts.some(p=>p.date===iso),
          completed: this.allWorkouts.some(w=>w.date===iso)
        }
        cells.push({ day: d, date: iso, marks })
      }
      while (cells.length % 7) cells.push(null)
      return cells
    },
    workoutsOnSelected() {
      const iso = this.selectedDateISO
      const planned = this.plannedWorkouts.filter(p=>p.date===iso)
      const completed = this.allWorkouts.filter(w=>w.date===iso)
      return [...planned, ...completed]
    },
    currentQuote() { return this.quotes[this.quoteIndex] },
    analyticsPoints() {
      const exId = this.analyticsExerciseId
      const map = new Map()
      for (const w of [...this.allWorkouts].reverse()) {
        let max = 0
        for (const s of (w.sets || [])) {
          if (s.exerciseId === exId) max = Math.max(max, Number(s.weight)||0)
        }
        if (max>0) map.set(w.date, max)
      }
      const arr = [...map.entries()].sort((a,b)=>a[0].localeCompare(b[0]))
      if (!arr.length) return ''
      const maxY = Math.max(...arr.map(x=>x[1])) || 1
      const stepX = arr.length>1 ? 100/(arr.length-1) : 100
      return arr.map(([,y],i)=>`${(i*stepX).toFixed(2)},${(40 - (y/maxY)*36 + 2).toFixed(2)}`).join(' ')
    }
  }
}
</script>

<style scoped>
/* Дополнительные мелкие правки при желании */
</style>
