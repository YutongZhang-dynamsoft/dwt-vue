<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
    Dynamic Web TWAIN
    </v-app-bar>

    <v-main>
      <DWT />
    </v-main>
  </v-app>
</template>

<script>
import DWT from './components/DWT';

export default {
  name: 'App',

  components: {
    DWT
  },

  data: () => ({
    //
  }),
};
</script>
