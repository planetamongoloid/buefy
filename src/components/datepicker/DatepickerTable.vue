<template>
    <section class="datepicker-table">
        <header class="datepicker-header">
            <div v-for="day in visibleDayNames"
                :key="day"
                class="datepicker-cell">
                {{ day }}
            </div>
        </header>
        <div class="datepicker-body">
            <b-datepicker-table-row
                v-for="(week, index) in weeksInThisMonth(focused.month, focused.year)"
                :key="index"
                :selectedDate="value"
                :week="week"
                :month="focused.month"
                :min-date="minDate"
                :max-date="maxDate"
                :disabled="disabled"
                :oneselectable-dates="oneselectableDates"
                :unselectable-dates="unselectableDates"
                @select="updateSelectedDate">
            </b-datepicker-table-row>
        </div>
    </section>
</template>

<script>
    import bDatepickerTableRow from './DatepickerTableRow'

    export default {
        name: 'bDatepickerTable',
        components: {
            bDatepickerTableRow
        },
        props: {
            value: Date,
            dayNames: Array,
            monthNames: Array,
            firstDayOfWeek: Number,
            minDate: Date,
            maxDate: Date,
            focused: Object,
            disabled: Boolean,
            oneselectableDates: Array,
            unselectableDates: Array
        },
        computed: {
            visibleDayNames() {
                const visibleDayNames = []
                let index = this.firstDayOfWeek
                while (visibleDayNames.length < this.dayNames.length) {
                    const currentDayName = this.dayNames[(index % this.dayNames.length)]
                    visibleDayNames.push(currentDayName)
                    index++
                }
                return visibleDayNames
            }
        },
        methods: {
            /*
            * Emit input event with selected date as payload for v-model in parent
            */
            updateSelectedDate(date) {
                this.$emit('input', date)
            },

            /*
            * Return array of all days in the week that the startingDate is within
            */
            weekBuilder(startingDate, month, year) {
                const thisMonth = new Date(year, month)

                const thisWeek = []

                const dayOfWeek = new Date(year, month, startingDate)
                    .getDay()

                const end = dayOfWeek >= this.firstDayOfWeek
                    ? (dayOfWeek - this.firstDayOfWeek) : ((7 - this.firstDayOfWeek) + dayOfWeek)

                let daysAgo = 1
                for (let i = 0; i < end; i++) {
                    thisWeek.unshift(new Date(thisMonth.getFullYear(), thisMonth.getMonth(), startingDate - daysAgo))
                    daysAgo++
                }

                thisWeek.push(new Date(year, month, startingDate))

                let daysForward = 1
                while (thisWeek.length < 7) {
                    thisWeek.push(new Date(year, month, startingDate + daysForward))
                    daysForward++
                }

                return thisWeek
            },

            /*
            * Return array of all weeks in the specified month
            */
            weeksInThisMonth(month, year) {
                const weeksInThisMonth = []
                const daysInThisMonth = new Date(year, month + 1, 0).getDate()

                let startingDay = 1

                while (startingDay <= daysInThisMonth + 6) {
                    const newWeek = this.weekBuilder(startingDay, month, year)
                    let weekValid = false

                    newWeek.forEach((day) => {
                        if (day.getMonth() === month) {
                            weekValid = true
                        }
                    })

                    if (weekValid) {
                        weeksInThisMonth.push(newWeek)
                    }

                    startingDay += 7
                }

                return weeksInThisMonth
            }
        }
    }
</script>
