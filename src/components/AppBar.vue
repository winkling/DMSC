<template>
  <v-app-bar app class="orange lighten-3">
    <v-col>
      <v-img
        alt="Logo"
        class="shrink pl-0"
        contain
        :src="require('../assets/fulgent_logo.svg')"
        transition="scale-transition"
        width="100"
      />
    </v-col>
    <v-col align-self="center" class="text-center text-h6 black--text"
      >DMSC</v-col
    >
    <v-col align-self="end" class="text-right text-subtitle-1 black--text"
      >v{{ version }}</v-col
    >
  </v-app-bar>
</template>

<script>
export default {
  name: "AppBar",
  props: {
    version: String,
  },
};
</script>
