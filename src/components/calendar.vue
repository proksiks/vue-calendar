<template>
    <div
        class="bg-linear-to-br from-emerald-50 via-slate-50 to-teal-50 rounded-2xl p-6 shadow-2xl shadow-emerald-200/50 font-sans transition-all duration-300 mx-auto">
        <div
            class="flex items-center justify-between mb-6 bg-emerald-50/80 px-4 py-3 rounded-xl shadow-inner border border-emerald-100">
            <button @click="handlePrevMonth"
                class="bg-emerald-100 hover:bg-emerald-200 active:bg-emerald-300 text-emerald-700 text-xl w-10 h-10 rounded-xl cursor-pointer outline-none transition-all duration-200 font-bold shadow-sm hover:shadow-md flex items-center justify-center group hover:scale-110"
                :disabled="isPrevMonthDisabled"
                :class="{ 'opacity-50 cursor-not-allowed hover:scale-100 hover:bg-emerald-100': isPrevMonthDisabled }"
                aria-label="Previous month">
                <span class="transition-transform duration-200 group-hover:-translate-x-1">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"><path fill="currentColor" d="M9.586 4L3 10.586a2 2 0 0 0 0 2.828L9.586 20a2 2 0 0 0 2.18.434l.145-.068A2 2 0 0 0 13 18.586V16h7a2 2 0 0 0 2-2v-4l-.005-.15A2 2 0 0 0 20 8l-7-.001V5.414A2 2 0 0 0 9.586 4"/></svg>
                </span>
            </button>

            <span class="text-xl font-bold bg-linear-to-r from-emerald-700 to-teal-700 bg-clip-text text-transparent">
                {{ displayMonth }} {{ displayYear }}
            </span>

            <button @click="handleNextMonth"
                class="bg-emerald-100 hover:bg-emerald-200 active:bg-emerald-300 text-emerald-700 text-xl w-10 h-10 rounded-xl cursor-pointer outline-none transition-all duration-200 font-bold shadow-sm hover:shadow-md flex items-center justify-center group hover:scale-110"
                :disabled="isNextMonthDisabled"
                :class="{ 'opacity-50 cursor-not-allowed hover:scale-100 hover:bg-emerald-100': isNextMonthDisabled }"
                aria-label="Next month">
                <span class="transition-transform duration-200 group-hover:translate-x-1">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"><path fill="currentColor" d="M12.089 3.634A2 2 0 0 0 11 5.414L10.999 8H4a2 2 0 0 0-2 2v4l.005.15A2 2 0 0 0 4 16l6.999-.001l.001 2.587A2 2 0 0 0 14.414 20L21 13.414a2 2 0 0 0 0-2.828L14.414 4a2 2 0 0 0-2.18-.434z"/></svg>
                </span>
            </button>
        </div>

        <div class="grid grid-cols-7 gap-2 mb-6">
            <span v-for="day in orderedWeekdays" :key="day"
                class="text-emerald-600 font-bold py-3 text-center text-sm bg-linear-to-r from-emerald-100 to-teal-100 rounded-xl shadow-md border border-emerald-200">
                {{ day }}
            </span>
        </div>

        <div class="relative">
            <transition name="slide" mode="out-in">
                <div :key="`${displayYear}-${displayMonthNumber}`" class="grid grid-cols-7 gap-2">

                    <div v-for="n in daysBeforeFirstDay" :key="`before-${n}`" class="min-h-12 min-w-12"
                        aria-hidden="true" />

                    <button v-for="day in daysInDisplayMonth" :key="day" @click="handleDayClick(day)"
                        :disabled="!isDateSelectable(day)" :class="[
                            'relative rounded-2xl min-h-12 min-w-12 font-bold text-lg shadow-lg border-2 transition-all duration-300 flex items-center justify-center',
                            'focus:outline-none focus:ring-4 focus:ring-emerald-200/60 active:scale-95',
                            {
                                'bg-linear-to-br from-emerald-50 via-white to-teal-50 text-emerald-800 border-transparent hover:border-emerald-300 hover:shadow-2xl hover:scale-105':
                                    !isSelected(day) && isDateSelectable(day),
                                'bg-linear-to-br from-emerald-500 via-teal-500 to-emerald-600 text-white border-emerald-400 shadow-2xl scale-110 font-black ring-4 ring-emerald-300/70':
                                    isSelected(day),
                                'bg-gray-100 text-gray-400 border-gray-200 cursor-not-allowed hover:scale-100 hover:shadow-lg hover:border-gray-200':
                                    !isDateSelectable(day),
                                'border-emerald-300 bg-linear-to-br from-emerald-200 via-white to-teal-200':
                                    isToday(day) && !isSelected(day) && isDateSelectable(day)
                            }
                        ]" :aria-label="getDayAriaLabel(day)" :aria-selected="isSelected(day)"
                        :aria-disabled="!isDateSelectable(day)">
                        <span class="relative z-10 leading-none">{{ day }}</span>


                        <div v-if="isSelected(day)"
                            class="absolute inset-0 rounded-2xl bg-linear-to-r from-emerald-400/30 to-teal-400/30 -z-10 opacity-100 blur-sm" />


                        <div v-else-if="isDateSelectable(day)"
                            class="absolute inset-0 rounded-2xl bg-linear-to-r from-emerald-400/20 to-teal-400/20 -z-10 opacity-0 group-hover:opacity-100 transition-opacity duration-300 blur-sm" />
                    </button>


                    <div v-for="n in daysAfterLastDay" :key="`after-${n}`" class="min-h-12 min-w-12"
                        aria-hidden="true" />
                </div>
            </transition>
        </div>
    </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'

