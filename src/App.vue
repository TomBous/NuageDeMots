<template>
  <v-app id="app">
    <v-app-bar
      app
      color="blue-grey darken-4"
      elevation="1"
      dark
      width="100vw"
    >
    <v-app-bar-title width="320">Générateur de nuages de mots</v-app-bar-title>
      <!-- <v-avatar
        :color="$vuetify.breakpoint.smAndDown ? 'grey darken-1' : 'transparent'"
        size="32"
      ></v-avatar> -->

      <v-tabs
        centered
        color="cyan lighten-1"
      >
        <v-tab
          v-for="link in links"
          :key="link"
        >
          {{ link }}
        </v-tab>
      </v-tabs>

      <!-- <v-avatar
        class="hidden-sm-and-down"
        color="grey darken-1 shrink"
        size="32"
      ></v-avatar> -->
    </v-app-bar>

    <v-main class="grey lighten-5">
      <WordCloud />
    </v-main>
  </v-app>
</template>
<script>
import WordCloud from './components/WordCloud.vue'
  export default {
    name: "App",
    components: {
      WordCloud,
    },
    data: () => ({
      links: [
        'Présentation',
        'Générateur',
      ],
    }),
  }
</script>

<style scoped>
#app {
  width: 100vw;
}
</style>