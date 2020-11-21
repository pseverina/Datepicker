<template>
  <div class="datapicker">
    <input
      v-model="internalValue"
      placeholder="ДД.MM.ГГГГ"
      class="datapicker__input"
      :class="svzDatapickerInput"
      @click="isOpen = !isOpen"
      @mouseover="hover = true"
      @mouseleave="hover = false"
    >
    <transition name="fade">
      <calendar
        v-if="isOpen"
        class="datapicker__calendar"
        :isOpen="isOpen"
        :inputDay="inputDay"
        :inputMonth="inputMonth"
        :inputYear="inputYear"
        @selected="chosenDate"
      ></calendar>
    </transition>
  </div>
</template>

<script>
import Calendar from './Calendar'

export default {
  name: 'Datapicker',

  components: {
    Calendar,
  },

  data () {
    return {
      isOpen: false,
      internalValue: '',
      showError: false,
      closeWindow: false,
      hover: false,
      inputDay: '',
      inputMonth: '',
      inputYear: '',
    }
  },

  computed: {
    svzDatapickerInput () {
      return [{
        'datapicker__input--error': this.showError,
        'datapicker__input--active': this.isOpen,
      }]
    },

    calendarStatus () {
      return [{
        'calendar--active': !this.internalValue.length && this.isOpen,
        'calendar--hidden': this.internalValue.length,
        'calendar--nonactive': !this.internalValue.length && !this.isOpen,
      }]
    },

    closeStatus () {
      return [{
        'close--active': this.internalValue.length,
        'close--hidden': !this.internalValue.length,
      }]
    },
  },

  watch: {
    internalValue (value) {
      this.internalValue = value
      this.inputDay = this.internalValue.slice(0, 2)
      this.inputMonth = this.internalValue.slice(3, 5)
      this.inputYear = this.internalValue.slice(6, 10)

      if (!this.internalValue.length) { this.showError = false }
      (this.internalValue.length) ? this.isOpen = true : this.isOpen = false

      if (this.internalValue.length > 1) {
        if (this.inputDay > 31) this.showError = true
        if (this.inputDay === '00') this.showError = true
      }

      if (this.internalValue.length > 4) {
        if (this.inputMonth > 12 || this.inputMonth < 1) {
          this.showError = true
        } else if (this.inputMonth === '02' && this.inputDay > 29) {
          this.showError = true
        } else {
          this.showError = false
        }
      }
    },
  },

  methods: {
    closeCalendar () {
      this.isOpen = false
    },

    chosenDate (value) {
      this.internalValue = value
    },

    clearInput () {
      this.internalValue = ''
    },
  },
}

</script>

<style lang="scss" scoped>

$dark-blue: #4D6C9E;

.datapicker {
  height: 40px;
  width: 180px;
  position: relative;
  margin: 0 auto;

  &__input {
    width: 200px;
    height: 25px;
    position: relative;
    border-radius: 10px;
    font-size: 15px;
    text-align: center;
    margin-bottom: 3px;
    outline: none;

    &::placeholder {
      font-size: 16px;
      color: grey;
      padding: 0 5px;
    }

    &:hover {
      border-color: darkgrey;
    }
    
    &:focus {
      border-color: $dark-blue;
      cursor: pointer;
    }

    &:disabled {
      border-color: gainsboro;
      color: gainsboro;
      &:hover {
        border-color: gainsboro;
      }
    }

    &--error:focus {
      border: 1px #FC213B solid;
    }

    &--active {
      border-color: $dark-blue;
    }
  }

  &__calendar {
    height: 310px;
    width: 300px;
    background: #FFFFFF;
    z-index: 100;
    font-size: 15px;
    border-radius: 20px;
    border: 2px solid $dark-blue;
    padding-top: 6px;
    padding-bottom: 6px;
    padding: 20px;

    &--active {
      fill: grey;
    }
  }
}

.calendar {
  &--hidden {
    display: none;
  }
  &--active {
    display: inline;
    fill: black;
  }
  &--nonactive {
    display: inline;
    fill: black;
  }
}

.calendar:hover + .datapicker__input{
  border-color: yellowgreen;
}

.datapicker__input:hover + .calendar,
.datapicker__input:focus + .calendar,
.datapicker__input:active + .calendar{
  fill: black;
}

.datapicker__input:disabled + .calendar {
  fill: gainsboro;
}

.close {
  &--hidden {
    display: none;
  }
  &--active {
    display: inline;
    cursor: pointer;
  }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}

</style>
