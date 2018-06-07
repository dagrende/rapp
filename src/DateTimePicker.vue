<template lang="html">
  <div class="picker">
    <div class="stepper">
      <div class="arrow-up" @click="add(adders.years, 1)"></div>
      <div class="number">
        {{year}}
      </div>
      <div class="arrow-down" @click="add(adders.years, -1)"></div>
    </div>
    <div class="separator">
      -
    </div>
    <div class="stepper">
      <div class="arrow-up" @click="add(adders.months, 1)"></div>
      <div class="number">
        {{month}}
      </div>
      <div class="arrow-down" @click="add(adders.months, -1)"></div>
    </div>
    <div class="separator">
      -
    </div>
    <div class="stepper">
      <div class="arrow-up" @click="add(adders.days, 1)"></div>
      <div class="number">
        {{dayOfMonth}}
      </div>
      <div class="arrow-down" @click="add(adders.days, -1)"></div>
    </div>
    <div class="separator space">

    </div>
    <div class="stepper">
      <div class="arrow-up" @click="add(adders.hours, 1)"></div>
      <div class="number">
        {{hour}}
      </div>
      <div class="arrow-down" @click="add(adders.hours, -1)"></div>
    </div>
    <div class="separator">
      :
    </div>
    <div class="stepper">
      <div class="arrow-up" @click="add(adders.minutes, 1)"></div>
      <div class="number">
        {{minute}}
      </div>
      <div class="arrow-down" @click="add(adders.minutes, -1)"></div>
    </div>
  </div>
</template>
<script>
import getYear from 'date-fns/get_year';
import addYears from 'date-fns/add_years';
import getMonth from 'date-fns/get_month';
import addMonths from 'date-fns/add_months';
import getDate from 'date-fns/get_date';
import addDays from 'date-fns/add_days';
import getHours from 'date-fns/get_hours';
import addHours from 'date-fns/add_hours';
import getMinutes from 'date-fns/get_minutes';
import addMinutes from 'date-fns/add_minutes';

export default {
  data() {
    return {
      date: new Date(),
      adders: {years: addYears, months: addMonths, days: addDays, hours: addHours, minutes: addMinutes}
    }
  },
  computed: {
    year() {
      return getYear(this.date)
    },
    month() {
      return zeroPad(getMonth(this.date), 2)
    },
    dayOfMonth() {
      return zeroPad(getDate(this.date), 2)
    },
    hour() {
      return zeroPad(getHours(this.date), 2)
    },
    minute() {
      return zeroPad(getMinutes(this.date), 2)
    },
    monthName() {
      return ['jan', 'feb', 'mar', 'apr', 'maj', 'jun', 'jul', 'aug', 'sep', 'okt', 'nov', 'dec'][this.month]
    }
  },
  methods: {
    add(adder, amount) {
      this.date = adder(this.date, amount);
      this.$emit('change', this.date)
    }
  }
}

function zeroPad(num, places) {
  var zero = places - num.toString().length + 1;
  return Array(+(zero > 0 && zero)).join("0") + num;
}

</script>

<style lang="css" scoped>
.picker {
  display:flex;
  flex-direction: row;
  align-items: center;
}
.stepper {
  display:flex;
  flex-direction: column;
  font-size: 200%;
}
.separator {
  font-size: 200%;
}
.separator.space {
  min-width: 0.5em;
}
.arrow-up {
  width: 0;
  height: 0;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
  border-bottom: 20px solid black;
}
.arrow-down {
  width: 0;
  height: 0;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
  border-top: 20px solid black;
}

</style>
