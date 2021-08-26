<template>
  <v-app-bar app color="orange lighten-2">
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
    <v-col align-self="center" class="text-center text-h6 grey--text text--lighten-5">DMSC</v-col>
    <v-col align-self="end" class="text-right text-subtitle-1 grey--text text--lighten-5">v0.1.0</v-col>
  </v-app-bar>
</template>

<script>
export default {
  name: "AppBar",
};
</script>