export type Locale = 'en' | 'ru' | 'fr' | 'de' | string

export interface CalendarProps {
    modelValue?: string | Date
    locale?: Locale
    firstDayOfWeek?: number
    minDate?: string | Date
    maxDate?: string | Date
    disabledDates?: (string | Date)[]
    highlightToday?: boolean
}

export interface CalendarEmits {
    (e: 'update:modelValue', value: string): void
    (e: 'month-change', payload: { year: number, month: number }): void
    (e: 'day-select', payload: { date: Date, formatted: string }): void
}

const props = withDefaults(defineProps<CalendarProps>(), {
    locale: 'ru',
    firstDayOfWeek: 1,
    highlightToday: true
})

const emit = defineEmits<CalendarEmits>()

const localeRegistry = ref<Record<string, { months: string[], daysShort: string[], daysLong: string[] }>>({
    en: {
        months: ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'],
        daysShort: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
        daysLong: ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday']
    },
    ru: {
        months: ['Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь', 'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь'],
        daysShort: ['Вс', 'Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб'],
        daysLong: ['Воскресенье', 'Понедельник', 'Вторник', 'Среда', 'Четверг', 'Пятница', 'Суббота']
    },
    fr: {
        months: ['Janvier', 'Février', 'Mars', 'Avril', 'Mai', 'Juin', 'Juillet', 'Août', 'Septembre', 'Octobre', 'Novembre', 'Décembre'],
        daysShort: ['Dim', 'Lun', 'Mar', 'Mer', 'Jeu', 'Ven', 'Sam'],
        daysLong: ['Dimanche', 'Lundi', 'Mardi', 'Mercredi', 'Jeudi', 'Vendredi', 'Samedi']
    },
    de: {
        months: ['Januar', 'Februar', 'März', 'April', 'Mai', 'Juni', 'Juli', 'August', 'September', 'Oktober', 'November', 'Dezember'],
        daysShort: ['So', 'Mo', 'Di', 'Mi', 'Do', 'Fr', 'Sa'],
        daysLong: ['Sonntag', 'Montag', 'Dienstag', 'Mittwoch', 'Donnerstag', 'Freitag', 'Samstag']
    }
})

const addedLocales = ref<Set<string>>(new Set(['en', 'ru', 'fr', 'de']))

