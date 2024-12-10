<template>
  <div class="app-container">
    <!-- Main Content -->
    <main class="main-content">
      <!-- Page Title -->
      <h1 class="text-center mb-4">Public Holidays Finder</h1>

      <!-- Error message component -->
      <ErrorMessage :message="error" />

      <!-- Public Holiday Today Notification -->
      <div v-if="isHolidayToday !== null" class="alert" :class="{'alert-success': isHolidayToday, 'alert-danger': !isHolidayToday}">
        <strong v-if="isHolidayToday">🎉 Today is a public holiday in the selected country!</strong>
        <strong v-else>🚫 Today is not a public holiday in the selected country.</strong>
      </div>

      <!-- Filters for year and sorting -->
      <div class="d-flex mb-4">
        <!-- Year Selector -->
        <select class="form-select me-3" v-model="selectedYear" @change="handleCountrySelected(selectedCountry)">
          <option value="" disabled>Select Year</option>
          <option v-for="year in years" :key="year" :value="year">{{ year }}</option>
        </select>

        <!-- Sorting Selector -->
        <select class="form-select" v-model="selectedSort" @change="sortHolidays">
          <option value="date">Sort by Date</option>
          <option value="name">Sort by Name</option>
        </select>
      </div>

      <!-- Country Selector component -->
      <CountrySelector :countries="countries" @country-selected="handleCountrySelected" />

      <!-- Calendar View -->
      <CalendarView v-if="holidays.length" :holidays="holidays" :selected-year="selectedYear"/>

      <!-- Holiday List component: only shown if holidays are available -->
      <HolidayList v-if="holidays.length" :holidays="holidays" />
    </main>
  </div>
</template>

<script>
import CountrySelector from './components/CountrySelector.vue';
import HolidayList from './components/HolidayList.vue';
import CalendarView from './components/CalendarView.vue';
import ErrorMessage from './components/ErrorMessage.vue';

export default {
  components: { CountrySelector, HolidayList, CalendarView, ErrorMessage },

  data() {
    return {
      // List of countries with their codes and names
      countries: [],

      // Array to store holidays fetched from the API
      holidays: [],

      // Variable to hold error messages for display
      error: '',

      // Variable that holds whether today is a public holiday
      isHolidayToday: null,

      // Variable to hold the country code that is emitted by child component
      selectedCountry: '',

      // Default to sorting by date
      selectedSort: 'date',

      // Default to the current year
      selectedYear: new Date().getFullYear(),
    };
  },

  // Fetch the list of countries when the app is loaded
  async created() {
    await this.fetchCountries();
  },

  computed: {
    // Generate years from 1974 to the current year as it is the farthest year that the API supports
    years() {
      const currentYear = new Date().getFullYear();
      const startYear = 1974;
      return Array.from({ length: currentYear - startYear + 1 }, (_, index) => startYear + index);
    },
  },

  methods: {
    // Fetches the list of available countries from the API.
    async fetchCountries() {
      // Reset error and countries before fetching new data
      this.error = '';
      this.countries = [];

      try {
        // API endpoint that returns the dynamic countries list
        const response = await fetch('https://date.nager.at/api/v3/AvailableCountries');

        // Check if the response is not OK (e.g., 404, 500)
        if (!response.ok) throw new Error(`Failed to fetch countries: ${response.status}`);

        // Parse the JSON response
        const data = await response.json();

        // Assign the fetched data to the countries array
        this.countries = data;
      } catch (err) {
        // Handle any errors and display a user friendly message
        this.error = 'Failed to fetch the list of countries. Please try again.';
      }
    },

    // Fetches public holidays for the selected country from the API.
    async fetchHolidays(countryCode) {
      // Reset error and holidays before fetching new data
      this.error = '';
      this.holidays = [];

      try {
        // API endpoint with dynamic country code and static year
        const response = await fetch(`https://date.nager.at/api/v3/PublicHolidays/${this.selectedYear}/${countryCode}`);

        // Check if the response is not OK
        if (!response.ok) throw new Error('Failed to fetch holidays');

        // Parse and store the JSON response
        this.holidays = await response.json();

        // Update the country code variable
        this.selectedCountry = countryCode;
      } catch (err) {
        // Handle any errors and display a user friendly message
        this.error = err.message || 'An unexpected error occurred.';
      }
    },
    
    // Calculate the timezone offset in hours
    getOffset() {
      // Returns minutes offset from UTC
      const offsetMinutes = new Date().getTimezoneOffset();

      // Convert to hours and invert sign
      return offsetMinutes / -60;
    },

    async checkIfTodayIsPublicHoliday(countryCode) {
      // Reset error and holidayToday variable before fetching the response
      this.error = '';
      this.isHolidayToday = null;

      // Get the timezone offset
      const offset = this.getOffset();
      const apiUrl = `https://date.nager.at/api/v3/IsTodayPublicHoliday/${countryCode}?offset=${offset}`;

      try {
        // Make a request to the API
        const response = await fetch(apiUrl, { method: 'GET' });

        if (response.status === 200) {
          // Today is a public holiday
          this.isHolidayToday = true; 
        } else if (response.status === 204) {
          // Today is not a public holiday
          this.isHolidayToday = false;
        } else {
          throw new Error('Failed to determine if today is a public holiday');
        }
      } catch (err) {
        console.log(err);
        this.error = 'Failed to determine if today is a public holiday.';

        //Reset on error
        this.isHolidayToday = null;
      }
    },

    async handleCountrySelected(countryCode) {
      // Fetch both holiday list and today's public holiday status
      await this.fetchHolidays(countryCode);
      await this.checkIfTodayIsPublicHoliday(countryCode);
    },

    sortHolidays() {
      // Sorting logic
      if (this.selectedSort === 'date') {
        this.holidays.sort((a, b) => new Date(a.date) - new Date(b.date));
      } else {
        this.holidays.sort((a, b) => a.name.localeCompare(b.name));
      }
    },
  },
};
</script>

<style scoped>
/* The outer container that takes full height and gives the background color */
.app-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  min-width: 100vw;
  background-color: #007066;
}

/* The main content */
.main-content {
  width: 95vw;
  background-color: #ffffff;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); 
  max-width: 100%;
  box-sizing: border-box;
}

@media (min-width: 768px){
  .main-content {
    width: 75vw;
    padding: 45px;
  }
}

@media (min-width: 1200px) {
  .main-content {
    width: 45vw;
    padding: 60px;
  }
}
</style>