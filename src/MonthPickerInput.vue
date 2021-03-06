<template>
  <div
    v-click-outside="hide"
    class="month-picker-input-container"
  >
  <i class="far fa-calendar-alt" @click="showMonthPicker()"></i>
    <input
      v-model="selectedDate"
      class="month-picker-input form-control simple"
      type="text"
      readonly
      :disabled="disabled"
      :placeholder="placeholder"
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
      :step="step"
      :style="alignment"
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
  mixins: [monthPicker],
  emits: [
    'change',
    'input'
  ],
  props:{
    step: {
      type: Boolean,
      default: false,
    },
    defaultYear: {
      type: [Number, String],
      default: ''
    },
    defaultMonth: {
      type: [Number, String],
      default: ''
    },
    yearOnly: {
      type: Boolean,
      default: false,
    },
    placeholder: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    position: {
      type: String,
      default: 'center'
    }
  },
  data() {
    return {
      monthPickerVisible: false,
      selectedDate: ''
    }
  },
  watch: {
    defaultMonth(v, ov) {
      if(v !== '' || v !== 0) {
        this.selectedDate = `${this.monthsByLang[this.defaultMonth - 1]}, ${this.defaultYear}`
      } else {
        this.selectedDate = ''
      }
    },
    defaultYear(v, ov) {
      if (v && this.defaultMonth === '') {
        this.selectedDate = this.defaultYear
      }
    }
  },
  async mounted() {
    if (this.yearOnly) {
      const x = await this.defaultYear
      this.selectedDate = x.toString()
    }
  },
  computed: {
    alignment() {
      let positonStyle = this.position
      if (this.position == 'left') {
        return { left: 0 + '%' }
      } else if (this.position == 'right') {
        return { right: 0 + '%' }
      } else {
        return {
          left: 50 + '%',
          transform: 'translateX(' + -50 + '%)' 
        }
      }
    }
  },
  methods: {
    populateInput(date) {
      if (typeof(date) === 'string') {
        this.selectedDate = date
        this.$emit("input", date)
        this.monthPickerVisible = false
      } else {
        this.selectedDate = `${date.month}, ${date.year}`
        this.monthPickerVisible = false
        this.$emit("input", date)
      }
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
<style>
  .month-picker-input-container {
    position: relative;
    width: auto;
  }
  .month-picker-input-container.disabled {
    opacity: .4;
  }

  .month-picker-input {
    font-size: inherit;
    border-radius: 0;
    outline: none;
    border: 0;
    transition: all 350ms cubic-bezier(0.165, 0.84, 0.44, 1);
    width: 100%;
    border-bottom: 1px solid #b1b1b1;
  }

  .month-picker-input:focus {
    border-color: rgba(0,0,0,.25);
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  }

  .month-picker__container {
    position: absolute;
    top: 3.5em;
  }
  i.far.fa-calendar-alt {
    position: absolute;
    right: 0%;
    top: 20%;
    font-size: 20px;
    color: #2b59c3;
  }
</style>