const addLocale = (code: string, data: { months: string[], daysShort: string[], daysLong: string[] }) => {
    if (!addedLocales.value.has(code)) {
        localeRegistry.value[code] = data
        addedLocales.value.add(code)
        return true
    }
    return false
}

const parseDate = (date: string | Date | undefined): Date => {
    if (!date) return new Date()
    if (date instanceof Date) return new Date(date.getTime())

    const parsed = new Date(date)
    return isNaN(parsed.getTime()) ? new Date() : parsed
}

const getDisplayDate = () => {
    const selected = parseDate(props.modelValue)
    return new Date(selected.getFullYear(), selected.getMonth(), 1)
}

const displayDate = ref<Date>(getDisplayDate())

const updateDisplayDate = () => {
    const newDisplayDate = getDisplayDate()
    if (
        displayDate.value.getFullYear() !== newDisplayDate.getFullYear() ||
        displayDate.value.getMonth() !== newDisplayDate.getMonth()
    ) {
        displayDate.value = newDisplayDate
    }
}

const localeData = computed(() => {
    const data = localeRegistry.value[props.locale]
    return data || localeRegistry.value.ru
})

const orderedWeekdays = computed(() => {
    const days = localeData.value?.daysShort || []
    return [...days.slice(props.firstDayOfWeek), ...days.slice(0, props.firstDayOfWeek)]
})

const displayYear = computed(() => displayDate.value.getFullYear())
const displayMonthNumber = computed(() => displayDate.value.getMonth())
const displayMonth = computed(() => localeData.value?.months[displayMonthNumber.value])

const daysInDisplayMonth = computed(() => {
    return new Date(displayYear.value, displayMonthNumber.value + 1, 0).getDate()
})

const firstDayOfMonth = computed(() => {
    const firstDay = new Date(displayYear.value, displayMonthNumber.value, 1).getDay()
    return (firstDay - props.firstDayOfWeek + 7) % 7
})

const daysBeforeFirstDay = computed(() => firstDayOfMonth.value)

const totalCells = 42
const daysAfterLastDay = computed(() => {
    const totalUsed = daysBeforeFirstDay.value + daysInDisplayMonth.value
    return Math.max(0, totalCells - totalUsed)
})

const normalizeDate = (date: Date): Date => {
    const normalized = new Date(date)
    normalized.setHours(0, 0, 0, 0)
    return normalized
}


const isDateSelectable = (day: number): boolean => {
    const date = new Date(displayYear.value, displayMonthNumber.value, day)
    const normalizedDate = normalizeDate(date)

    if (props.minDate) {
        const minDate = normalizeDate(parseDate(props.minDate))
        if (normalizedDate < minDate) return false
    }

    if (props.maxDate) {
        const maxDate = normalizeDate(parseDate(props.maxDate))
        if (normalizedDate > maxDate) return false
    }

    if (props.disabledDates?.length) {
        const dateStr = formatDate(date)
        const isDisabled = props.disabledDates.some(disabledDate => {
            const disabled = parseDate(disabledDate)
            return formatDate(disabled) === dateStr
        })
        if (isDisabled) return false
    }

    return true
}

const isSelected = (day: number): boolean => {
    if (!props.modelValue) return false

    const selectedDate = parseDate(props.modelValue)
    return (
        day === selectedDate.getDate() &&
        displayMonthNumber.value === selectedDate.getMonth() &&
        displayYear.value === selectedDate.getFullYear()
    )
}

const isToday = (day: number): boolean => {
    if (!props.highlightToday) return false

    const today = new Date()
    return (
        day === today.getDate() &&
        displayMonthNumber.value === today.getMonth() &&
        displayYear.value === today.getFullYear()
    )
}

const isPrevMonthDisabled = computed(() => {
    if (!props.minDate) return false

    const prevMonth = new Date(displayYear.value, displayMonthNumber.value - 1, 1)
    const minDate = parseDate(props.minDate)
    const minMonth = new Date(minDate.getFullYear(), minDate.getMonth(), 1)
    minMonth.setHours(0, 0, 0, 0)

    return prevMonth < minMonth
})

