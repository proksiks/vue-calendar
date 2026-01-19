<template>
  <div class="min-h-screen bg-linear-to-br from-emerald-50 to-teal-50 py-12 px-4">
    <div class="max-w-7xl mx-auto">
      <div class="text-center mb-12">
        <h1 class="text-4xl font-bold bg-linear-to-r from-emerald-700 to-teal-700 bg-clip-text text-transparent mb-4">
          Vue Calendar Component
        </h1>
        <p class="text-gray-600 text-lg max-w-2xl mx-auto">
          Полностью настраиваемый календарь с поддержкой локализации, диапазонов дат и анимациями.
        </p>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
        <div class="lg:col-span-2">
          <div class="bg-white rounded-2xl shadow-2xl p-6">
            <h2 class="text-2xl font-bold text-gray-800 mb-6">Демонстрация календаря</h2>

            <div class="mb-8">
              <Calendar ref="calendarRef" v-model="selectedDate" :locale="selectedLocale"
                :first-day-of-week="firstDayOfWeek" :min-date="minDate" :max-date="maxDate"
                :disabled-dates="disabledDates" :highlight-today="highlightToday" @month-change="handleMonthChange"
                @day-select="handleDaySelect" />
            </div>

            <div class="space-y-6">
              <div class="bg-gray-50 rounded-xl p-4">
                <h3 class="font-semibold text-gray-700 mb-3">Быстрые действия</h3>
                <div class="flex flex-wrap gap-3">
                  <button @click="goToToday"
                    class="px-4 py-2 bg-emerald-500 text-white rounded-lg hover:bg-emerald-600 transition-colors">
                    Сегодня
                  </button>
                  <button @click="goToNextMonth"
                    class="px-4 py-2 bg-teal-500 text-white rounded-lg hover:bg-teal-600 transition-colors">
                    Следующий месяц
                  </button>
                  <button @click="goToPrevMonth"
                    class="px-4 py-2 bg-teal-500 text-white rounded-lg hover:bg-teal-600 transition-colors">
                    Предыдущий месяц
                  </button>
                  <button @click="addSpanishLocale" :disabled="isSpanishLocaleAdded" :class="[
                    'px-4 py-2 rounded-lg transition-colors',
                    isSpanishLocaleAdded
                      ? 'bg-purple-300 text-purple-700 cursor-not-allowed'
                      : 'bg-purple-500 text-white hover:bg-purple-600'
                  ]">
                    {{ isSpanishLocaleAdded ? 'Испанский добавлен' : 'Добавить испанский' }}
                  </button>
                </div>
              </div>

              <div class="bg-emerald-50 rounded-xl p-4">
                <h3 class="font-semibold text-emerald-700 mb-2">Выбранная дата</h3>
                <div class="flex items-center space-x-4">
                  <div class="text-2xl font-bold text-emerald-800">
                    {{ formattedSelectedDate }}
                  </div>
                  <div class="text-gray-600">
                    {{ getDayOfWeek(selectedDate) }}
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="space-y-8">
          <div class="bg-white rounded-2xl shadow-2xl p-6">
            <h2 class="text-xl font-bold text-gray-800 mb-4">Настройки локализации</h2>

            <div class="space-y-4">
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Язык</label>
                <select v-model="selectedLocale"
                  class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500">
                  <option v-for="locale in availableLocales" :key="locale" :value="locale">
                    {{ getLocaleName(locale) }}
                  </option>
                </select>
              </div>

              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Первый день недели</label>
                <select v-model="firstDayOfWeek"
                  class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500">
                  <option value="0">Воскресенье</option>
                  <option value="1">Понедельник</option>
                </select>
              </div>

              <div class="flex items-center">
                <input v-model="highlightToday" type="checkbox" id="highlightToday"
                  class="h-4 w-4 text-emerald-600 focus:ring-emerald-500 border-gray-300 rounded" />
                <label for="highlightToday" class="ml-2 block text-sm text-gray-700">
                  Подсвечивать сегодняшний день
                </label>
              </div>
            </div>
          </div>

          <div class="bg-white rounded-2xl shadow-2xl p-6">
            <h2 class="text-xl font-bold text-gray-800 mb-4">Диапазон дат</h2>

            <div class="space-y-4">
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Минимальная дата</label>
                <input v-model="minDate" type="date"
                  class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500" />
              </div>

              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Максимальная дата</label>
                <input v-model="maxDate" type="date"
                  class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500" />
              </div>

              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Отключенные даты</label>
                <div class="space-y-2">
                  <div class="flex items-center space-x-2">
                    <input v-model="disabledDateInput" type="date"
                      class="flex-1 px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500" />
                    <button @click="addDisabledDate"
                      class="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600 transition-colors">
                      Добавить
                    </button>
                  </div>
                  <div v-if="disabledDates.length > 0" class="mt-2">
                    <div class="text-sm text-gray-600 mb-1">Отключенные даты:</div>
                    <div class="flex flex-wrap gap-2">
                      <span v-for="(date, index) in disabledDates" :key="index"
                        class="px-3 py-1 bg-red-100 text-red-700 rounded-full text-sm flex items-center">
                        {{ formatDisplayDate(date) }}
                        <button @click="removeDisabledDate(index)" class="ml-2 text-red-500 hover:text-red-700">
                          ×
                        </button>
                      </span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="bg-white rounded-2xl shadow-2xl p-6">
            <h2 class="text-xl font-bold text-gray-800 mb-4">Лог событий</h2>

            <div class="bg-gray-50 rounded-lg p-4 h-64 overflow-y-auto space-y-3">
              <div v-for="(event, index) in eventLog" :key="index" class="border-l-4 border-emerald-500 pl-3 py-1">
                <div class="text-sm font-medium text-gray-800">{{ event.message }}</div>
                <div class="text-xs text-gray-500">{{ formatTime(event.timestamp) }}</div>
                <div v-if="event.data" class="text-xs text-gray-600 mt-1">
                  <pre class="bg-gray-100 p-2 rounded">{{ JSON.stringify(event.data, null, 2) }}</pre>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'
