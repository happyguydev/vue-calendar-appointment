<template>
  <v-app>
    <v-main>
      <CalendarAppointment/>
    </v-main>
  </v-app>
</template>

<script>
import CalendarAppointment from './components/CalendarAppointment';

export default {
  name: 'App',

  components: {
    CalendarAppointment,
  },
};
</script>
