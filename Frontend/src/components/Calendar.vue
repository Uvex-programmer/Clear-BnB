<template>
  <div class="calendar">
    <form class="box-bg" @submit.prevent>
      <div class="mb-4">
        <div class="text">
          <span><slot class="start-text" name="start"></slot></span>
          <span><slot class="end-text" name="end"></slot></span>
        </div>
        <DatePicker
          color="green"
          :columns="size < 800 ? 1 : 2"
          v-if="range.end != null"
          v-model="range"
          mode="date"
          :masks="masks"
          is-range
          :min-date="
            rentalObject == undefined ? new Date() : rentalObject.availableFrom
          "
          :max-date="rentalObject == undefined ? '' : rentalObject.availableTo"
          :disabled-dates="disabledDates"
        >
          <template v-slot="{ inputValue, inputEvents, isDragging }">
            <div
              class="date-range"
              :class="[booking ? 'solid-border white' : '']"
            >
              <div
                class="single-date-box"
                :class="[
                booking ? 'smaller-width font-smaller-thicker border-right' : '', 
                addHouse ? 'border  border-radius' : '',
                searchBar ? 'nav-width': '']"
              >
                <svg
                  class="calendar-logo"
                  fill="none"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  viewBox="0 0 24 24"
                  stroke="black"
                >
                  <path
                    d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
                  ></path>
                </svg>
                <input
                  class="chosen-date-box border-radius"
                  :class="[
                    isDragging ? 'text-gray-500' : 'text-black-500',
                    booking ? 'booking change-width' : '',
                    size < 430 ? 'booking' : '', 
                    addHouse ? 'bg' : '',
                    searchBar ? 'border1' : '',
                    searchModal ? 'bg border' : '',
                  ]"
                  :value="inputValue.start"
                  v-on="inputEvents.start"
                />
              </div>
              <span
                class="divider-arrow-box"
                :class="booking ? 'display-none' : ''"
              >
                <svg class="divider-arrow" viewBox="0 0 24 24">
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M14 5l7 7m0 0l-7 7m7-7H3"
                  />
                </svg>
              </span>
              <div
                class="single-date-box text-black-500"
                :class="[
                  booking ? 'smaller-width font-smaller-thicker margin' : '',
                  addHouse? 'border  border-radius' : '',
                  searchBar ? 'nav-width': ''
                ]"
              >
                <svg
                  class="calendar-logo"
                  fill="none"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  viewBox="0 0 24 24"
                  stroke="black"
                >
                  <path
                    d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
                  ></path>
                </svg>
                <input
                  class="chosen-date-box co border-radius"
                  :class="[
                    isDragging ? 'text-gray-500' : 'text-black-500',
                    addHouse ? 'bg' : '',
                    searchBar ? 'border1' : '',
                    searchModal ? 'bg border' : '',
                    booking ? 'booking change-width' : '',
                  ]"
                  :value="inputValue.end"
                  v-on="inputEvents.end"
                />
              </div>
            </div>
          </template>
        </DatePicker>
      </div>
    </form>
  </div>
</template>

<script>
import { Calendar, DatePicker } from 'v-calendar'

