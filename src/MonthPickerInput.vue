<template>
  <div
    v-click-outside="hide"
    class="month-picker-input-container"
  >
  <i class="far fa-calendar-alt"></i>
    <input
      v-model="selectedDate"
      class="month-picker-input form-control simple"
      type="text"
      readonly
      @click="showMonthPicker()"
    >
    <month-picker
      v-show="monthPickerVisible"
      :default-year="defaultYear"
      :default-month="defaultMonth"
      :lang="lang"
      :months="months"
      :no-default="noDefault"
      :show-year="showYear"
      :clearable="clearable"
      :variant="variant"
      :editable-year="editableYear"
      :max-date="maxDate"
      :min-date="minDate"
      :year-only="yearOnly"
      @input="populateInput"
      @change="$emit('change', $event)"
    />
  </div>
</template>

<script>
import MonthPicker from './MonthPicker.vue'
import monthPicker from './month-picker'

export default {
  name: 'MonthPickerInput',
  directives: {
    clickOutside: {
      bind: function (el, binding, vnode) {
        el.event = function (event) {
          if (!(el === event.target || el.contains(event.target))) {
            vnode.context[binding.expression](event)
          }
        }
        document.body.addEventListener('click', el.event)
      },
      unbind: function (el) {
        document.body.removeEventListener('click', el.event)
      }
    }
  },
  components: {
		MonthPicker
	},
  components: {
    MonthPicker
  },
  mixins: [monthPicker],
  emits: [
    'change',
    'input'
  ],
  data() {
    return {
      monthPickerVisible: false,
      selectedDate: null
    }
  },
  watch: {
    defaultMonth(v, ov) {
      if(v) {
        this.selectedDate = `${this.monthsByLang[this.defaultMonth - 1]}, ${this.defaultYear}`
      }
    }
  },
  mounted() {
    if (this.inputPreFilled && this.defaultMonth !== null && this.defaultYear !== null) {
      this.selectedDate = `${this.monthsByLang[this.defaultMonth - 1]}, ${this.defaultYear}`
    }
  },
  methods: {
    populateInput(date) {
      this.selectedDate = `${date.month}, ${date.year}`
      this.monthPickerVisible = false
      this.$emit("input", date)
    },
    showMonthPicker() {
      this.monthPickerVisible = !this.monthPickerVisible
    },
    hide() {
      this.monthPickerVisible = false
    }
  }
}
</script>
<style scoped>
  .month-picker-input-container {
    position: relative;
    width: 180px;
    min-width: 140px;
  }

  .month-picker-input {
    font-size: inherit;
    border-radius: 0;
    outline: none;
    border: 0;
    transition: all 350ms cubic-bezier(0.165, 0.84, 0.44, 1);
    width: inherit;
    border-bottom: 1px solid #b1b1b1;
  }

  .month-picker-input:focus {
    border-color: rgba(0,0,0,.25);
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  }

  .month-picker__container {
    position: absolute;
    top: 3.5em;
    right:-40%;
  }
  i.far.fa-calendar-alt {
    position: absolute;
    right: 10%;
    top: 30%;
    font-size: 20px;
    color: #2b59c3;
  }
</style>
