<template>
  <v-footer padless color="#ffffb0" :class="{ error: errorMessage !== '' }">
    <v-col class="text-left" cols="3"> <strong>Server:</strong> {{ serverURL }} </v-col>
    <v-col class="text-center" cols="6"> <strong v-if="errorMessage !== ''">Error:</strong> {{ errorMessage }} </v-col>
    <v-col class="text-right" cols="3"> <strong>Total records:</strong> {{ totalRecords }} </v-col>
  </v-footer>
</template>

<script>
export default {
  name: "Footer",
  props: {
    serverURL: String,
    totalRecords: Number,
    errorMessage: String,
  },
};
</script>
