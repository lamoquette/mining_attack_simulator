<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >

      <h1>
      Mining Simulation
      </h1>
      <v-spacer></v-spacer>

      <v-btn
        href="https://github.com/lamoquette/mining_attack_simulator"
        target="_blank"
        text
      >
        <span class="mr-2">Github Project</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-content>
      <HelloWorld/>
    </v-content>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld';

export default {
  name: 'App',

  components: {
    HelloWorld,
  },

  data: () => ({
    //
  }),
};
</script>