import Calendar, { type Locale } from "./components/calendar.vue"

const calendarRef = ref<InstanceType<typeof Calendar>>()
const selectedDate = ref<string>('2026-01-11')
const selectedLocale = ref<Locale>('ru')
const firstDayOfWeek = ref<0 | 1>(1)
const minDate = ref<string>('2026-01-01')
const maxDate = ref<string>('2027-12-31')
const disabledDates = ref<(string | Date)[]>([])
const disabledDateInput = ref<string>('')
const highlightToday = ref<boolean>(true)
const availableLocales = ref<Locale[]>(['ru', 'en', 'fr', 'de'])
const isSpanishLocaleAdded = ref<boolean>(false)

const eventLog = ref<Array<{
  message: string
  timestamp: Date
  data?: any
}>>([])

const localeNames: Record<Locale, string> = {
  ru: 'Русский',
  en: 'English',
  fr: 'Français',
  de: 'Deutsch',
  es: 'Español'
}

watch(() => selectedDate.value, (newValue) => {
  if (newValue && calendarRef.value) {
    const date = new Date(newValue);
    if (calendarRef.value && typeof calendarRef.value.goToDate === 'function') {
      calendarRef.value.goToDate(date);
    }
  }
})


const formattedSelectedDate = computed(() => {
  const date = new Date(selectedDate.value)
  return date.toLocaleDateString(selectedLocale.value === 'ru' ? 'ru-RU' : 'en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
})

const getLocaleName = (locale: Locale): string => {
  return localeNames[locale] || locale
}

const logEvent = (message: string, data?: any) => {
  eventLog.value.unshift({
    message,
    timestamp: new Date(),
    data
  })


  if (eventLog.value.length > 10) {
    eventLog.value = eventLog.value.slice(0, 10)
  }
}

const getDayOfWeek = (dateString: string): string => {
  const date = new Date(dateString)
  const days: Record<string, string[]> = {
    ru: ['Воскресенье', 'Понедельник', 'Вторник', 'Среда', 'Четверг', 'Пятница', 'Суббота'],
    en: ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'],
    fr: ['Dimanche', 'Lundi', 'Mardi', 'Mercredi', 'Jeudi', 'Vendredi', 'Samedi'],
    de: ['Sonntag', 'Montag', 'Dienstag', 'Mittwoch', 'Donnerstag', 'Freitag', 'Samstag'],
    es: ['Domingo', 'Lunes', 'Martes', 'Miércoles', 'Jueves', 'Viernes', 'Sábado']
  }
  const localeDays = days[selectedLocale.value]
  const dayIndex = date.getDay();
  const englishDays = days.en;
  return localeDays && localeDays[dayIndex] ? localeDays[dayIndex] : (englishDays && englishDays[dayIndex]) ? englishDays[dayIndex] : 'Unknown';
}

const formatDisplayDate = (date: string | Date): string => {
  const d = typeof date === 'string' ? new Date(date) : date
  return d.toLocaleDateString('ru-RU')
}

const formatTime = (date: Date): string => {
  return date.toLocaleTimeString('ru-RU', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })
}

