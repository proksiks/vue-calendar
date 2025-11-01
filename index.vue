<template>
    <div class="calendar">
        <div class="calendar-header">
            <button @click="prevMonth">&lt;</button>
            <span>{{ monthLabel }} {{ currentYear }}</span>
            <button @click="nextMonth">&gt;</button>
        </div>

        <div class="calendar-weekdays">
            <span v-for="day in daysShort" :key="day">{{ day }}</span>
        </div>

        <div class="calendar-days">
            <span class="calendar-day calendar-day_empty" v-for="n in paddingDays" :key="`p-${n}`"></span>
            <button v-for="day in daysInMonth" :key="day" :class="['calendar-day', { selected: isSelected(day) }]"
                @click="selectDay(day)">
                {{ day }}
            </button>
        </div>
    </div>
</template>

<script lang="ts" setup>
type Locale = 'en' | 'ru';

interface CalendarProps {
    modelValue?: string;
    locale?: Locale;
}


const props = defineProps<CalendarProps>();
const emit = defineEmits<{ (e: 'update:modelValue', value: string): void }>();

const locales = {
    en: {
        months: [
            "Jenuary",
            "February",
            "March",
            "April",
            "May",
            "June",
            "July",
            "August",
            "September",
            "October",
            "November",
            "December"
        ],
        daysShort: ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"],
    },
    ru: {
        months: [
            "Январь",
            "Февраль",
            "Март",
            "Апрель",
            "Май",
            "Июнь",
            "Июль",
            "Август",
            "Сентябрь",
            "Октябрь",
            "Ноябрь",
            "Декабрь"
        ],
        daysShort: ["Вс", "Пн", "Вт", "Ср", "Чт", "Пт", "Сб"],
    },
};

const getToday = () => {
    const today = new Date();

    return {
        year: today.getFullYear(),
        month: today.getMonth(),
        day: today.getDate(),
    }
}

const initialDate = ref(props.modelValue ? new Date(props.modelValue) : new Date());

const currentYear = ref(initialDate.value.getFullYear());
const currentMonth = ref(initialDate.value.getMonth());
const currentDay = ref(initialDate.value.getDate());

const daysShort = computed(() => locales[props.locale || 'ru'].daysShort);

const monthLabel = computed(() => locales[props.locale || 'ru'].months[currentMonth.value]);

const daysInMonth = computed(() => {
    const date = new Date(currentYear.value, currentMonth.value + 1, 0);
    return date.getDate();
});

const firstDayOfMonthWeekday = computed(() => {
    const date = new Date(currentYear.value, currentMonth.value, 1);
    return date.getDay();
});

const paddingDays = computed(() => firstDayOfMonthWeekday.value);

function prevMonth() {
    if (currentMonth.value === 0) {
        currentMonth.value = 11;
        currentYear.value--;
    } else {
        currentMonth.value--;
    }
}

function nextMonth() {
    if (currentMonth.value === 11) {
        currentMonth.value = 0;
        currentYear.value++;
    } else {
        currentMonth.value++;
    }
}

function isSelected(day: number) {
    return (
        day === currentDay.value &&
        currentMonth.value === initialDate.value.getMonth() &&
        currentYear.value === initialDate.value.getFullYear()
    )
}


function selectDay(day: number) {
    currentDay.value = day
    const date = new Date(currentYear.value, currentMonth.value, day);
    const yyyy = date.getFullYear();
    const mm = `${date.getMonth() + 1}`.padStart(2, '0');
    const dd = `${day}`.padStart(2, '0');
    emit('update:modelValue', `${yyyy}-${mm}-${dd}`);
}

watch(() => props.modelValue, (newValue) => {
    if (newValue) {
        initialDate.value = new Date(newValue);
        currentYear.value = initialDate.value.getFullYear();
        currentMonth.value = initialDate.value.getMonth();
        currentDay.value = initialDate.value.getDate();
    }
}, { immediate: true })
</script>

<style scoped>
.calendar {
    max-width: 21.25rem;
    background-color: #f8fafc;
    border-radius: 1rem;
    box-shadow: 0 0.5rem 2rem rgba(60, 80, 150, 0.07), 0 0.125rem, 0.75rem #e7eaff;
    padding: 1.5rem, 1rem 1rem 1rem;
    margin: 2rem, auto;
    font-family: inherit;
    transition: box-shadow 0.2s ease-in-out;
}

.calendar-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-weight: 500;
    margin-bottom: 1rem;
    counter-reset: #465a73;
    background-color: #eceefd;
    padding: 0.5rem;
    border-radius: 0.75rem;
    box-shadow: 0 0.125rem 0.375rem 0 #e8eaf6;
}

.calendar-header button {
    background-color: #e3e6fa;
    border: none;
    color: #7992c6;
    font-size: 1.125rem;
    width: 2rem;
    height: 2rem;
    border-radius: 0.5rem;
    cursor: pointer;
    outline: none;
    transition: background-color 0.2s ease-in-out;

}

.calendar-header button:hover {
    background-color: #d1d8f5;
}

.calendar-weekdays,
.calendar-days {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 0.5rem;
    margin-bottom: 0.5rem;
}

.calendar-weekdays span {
    color: #7d87b6;
    font-weight: 500;
    padding: 0.5rem 0;
    font-size: 1rem;
    text-align: center;
    background-color: #f3f5fd;
    border-radius: 0.5rem;
}

.calendar-day {
    background-color: #eef2fa;
    color: #516182;
    font-size: 1.125rem;
    border-radius: 0.5rem;
    height: 2.375rem;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: background-color 0.2s ease-in-out, color 0.2s ease-in-out;
    user-select: none;
    box-shadow: 0 0.125rem 0 #ececfc;
    border: none;
}

.calendar-day_empty {
    background-color: none;
    cursor: default;
    box-shadow: none;
}

.calendar-day.selected {
    background-color: #bcd8ff;
    color: #3465be;
    font-weight: 700;
    box-shadow: 0 0 0 0.125rem #afd2ff;
}

.calendar-day:hover:not(.calendar-day_empty):not(.selected) {
    background-color: #ddeafe;
    color: #164f9f;
}

@media (max-width: 480px) {
    .calendar {
        padding: 1rem 0.5rem 0.52rem 0.5rem;
        font-size: 0.875rem;
    }

    .calendar-header {
        padding: 0.5rem 0.25rem;
        font-size: 0.875rem;
    }

    .calendar-weekdays span,
    .calendar-day {
        font-size: 0.875rem;
        height: 2rem;
    }
}
</style>