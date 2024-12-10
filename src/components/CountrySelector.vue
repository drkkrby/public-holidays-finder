<template>
    <div>
        <label for="country" class="form-label">Select a Country:</label>
        <select id="country" class="form-select" @change="onCountryChange">
            <option value="" disabled selected>
                {{ countries.length ? "Select a country" : "Loading countries..." }}
            </option>
            <option
                v-for="country in countries"
                :key="country.countryCode"
                :value="country.countryCode"
            >
                {{ country.name }}
            </option>
        </select>   
    </div>
  </template>
  
  <script>
  export default {
    props: {
      countries: {
        type: Array,
        required: true, // Ensure the parent provides the countries array
      },
    },
    methods: {
      onCountryChange(event) {
        const selectedCountry = event.target.value;
        this.$emit('country-selected', selectedCountry);
      },
    },
  };
  </script>