const addDisabledDate = () => {
  if (disabledDateInput.value) {
    const date = disabledDateInput.value
    disabledDates.value.push(date)
    disabledDateInput.value = ''
    logEvent(`Добавлена отключенная дата: ${formatDisplayDate(date)}`)
  }
}

const removeDisabledDate = (index: number) => {
  const removed = disabledDates.value.splice(index, 1)
  logEvent(`Удалена отключенная дата: ${formatDisplayDate(removed[0] as string | Date)}`)
}

const goToToday = () => {
  const today = new Date()
  const todayFormatted = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`
  selectedDate.value = todayFormatted
  calendarRef.value?.goToToday()
  logEvent('Переход к сегодняшней дате')
}

const goToPrevMonth = () => {
  calendarRef.value?.prevMonth()
  logEvent('Переход к предыдущему месяцу')
}

const goToNextMonth = () => {
  calendarRef.value?.nextMonth()
  logEvent('Переход к следующему месяцу')
}

const addSpanishLocale = () => {
  if (isSpanishLocaleAdded.value) return

  const success = calendarRef.value?.addLocale('es', {
    months: ['Enero', 'Febrero', 'Marzo', 'Abril', 'Mayo', 'Junio', 'Julio', 'Agosto', 'Septiembre', 'Octubre', 'Noviembre', 'Diciembre'],
    daysShort: ['Dom', 'Lun', 'Mar', 'Mié', 'Jue', 'Vie', 'Sáb'],
    daysLong: ['Domingo', 'Lunes', 'Martes', 'Miércoles', 'Jueves', 'Viernes', 'Sábado']
  })

  if (success) {
    if (!availableLocales.value.includes('es')) {
      availableLocales.value.push('es')
    }
    isSpanishLocaleAdded.value = true
    logEvent('Добавлен испанский язык (es)', {
      months: ['Enero', 'Febrero', 'Marzo', 'Abril', 'Mayo', 'Junio', 'Julio', 'Agosto', 'Septiembre', 'Octubre', 'Noviembre', 'Diciembre']
    })
  } else {
    logEvent('Испанский язык уже добавлен')
  }
}

const handleMonthChange = (payload: { year: number; month: number }) => {
  const monthNames: Record<string, string[]> = {
    ru: ['Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь', 'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь'],
    en: ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'],
    fr: ['Janvier', 'Février', 'Mars', 'Avril', 'Mai', 'Juin', 'Juillet', 'Août', 'Septembre', 'Octobre', 'Novembre', 'Décembre'],
    de: ['Januar', 'Februar', 'März', 'April', 'Mai', 'Juni', 'Juli', 'August', 'September', 'Oktober', 'November', 'Dezember'],
    es: ['Enero', 'Febrero', 'Marzo', 'Abril', 'Mayo', 'Junio', 'Julio', 'Agosto', 'Septiembre', 'Octubre', 'Noviembre', 'Diciembre']
  }

  const localeMonths = monthNames[selectedLocale.value] || monthNames.en;
  const monthIndex = payload.month >= 0 && payload.month < 12 ? payload.month : 0;
  const monthName = localeMonths && localeMonths.length > monthIndex ? localeMonths[monthIndex] : 'Unknown';
  logEvent(`Изменен месяц: ${monthName} ${payload.year}`, payload)
}

const handleDaySelect = (payload: { date: Date; formatted: string }) => {
  logEvent(`Выбрана дата: ${formatDisplayDate(payload.date)}`, payload)
}

onMounted(() => {
  disabledDates.value = [
    '2026-01-01',
    '2026-01-07',
    '2026-02-23',
    '2026-03-08',
    '2026-05-01'
  ]

  logEvent('Календарь инициализирован', {
    selectedDate: selectedDate.value,
    locale: selectedLocale.value
  })
})
</script>

<style scoped>
pre {
  font-family: 'JetBrains Mono', 'Fira Code', monospace;
}

::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}
</style>