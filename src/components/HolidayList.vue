<template>
    <div>
      <ul class="list-group">
        <li 
          class="list-group-item" 
          v-for="holiday in holidays" 
          :key="holiday.date"
        >
          <strong>{{ holiday.date }}</strong> - {{ holiday.name }}
          <div v-if="holiday.localName">
            <p><strong>Local Name:</strong> {{ holiday.localName }}</p>
          </div>
        </li>
      </ul>
    </div>
</template>
  
<script>
  export default {
    props: {
      holidays: {
        type: Array,
        required: true, // Ensure the parent passes the holidays array
      },
    },
  };
</script>
  
<style scoped>
  .list-group-item {
    margin-bottom: auto;
  }
  
  strong {
    font-size: 1.2em;
  }
  
  p {
    font-size: 0.9em;
    margin-top: 5px;
  }
</style>