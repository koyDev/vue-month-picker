<template>
  <div
    class="month-picker__container"
    :class="{
      [`month-picker--${variant}`]: true,
      'year-picker': yearOnly
    }"
  > 
    <div v-if="!step" class="normal">
      <div
        v-if="showYear"
        class="month-picker__year"
      >
        <div class="" v-for=""> 
          <button
            type="button"
            @click="changeYear(-1)"
          >
            <i class="fas fa-caret-left"></i>
          </button>
          <p
            v-if="!editableYear"
          >
            {{ year }}
          </p>
          <input
            v-else
            v-model.number="year"
            type="text"
            @change="onChange()"
          >
          <button
            type="button"
            @click="changeYear(+1)"
          >
            <i class="fas fa-caret-right"></i>
          </button>
        </div>
      </div>
      <div
        v-if="!yearOnly"
        class="month-picker"
        :class="{'visable': next}"
      >
        <div
          v-for="(month, i) in monthsByLang"
          :key="month"
          :class="{
            'inactive': isInactive(month),
            'clearable': clearable,
            'selected': currentMonth === month
          }"
          class="month-picker__month"
          @click="selectMonth(i, true)"
        >
          <div class="mon-btn">{{ month }}</div>
        </div>
      </div>
    </div>
    <!-- step style  -->
    <div v-else class="step">
      <div class="cpnewpick">
        <div
          class="year-part"
          :class="{'visable': !next, 'invisable' : next}"
        >
        <p class="mb-0 text-center">{{ year }}</p>
          <div class="month-picker__year new">
            <button
              type="button"
              @click="changeYear(-1)"
            >
              <i class="fas fa-caret-left"></i>
            </button>
            <p
              v-if="!editableYear && !yearOnly"
              v-for="year in yearRange"
              @click="selectYear"
            >
              {{ year }}
            </p>
            <p
              v-if="!editableYear && yearOnly"
              v-for="year in yearRange"
              @click="selectYearOnly"
            >
              {{ year }}
            </p>
            <!-- <input
              v-else
              v-model.number="year"
              type="text"
              @change="onChange()"
            > -->
            <button
              type="button"
              @click="changeYear(+1)"
            >
              <i class="fas fa-caret-right"></i>
            </button>
          </div>
        </div>
        <div
          v-if="!yearOnly"
          class="month-picker"
          :class="{'visable': next, 'invisable' : !next}"
        >
          <p class="mb-0 text-center">{{ year }}</p>
          <div
            v-for="(month, i) in monthsByLang"
            :key="month"
            :class="{
              'inactive': isInactive(month),
              'clearable': clearable,
              'selected': currentMonth === month
            }"
            class="month-picker__month"
            @click="selectMonth(i, true)"
          >
            <div class="mon-btn">{{ month }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import languages from './languages'
import monthPicker from './month-picker'

export default {
  name: 'MonthPicker',
  mixins: [monthPicker],
  emits: [
    'change',
    'clear',
    'input',
    'change-year'
  ],
  props: {
    step: {
      type: Boolean,
      default: false
    },
    defaultMonth: {
      type: [Number, String],
      default: 0
    },
    defaultYear: {
      type: [Number, String],
      default: 0
    },
    yearOnly: {
      type: Boolean,
      default: false
    }
  },
  data: () => ({
    currentMonthIndex: null,
    year: new Date().getFullYear(),
    initYear: new Date().getFullYear(),
    selectedYear: '',
    next: false
  }),
  computed: {
    currentMonth: function() {
      if (this.currentMonthIndex !== null) {
        return this.monthsByLang[this.currentMonthIndex]
      }

      return null
    },
    date: function() {
      const month = this.monthsByLang.indexOf(this.currentMonth) + 1
      const date = new Date(`${this.year}/${month}/01`)
      const year = date.getFullYear()

      return {
        from: date,
        to: new Date(year, month, 1),
        month: this.monthsByLang[month - 1],
        monthIndex: month,
        year: year
      }
    },
    yearRange() {
     return Array.apply(0, Array(20)).map((element, index) => index + this.initYear - 10)
    }
  },
  watch: {
    defaultMonth (newVal) {
      this.currentMonthIndex = newVal
    },
    defaultYear (newVal) {
      this.year = newVal
    }
  },
  async mounted() {
    if (this.defaultYear) {
      this.year = await this.defaultYear
    }

    if (this.defaultMonth) {
      await this.selectMonth(this.defaultMonth - 1)
    } else if (!this.noDefault) {
      this.selectMonth(0)
    }
  },
  methods: {
    selectYearOnly(e) {
      this.year = e.target.outerText
      this.$emit('input', this.year)
    },
    selectYear(e) {
      this.year = e.target.outerText
      this.$emit('change-year', e.target.outerText)
      this.next = true
    },
    onChange() {
      if (!Number.parseInt(this.year)) {
        this.year = this.defaultYear || new Date().getFullYear()
      }

      this.$emit('change', this.date)
    },
    selectMonth(index, input = false) {
      this.next = false
      if (this.isInactive(index)) {
        return
      }
      const isAlreadySelected = this.currentMonthIndex === index
      if (this.clearable && isAlreadySelected) {
        this.currentMonthIndex = null
        this.$emit('clear')
        return
      }

      if (this.isAlreadySelected) {
        return
      }

      this.currentMonthIndex = index
      this.onChange()

      if (input) {
        this.$emit('input', this.date)
      }
    },
    changeYear(value) {
      if (this.step) {
        this.initYear += value
      } else {
        this.year += value
      }
      if (this.isInactive(0)) {
        return
      }

      this.onChange()
      this.$emit('change-year', this.year)
      console.log(this.year)
    },
    isInactive(month) {
      let monthValue = month
      if (this.minDate === null && this.maxDate === null) {
        return false
      }

      if (Number.isInteger(monthValue)) {
        monthValue = this.monthsByLang[monthValue]
      }

      const monthKey = this.monthsByLang.indexOf(monthValue) + 1
      const date = new Date(`${this.year}/${monthKey}/01`)
      const isValidDate = (date) => date !== null && (date instanceof Date)

      if (isValidDate(this.minDate) && date < this.minDate) {
        return true
      }

      if (isValidDate(this.maxDate) && date > this.maxDate) {
        return true
      }

      return false
    }
  }
}
</script>

<style lang="scss">
.month-picker__container {
  width: 270px;
  position: relative;
  border: 1px solid #DDDDDD;
  border-radius: 5px;
  z-index: 5;
  box-shadow: 0px 3px 6px #00000029;
}

.month-picker__container.year-picker {
  width: 225px;
}
.month-picker {
  box-sizing: border-box;
  flex: 1;
  width: auto;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  box-shadow: 1px 2px 5px rgba(0, 0, 0, 0.1);
  font-family: sans-serif;
  border-radius: 5px;
  overflow: hidden;
  padding:0px 30px 5px;
}

.month-picker__year p {
  width: 100%;
  font-weight: 600;
  margin: 0;
  font-size: 14px;
  padding: 2px 3px;
}

.month-picker__year input {
  padding: 0;
  font-weight: 600;
  border-radius: 5px 5px 0 0;
  outline: none;
  border: none;
  font-size: 1.2em;
  width: auto;
  text-align: center;
  box-sizing: border-box;
  width: 100%;
  height: 3em;
  position: relative;
  z-index: 1;
}

.month-picker__year input:focus {
  border: 1px solid #55B0F2;
}

.month-picker__year div,
.month-picker__year button,
.month-picker__year p {
  text-align: center;
  flex: 1;
}

.month-picker__year button {
  background-color: transparent;
  position: absolute;
  width: 1em;
  font-size: 1.7em;
  border-radius: 5px;
  outline: none;
  border: 0;
  top: 50%;
  /* border: 1px solid #E8E8E8; */
  z-index: 2;
  transform: translateY(-50%);
  color: #ABB1C1;
}

.month-picker__year button:hover {
  /* background-color: rgba(0, 0, 0, 0.025); */
  color:#2B59C3;
}

.month-picker__year button:active {
  /* background-color: rgba(0, 0, 0, 0.04); */
  color:#2B59C3;
}

.month-picker__year button:first-child {
  left: 0px;
}

.month-picker__year button:last-child {
  right: 0px;
}
.normal .month-picker__year::after {
    content: '';
    border-bottom: 2px solid #676767;
    width: 80%;
    display: block;
    margin: auto;
    margin-bottom: 7px;
}
.month-picker__month {
  flex-basis: calc(33.333%);
  padding: 0.2em 0.15em;
  cursor: pointer;
  text-align: center;
  /* border: 1px solid rgba(245, 245, 245, .75); */
  transition: all 250ms cubic-bezier(0.165, 0.84, 0.44, 1);
  /* background-color: #FEFEFE; */
}

.month-picker .month-picker__month:hover {
  /* box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); */
  z-index: 10;
}
.mon-btn {
    font-size: 12px;
    padding: 2px 0px;
}
.month-picker__month.selected {
  /* background-color: #2b59c3;
  color: #FFFFFF;
  border-radius: 55px; */
}
.month-picker__month.selected .mon-btn{
  background-color: #2b59c3;
  color: #FFFFFF;
  border-radius: 55px;
}
.month-picker__month.inactive {
  background-color: #f0f0f0;
  color: #8a8a8a;
  cursor: not-allowed;
  box-shadow: none;
  text-shadow: none;
}

.month-picker__month.inactive:hover {
  box-shadow: none;
}

/* Dark theme */
.month-picker--dark {
  background-color: #5F5F5F;
}

.month-picker--dark .month-picker__year p,
.month-picker--dark .month-picker__year input {
  color: #EBEBEB;
}

.month-picker--dark .month-picker__year input {
  background-color: #5F5F5F;
}

.month-picker--dark .month-picker__year button {
  background-color: #505050;
  color: #C9C9C9;
  border-color: #1E1E1E;
}

.month-picker--dark .month-picker__year button:hover {
  background-color: rgba(0, 0, 0, 0.3);
}

.month-picker--dark .month-picker__year button:active {
  background-color: rgba(0, 0, 0, 0.6);
}

.month-picker--dark .month-picker__month {
  background-color: #2F2F30;
  border-color: rgba(245, 245, 245, .15);
  color: #C9C9C9;
}

.month-picker--dark .month-picker__month.selected {
  background-color: #505050;
  box-shadow: inset 0 0 3px #505050, 0px 2px 5px #505050;
  color: #FFFFFF;
  border-color: #1E1E1E;
}

.month-picker--dark .month-picker__month:hover {
  box-shadow: 0 0 6px rgba(0, 0, 0, 0.8);
  z-index: 10;
}

.month-picker--dark .month-picker__month.inactive {
  background-color: #3f3f3f;
  color: #8a8a8a;
  cursor: not-allowed;
  box-shadow: none;
  text-shadow: none;
}

@media only screen and (max-width: 768px) {
  .month-picker__container {
    width: 100%;
  }
}

.cpnewpick{
  .mb-0.text-center{
    font-size: 12px;
    font-weight: bold;
    border-bottom: 1px solid #707070;
    padding-bottom: 5px;
    width: 100%;
    color:#333;
  }
  .month-picker__year.new {
    display: flex;
    flex-wrap: wrap;
    padding-top:5px;
  }
  .month-picker__year.new p {
    width: 25%;
    flex:unset;
    font-size:12px;
    color:#333;
    cursor: pointer;
    border-radius: 9px;
    &:hover{
      background: #2B59C3;
      color: #fff;
    }
  }
  .invisable{
    opacity: 0;
    display: none !important;
  }
  .visable{
    opacity: 1;
    display: block;
  }
  .year-part{
    padding:4px 20px;
  }
  .month-picker {
    box-sizing: border-box;
    flex: 1;
    width: auto;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    font-family: sans-serif;
    border-radius: 5px;
    overflow: hidden;
    height: 100%;
    align-content: space-between;
    padding:4px 20px;
  }
}
</style>
