<template>
    <div class="calendar-container">
      <vue-cal
        id="calendar"
        class="vuecal"
        hide-view-selector
        active-view="month"
        :events="formattedHolidays"
        :time="false"
        :disable-views="['years','year','week','day']"
        :selected-date="selectedDate"
        events-on-month-view="long"
      />
    </div>
</template>
  
<script>
  import VueCal from "vue-cal";
  import "vue-cal/dist/vuecal.css";
  
  export default {
    components: { VueCal },
    props: {
      holidays: {
        type: Array,
        required: true,
      },
      selectedYear: {
        type: Number,
        required: true,
      },
    },
    computed: {
        // Format holidays into events for the calendar
        formattedHolidays() {
            return this.holidays.map((holiday) => ({
                start: holiday.date,
                end: holiday.date,
                title: holiday.name,
                class: 'holiday-event',
            }));
        },
        // Set selected date to the first day of the selected year
        selectedDate() {
            const currentDay = new Date().getDate();
            const currentMonth = new Date().getMonth() + 1;
            return `${this.selectedYear}-${currentMonth}-${currentDay}`;
        },
    },
  };
</script>
  
<style>
    .calendar-container {
        position: relative;
        height: 55vh;
        width: auto;
        margin: 15px 0;
    }

    .holiday-event {
        background-color: #f0fff1;
        color: #004D41;
    }

    .vuecal__title-bar {
        background-color: #00AB95;
    }

    .vuecal--month-view .vuecal__cell-content .vuecal__cell {
        justify-content: flex-start;
        height: 100%;
        align-items: flex;
    }
</style>