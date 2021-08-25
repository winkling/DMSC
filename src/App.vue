<template>
  <v-app>
    <v-main>
      <AppBar />
      <SearchPanel />
      <Queue />
      <Footer />
    </v-main>
  </v-app>
</template>

<script>
import AppBar from "./components/AppBar";
import SearchPanel from "./components/SearchPanel";
import Queue from "./components/Queue";
import Footer from "./components/Footer";

export default {
  name: "App",

  components: {
    AppBar,
    SearchPanel,
    Queue,
    Footer,
  },

  data: () => ({
    //
  }),
};
</script>
