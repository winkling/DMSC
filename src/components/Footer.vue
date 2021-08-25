<template>
  <v-footer padless>
    <v-col class="text-left" cols="6">
      <strong>Server:</strong> {{ serverURL }}
    </v-col>

    <v-col class="text-right" cols="6">
      <strong>Total records:</strong> {{ totalRecords }}
    </v-col>
  </v-footer>
</template>

<script>
export default {
  name: "Footer",

  data() {
    return {
      serverURL: "https://dms.fulgentinternal.com",
      totalRecords: 33654835,
    };
  },
};
</script>
