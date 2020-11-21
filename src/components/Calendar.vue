<template>
  <div>
    <div class="datapicker__top-part">
      <div
        class="arrow"
        @click="previousMonth"
      >&#60;</div>
      <span class="datapicker__calendar-month">
        {{ monthToString }}
      </span>
      <div
        class="arrow"
        @click="nextMonth"
      >&#62;</div>
      <div
        class="arrow"
        @click="previousYear"
      >&#60;</div>
      <span class="datapicker__calendar-year">
        {{ year }}
      </span>
      <div
        class="arrow"
        @click="nextYear"
      >&#62;</div>
    </div>

    <thead class="weekdays">
      <tr
        v-for="name in weekDays"
        :key="name.id"
        class="weekdays__day"
      >
        {{ name }}
      </tr>
    </thead>
    <tbody
      id="datapicker__calendar-body"
      class="date"
    >
      <tr
        v-for="prevDay in prevMonthDays"
        :key="prevDay.id"
        class="date__day date__day--grey"
        @click="selectedValuePrevMonth(prevDay)"
      >
        <td> {{ prevDay }}</td>
      </tr>
      <tr
        v-for="day in days"
        :key="day.id"
        class="date__day date__day--regular"
        :class="{'date__day--selected' : Number(inputDay) === day}"
        tabindex="0"
        @click="selectedValue(day)"
      >
        <td
          v-if="day === 1"
          class="date__day--purple"
        >
          {{ day }}
        </td>
        <td v-else>
          {{ day }}
        </td>
      </tr>
      <tr
        v-for="nextDay in nextMonthDays"
        :key="nextDay.id"
        class="date__day date__day--grey"
        @click="selectedValueNextMonth(nextDay)"
      >
        <td>{{ nextDay }}</td>
      </tr>
    </tbody>
  </div>
</template>

<script>

export default {
  name: 'Calendar',

  props: {
    isOpen: {
      type: Boolean,
      default: false,
    },
    inputDay: {
      type: String,
      default: '',
    },
    inputMonth: {
      type: String,
      default: '',
    },
    inputYear: {
      type: String,
      default: '',
    },
  },

  data () {
    return {
      internalValue: '',
      days: [],
      prevMonthDays: [],
      nextMonthDays: [],
      day: '',
      monthToString: '',
      month: '',
      monthNumber: '',
      year: '',
      weekDays: [
        'Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс',
      ],
      monthName: [
        'Январь',
        'Февраль',
        'Март',
        'Апрель',
        'Май',
        'Июнь',
        'Июль',
        'Август',
        'Сентябрь',
        'Октябрь',
        'Ноябрь',
        'Декабрь',
      ],
      showError: false,
      closeWindow: false,
      chosenDay: false,
      date: '',
    }
  },

  watch: {
    inputDay (val) {
      this.day = val
      this.genCalendar()
    },
    inputMonth (val) {
      this.month = val
      this.genCalendar()
    },
    inputYear (val) {
      this.year = val
      this.genCalendar()
    },
  },

  created () {
    this.genCalendar()
  },

  methods: {
    genCalendar (day = this.inputDay, month = this.inputMonth, year = this.inputYear) {
      this.days = []
      this.date = new Date()
      console.log('day', day)

      this.genMonth(month)
      this.genYear(year)
      const daysInMonth = new Date(this.year, this.month, 0).getDate()
      for (let eachDay = 1; eachDay < daysInMonth + 1; eachDay++) {
        this.days.push(eachDay)
      }
      this.genPreviousMonth()
      this.genNextMonth()
    },

    genMonth (month) {
      month = Number(month)
      if (month === 0) {
        this.month = this.date.getMonth() + 1
        this.monthToString = this.monthName[this.month - 1]
      } else {
        this.month = month
        this.monthToString = this.monthName[this.month - 1]
      }
    },

    genYear (year) {
      if (year.length < 4) {
        this.year = this.date.getFullYear()
      } else {
        this.year = year
      }
    },

    genPreviousMonth () {
      this.prevMonthDays = []
      // первый день недели месяца
      const prevMonth = this.month - 1
      const firstDay = new Date(this.year, prevMonth, 1)
      let firstDayWeekday = firstDay.getDay()
      if (firstDayWeekday !== 0) {
        firstDayWeekday--
      } else {
        firstDayWeekday = 6
      }
      // выщитываем, сколько дней из предыдущего месяца нунжо добавить в календарь
      const daysFromPreviousMonth = new Date(this.year, prevMonth, 0).getDate()
      const daysInFirstRow = daysFromPreviousMonth - firstDayWeekday
      for (let daysInPrevMonth = daysFromPreviousMonth; daysInPrevMonth > daysInFirstRow; daysInPrevMonth--) {
        this.prevMonthDays.push(daysInPrevMonth)
      }
      this.prevMonthDays.reverse()
    },

    genNextMonth () {
      this.nextMonthDays = []
      // последний день недели месяца
      const lastDayOfCurrentMonth = new Date(this.year, this.month, 0)
      const lastDayWeekday = lastDayOfCurrentMonth.getDay()
      const daysInLastRow = 7 - lastDayWeekday
      // считаем денёчки всё
      if (daysInLastRow !== 7) {
        for (let dayFromNextMonth = 1; dayFromNextMonth <= daysInLastRow; dayFromNextMonth++) {
          this.nextMonthDays.push(dayFromNextMonth)
        }
      }
    },

    nextMonth () {
      this.month++
      if (this.month > 12) {
        this.month = 1
        this.year = this.year + 1
      }

      this.monthToString = this.monthName[this.month - 1]
      this.days = new Date(this.year, this.month, 0).getDate()
      this.genPreviousMonth()
      this.genNextMonth()
    },

    previousMonth () {
      this.month--

      if (this.month < 1) {
        this.month = 12
        this.year = this.year - 1
      }

      this.monthToString = this.monthName[this.month - 1]
      this.days = new Date(this.year, this.month, 0).getDate()
      this.genPreviousMonth()
      this.genNextMonth()
    },

    nextYear () {
      this.year = this.year + 1
      this.days = new Date(this.year, this.month, 0).getDate()
      this.genPreviousMonth()
      this.genNextMonth()
    },

    previousYear () {
      this.year = this.year - 1
      this.days = new Date(this.year, this.month, 0).getDate()
      this.genPreviousMonth()
      this.genNextMonth()
    },

    selectedValue (day) {
      (day < 10 && !(day.toString()).startsWith('0')) ? this.day = '0' + day : this.day = day

      if (this.month < 10 && this.month[0] !== '0') {
        this.month = '0' + this.month
      }

      this.internalValue = `${this.day}.${this.month}.${this.year}`
      this.$emit('selected', this.internalValue)
    },

    selectedValuePrevMonth (day) {
      (day < 10 && !(day.toString()).startsWith('0')) ? this.day = '0' + day : this.day = day

      if (this.month === 1) {
        this.month = 12
        this.monthNumber = 12
        this.year = this.year - 1
      } else {
        this.month--
        (this.month < 10 && this.month[0] !== '0') ? this.monthNumber = '0' + this.month : this.monthNumber = this.month
      }
      this.days = new Date(this.year, this.month, 0).getDate()
      this.monthToString = this.monthName[this.month - 1]
      this.internalValue = `${this.day}.${this.monthNumber}.${this.year}`
      this.genPreviousMonth()
      this.genNextMonth()
      this.genCalendar()
      this.$emit('selected', this.internalValue)
    },

    selectedValueNextMonth (day) {
      (day < 10 && !(day.toString()).startsWith('0')) ? this.day = '0' + day : this.day = day

      if (this.month === 12) {
        this.month = 1
        this.monthNumber = '01'
        this.year = this.year + 1
      } else {
        this.month++
        (this.month < 10 && this.month[0] !== '0') ? this.monthNumber = '0' + this.month : this.monthNumber = this.month
      }

      this.days = new Date(this.year, this.month, 0).getDate()
      this.monthToString = this.monthName[this.month - 1]
      this.internalValue = `${this.day}.${this.monthNumber}.${this.year}`
      this.genPreviousMonth()
      this.genNextMonth()
      this.genCalendar()
      this.$emit('selected', this.internalValue)
    },
  },
}
</script>