export default {
  components: {
    Calendar,
    DatePicker,
  },

  props: ['textOne', 'searchBar', 'booking', 'addHouse', 'searchModal'],

  unmounted() {
    window.removeEventListener('resize', this.myEventHandler)
  },

  data() {
    return {
      receipts: null,
      rentalObject: '',
      range: {
        start: new Date().setHours(0, 0, 0, 0),
        end: null,
      },
      masks: {
        input: 'YYYY-MM-DD',
      },
      disabledDates: [],
      size: window.innerWidth,
    }
  },

  watch: {
    range: function () {
      this.$store.commit('setChosenDates', [
        this.range.start.valueOf(),
        this.range.end.valueOf(),
      ])
      this.$emit('dates', this.range.start, this.range.end)
      this.$emit('days-selected', this.findSelectedDays().length - 1)
      this.$emit('dateArray', this.findAllNights())
    },
  },

  methods: {
    myEventHandler(e) {
      this.size = window.innerWidth
    },

    findAllNights() {
      let allDates = this.findSelectedDays()
      allDates.pop()
      return allDates
    },
    findSelectedDays() {
      for (
        var arr = [], dt = new Date(this.range.start);
        dt <= this.range.end;
        dt.setDate(dt.getDate() + 1)
      ) {
        arr.push(new Date(dt))
      }
      return arr
    },
    findAllDisabledDates() {
      let disabled = []
      for (let i = 0; i < this.receipts.length; i++) {
        for (
          let dt = new Date(this.receipts[i].startDate);
          dt <= this.addDays(new Date(this.receipts[i].endDate), -1); //-1 to avoid disabling the check-out date
          dt.setDate(dt.getDate() + 1)
        ) {
          disabled.push(new Date(dt))
        }
        this.disabledDates = disabled
      }
      this.findFirstAvailable()
    },
    async filterReceipts() {
      this.receipts = await this.$store.state.rentalReceipts.filter(
        (rec) => this.rentalObject.id == rec.rentalObjectId
      )
      this.findAllDisabledDates()
    },
    addDays(firstDate, daysToAdd) {
      let secondDate = new Date(firstDate.valueOf())
      secondDate.setDate(secondDate.getDate() + daysToAdd)
      return secondDate
    },
    findFirstAvailable() {
      let startDate = new Date(this.range.start)
      let endDate = this.addDays(startDate, 1)
      for (let j = 0; j < this.disabledDates.length; j++) {
        if (
          (startDate.getDate() === this.disabledDates[j].getDate() &&
            startDate.getMonth() === this.disabledDates[j].getMonth()) ||
          (endDate.getDate() === this.disabledDates[j].getDate() &&
            endDate.getMonth() === this.disabledDates[j].getMonth())
        ) {
          startDate = this.addDays(this.disabledDates[j], 1)
          endDate = this.addDays(this.disabledDates[j], 2)
          j = 0
        }
      }
      this.range.start = startDate
      this.range.end = this.addDays(endDate, 1)
    },
    checkDateEquality(firstDate, secondDate) {
      return firstDate.valueOf() == secondDate.valueOf()
    },
  },

  async created() {
    this.rentalObject = this.$store.state.rentalObject
    if (this.rentalObject != undefined) {
      this.range.start =
        this.rentalObject.availableFrom.valueOf() >
        new Date().setHours(0, 0, 0, 0).valueOf()
          ? this.rentalObject.availableFrom
          : new Date().setHours(0, 0, 0, 0)
      this.rentalObject.availableFrom =
        this.rentalObject.availableFrom.valueOf() <
        new Date().setHours(0, 0, 0, 0).valueOf()
          ? new Date().setHours(0, 0, 0, 0)
          : this.rentalObject.availableFrom
    }
    if (this.rentalObject != undefined) {
      await this.filterReceipts()
    }
    this.range.end = this.addDays(this.range.start, 2)

    window.addEventListener('resize', this.myEventHandler)
  },
}
</script>

<style scoped>
.white {
  background-color: white;
  padding: 10px;
}

.border-right {
  border-right: 1px solid black;
}

.height32{
  height: 32px;
}

.border-none{
  border: none;
}

.calendar {
  /* font-family: 'Yanone Kaffeesatz', 'Lucida Grande', Lucida, Verdana, sans-serif; */
  display: flex;
  width: 100%;
  flex-direction: column;
  justify-content: center;
}

.change-width {
  width: 8rem !important;
}

.display-none {
  display: none !important;
}

.booking {
  border-radius: 7px;
  
  height: 2rem;
}

.select-date {
  text-align: left;
  font-size: 0.875rem;
  font-weight: 700;
  display: block;
}

.date-range {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-self: flex-end;
  min-width: 100%;
}
.solid-border {
  font-size: 1.1rem;
  border: 1px solid black;
  border-radius: 7px;
}

.single-date-box {
  align-self: center;
  position: relative;
  flex-grow: 1;
  max-width: 8.5rem;
}

.calendar-logo {
  width: 1rem;
  position: absolute;
  pointer-events: none;
  margin: 0 0.5rem 0 0.5rem;
  height: 100%;
}

.chosen-date-box {
  width: 100%;
  padding: 0.25rem 0.25rem 0.25rem 2rem;
  flex-grow: 1;
  border: none;
  line-height: inherit;
  overflow: visible;
  font-family: inherit;
  font-size: 100%;
  background-color: inherit;
}

.padding10{
  padding: 10px;
}

.border {
  border: solid 1px grey;
}
.divider-arrow-box {
  display: flex;
  margin: 0.5rem;
  flex-shrink: 0;
}
.divider-arrow {
  width: 1rem;
  stroke: currentColor;
  height: 1rem;
  display: block;
  align-self: center;
}

.text {
  display: flex;
  justify-content: space-around;
  font-size: 1rem;
}
.start-text {
  margin-left: 2rem;
}
.end-text {
  margin-left: 0.2;
}

.text-gray-500 {
  --text-opacity: 1;
  color: rgba(8, 8, 8, 0.61);
  font-weight: 500;
}

.text-black-500 {
  --text-opacity: 1;
  color: black, var(--text-opacity);
  font-weight: 500;
}

  .box-bg{
    padding: 0 10px;
  }
.bg {
  background: rgb(255, 255, 255);
}

.nav-width{
  width: 8.5rem;
}
  

@media screen and (max-width: 450px) {
  .box-bg{
    padding: 0;
  }
}
@media screen and (max-width: 400px) {
  .single-date-box {
    width: 130px;
  }
}

@media screen and (max-width: 400px) {
  .date-range {
    padding-left: 0;
    padding-right: 0;
  }
}

</style>