const isNextMonthDisabled = computed(() => {
    if (!props.maxDate) return false

    const nextMonth = new Date(displayYear.value, displayMonthNumber.value + 1, 1)
    const maxDate = parseDate(props.maxDate)
    const maxMonth = new Date(maxDate.getFullYear(), maxDate.getMonth(), 1)
    maxMonth.setHours(23, 59, 59, 999)

    return nextMonth > maxMonth
})

const handlePrevMonth = () => {
    if (isPrevMonthDisabled.value) return

    displayDate.value = new Date(displayYear.value, displayMonthNumber.value - 1, 1)
    emitMonthChange()
}

const handleNextMonth = () => {
    if (isNextMonthDisabled.value) return

    displayDate.value = new Date(displayYear.value, displayMonthNumber.value + 1, 1)
    emitMonthChange()
}

const handleDayClick = (day: number) => {
    if (!isDateSelectable(day)) return

    const newDate = new Date(displayYear.value, displayMonthNumber.value, day)
    const formatted = formatDate(newDate)

    displayDate.value = new Date(displayYear.value, displayMonthNumber.value, 1)

    emit('update:modelValue', formatted)
    emit('day-select', { date: newDate, formatted })
}

const formatDate = (date: Date): string => {
    const year = date.getFullYear()
    const month = String(date.getMonth() + 1).padStart(2, '0')
    const day = String(date.getDate()).padStart(2, '0')
    return `${year}-${month}-${day}`
}

const getDayAriaLabel = (day: number): string => {
    const date = new Date(displayYear.value, displayMonthNumber.value, day)
    const dayName = localeData.value?.daysLong[date.getDay()]
    return `${dayName}, ${day} ${displayMonth.value} ${displayYear.value}`
}

const emitMonthChange = () => {
    emit('month-change', {
        year: displayYear.value,
        month: displayMonthNumber.value
    })
}

// Это типо классов, когда мы можем сделать публичные и приватные методы
defineExpose({
    addLocale,
    goToToday: () => {
        const today = new Date()
        const todayNormalized = new Date(today.getFullYear(), today.getMonth(), 1)
        displayDate.value = todayNormalized
        emitMonthChange()
    },
    goToDate: (date: string | Date) => {
        const targetDate = parseDate(date)
        displayDate.value = new Date(targetDate.getFullYear(), targetDate.getMonth(), 1)
        emitMonthChange()
    },
    prevMonth: handlePrevMonth,
    nextMonth: handleNextMonth,
    getAvailableLocales: () => Array.from(addedLocales.value),
    updateDisplayDate
})
</script>

<style scoped>
.slide-enter-active,
.slide-leave-active {
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-enter-from {
    opacity: 0;
    transform: translateX(30px);
}

.slide-leave-to {
    opacity: 0;
    transform: translateX(-30px);
}

button[aria-selected="true"] {
    box-shadow:
        0 0 20px rgba(16, 185, 129, 0.6),
        0 0 40px rgba(34, 197, 94, 0.4),
        0 0 60px rgba(16, 185, 129, 0.3);
    animation: pulse 2s infinite;
}

@keyframes pulse {
    0% {
        box-shadow:
            0 0 20px rgba(16, 185, 129, 0.6),
            0 0 40px rgba(34, 197, 94, 0.4),
            0 0 60px rgba(16, 185, 129, 0.3);
    }

    50% {
        box-shadow:
            0 0 25px rgba(16, 185, 129, 0.8),
            0 0 50px rgba(34, 197, 94, 0.6),
            0 0 75px rgba(16, 185, 129, 0.4);
    }

    100% {
        box-shadow:
            0 0 20px rgba(16, 185, 129, 0.6),
            0 0 40px rgba(34, 197, 94, 0.4),
            0 0 60px rgba(16, 185, 129, 0.3);
    }
}
</style>