<style lang="scss" scoped>

.datapicker {
  height: 40px;
  width: 180px;
  position: relative;
  margin: 0 !important;

  &__top-part {
    display: flex;
    justify-content: space-between;
    -webkit-user-select: none; /* Chrome/Safari */
    -moz-user-select: none; /* Firefox */
    -ms-user-select: none;
    -o-user-select: none;
    user-select: none;
  }

  &__calendar {
    margin-top: -4px;
    position: absolute;
    height: 310px;
    width: 300px;
    background: #ffffff;
    z-index: 100;
    font-size: 15px;
    border-radius: 20px;
    border: 1px solid gray;
    padding-top: 6px;
    padding-bottom: 6px;
    padding: 20px;

    &:before {
      background: #ffffff;
      background: -webkit-gradient(linear, left top, left bottom, from(grey), to(#ffffff));
      background: linear-gradient(to bottom, grey, #ffffff);
      border-radius: 20px;
      content: "";
      display: block;
      height: 6px;
      left: 3px;
      position: absolute;
      right: 0;
      top: 0;
      width: 98%;
    }

    &-year {
      height: 20px;
      padding: 0 20px;
      -webkit-user-select: none; /* Chrome/Safari */
      -moz-user-select: none; /* Firefox */
      -ms-user-select: none;
      -o-user-select: none;
      user-select: none;
    }

    &-month {
      height: 20px;
      padding: 0 20px;
      width: 110px;
      text-align: center;
      -webkit-user-select: none; /* Chrome/Safari */
      -moz-user-select: none; /* Firefox */
      -ms-user-select: none;
      -o-user-select: none;
      user-select: none;
    }
  }
}

.arrow {
  cursor: pointer;
  height: 15px;
}

.weekdays {
  color: darkgrey;
  display: inline-grid;
  grid-template-columns: repeat(7, 1fr);
  width: 100%;
  height: 12px;
  margin: 20px 0;

  &__day {
    display: flex;
    justify-content: center;
    align-items: center;
  }
}

.date {
  display: inline-grid;
  grid-template-columns: repeat(7, 1fr);
  width: 100%;
  height: 75%;

  &__day {
    cursor: pointer;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 28px;
    width: 28px;
    margin: auto;

    &--regular:hover {
      background-color:  rgb(241, 237, 245);
    }

    &--purple {
      color: purple;
    }

    &--grey {
      color:gainsboro;
    }

    &--grey:hover {
      color: gray;
    }

    &--grey:active {
      color: black;
    }

    &--selected {
      color: #ffffff;
      background-color: purple !important;
      border-radius: 50%;
      outline: none !important;
    }

    &:hover {
      border-radius: 50%;
    }

    &:focus {
      color: #ffffff;
      background-color: purple;
      border-radius: 50%;
      outline: none !important;
    }
  }
}

</style>
