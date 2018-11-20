<template>
  <div class="calendar">
    <ul class="week">
      <li v-for="(weekDayName, index) in weekDayNames" v-bind:key="weekDayName" class="cell cell--dayName"
        v-bind:class="{ 
          'cell--marginRight': index===4,
          'cell--gray': index===5 || index==6,
        }">
        {{ weekDayName }}
      </li>
    </ul>
    <ul v-for="week in calendar" v-bind:key="week.week" class="week">
      <li v-for="day in week.days" v-bind:key="day.date()" class="cell cell--day"
          v-bind:class="{ 
            'cell--borderTop': hasBorderTop(day, week),
            'cell--borderLeft': hasBorderLeft(day, week),
            'cell--marginRight': hasMarginRight(day, week),
            'cell--gray': isGray(day),
          }">
        {{ formatDay(day) }}
      </li>
    </ul>
  </div>
</template>

<script>
import * as moment from 'moment'
import 'moment/locale/pl'

//config:
window.moment = moment //spike
moment.updateLocale('pl', {
  weekdaysShort : ['ND', 'PON', 'WT', 'ŚR', 'CZW', 'PT', 'SB']
})
moment.locale('pl')


//spike todo: calculate from given day, currently today
const WEEKS_NUMBER = 12
const FIRST_DAY_OF_WEEK = 1; // monday
const weekDayNamesStandard = moment.weekdaysShort()
const shiftedPartOfWeek = weekDayNamesStandard.slice(0, FIRST_DAY_OF_WEEK)
const restPartOfWeek = weekDayNamesStandard.slice(FIRST_DAY_OF_WEEK, 7)
const WEEK_DAY_NAMES = restPartOfWeek.concat(shiftedPartOfWeek);

const DATE_FORMATS = [
  'DD',
  'D.M',
  'DD.MM',
  'DD.MM.YYYY',
  'D-M',
  'DD-MM',
  'DD-MM-YYYY'
]

const constructCalendar = (startDayString) => {
    const startDay = startDayString ?
      moment(startDayString, DATE_FORMATS) :
      moment().startOf('day');
      console.log(startDayString, startDay.format('YYYY-MM-DD')) //spike
    const year = startDay.year();

    let calendar = [];
    const startWeek = startDay.week();
    const endWeek = startWeek + WEEKS_NUMBER;

    for(var week = startWeek; week<endWeek; week++){
      calendar.push({
        week: week,
        days: Array(7).fill(0).map((n, i) => moment().year(year).week(week).startOf('week').clone().add(i, 'day'))
      })
    }
    return calendar;
}

export default {
  name: 'HelloWorld',
  data() {
    return {
      calendar: [],
      weekDayNames: WEEK_DAY_NAMES
    }
  },
  created() {
    this.calendar = constructCalendar(this.$route.params.date)
  },

  watch: {
    '$route' () {
      this.calendar = constructCalendar(this.$route.params.date)
    }
  },

  methods: {
    formatDay(day) {
      if(this.shouldShowMonthName(day)) {
        return day.format("D MMMM")
      } else {
        return day.format("D")
      }
    },
    shouldShowMonthName(day) {
      return this.isFirstDayInCalendar(day) || this.isFirstDayOfMonth(day)
    },
    isFirstDayOfMonth(day){
      return day.date() === 1
    },
    isFirstDayInCalendar(day) {
      // debugger
      return day === this.calendar[0].days[0]
    },

    hasBorderLeft(day, week){
      const isFirstDayOfWeek = day === week.days[0];
      
      return !isFirstDayOfWeek && this.isFirstDayOfMonth(day)
    },

    hasBorderTop(day) {
      const monthOfDay = day.month()
      const monthOfDayBeforeOneWeek = day.clone().subtract(1, 'week').month()
      const isFirstWeekOfCalendar = this.calendar[0].days.includes(day)

      return !isFirstWeekOfCalendar && monthOfDay !== monthOfDayBeforeOneWeek;
    },

    hasMarginRight(day){
      return day.day() === 5; // Friday
    },

    isGray(day) {
      return day.day() === 6 || day.day() === 0; // Saturday or Sunday
    },
    
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.calendar {
  --border-color: black;
  --cell-width: 25mm;
  --cell-height: 20.5mm;
  
  text-align: left;
  border-width: 0 1px 1px 0;
  border-style: solid;
  border-color: var(--border-color);
  display: inline-block;
}

.week {
  border-top: 1px solid var(--border-color);
}

.cell {
  width: var(--cell-width);
  display: inline-block;
  font-size: 0.8em;
  font-weight: 900;
  border-left: 1px solid var(--border-color);
  margin: 0;
  background-color: white;
}

.cell--marginRight {
  border-right: 1px solid var(--border-color);
  margin-right: 1px;
}

.cell--dayName {
  text-align: center;
  height: 3mm;
  padding: 1mm;
  border-bottom: 1px solid black;
  margin-bottom: 1px;
}

.cell--day {
  padding: 1mm;
  height: var(--cell-height);
}

.cell--borderTop {
  border-top: 1px solid var(--border-color);
  height: calc(var(--cell-height) - 1px);
}

.cell--borderLeft {
  border-left: 2px solid var(--border-color);
  width: calc(var(--cell-width) - 1px);
}

.cell--gray {
  background-color: #f0f0f0;
}

ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}
</style>
