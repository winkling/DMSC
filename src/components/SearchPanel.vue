<template>
  <v-card class="pt-3 pb-2 mb-1 grey lighten-5">
    <v-form ref="form" v-model="valid" lazy-validation>
      <v-container>
        <v-row align="center">
          <v-col cols="12" md="4">
            <v-text-field v-model="externalId" label="External ID" :rules="externalIdRules"></v-text-field>
          </v-col>

          <v-col cols="12" md="4">
            <v-text-field v-model="containerName" label="Container Name" :rules="containerNameRules"></v-text-field>
          </v-col>

          <v-col cols="12" md="4">
            <v-btn :disabled="!valid" color="success" class="mr-4" @click="validate">
              Search
            </v-btn>
            <v-btn color="error" class="mr-4" @click="reset">
              Reset
            </v-btn>
          </v-col>
        </v-row>
      </v-container>
    </v-form>
  </v-card>
</template>

<script>
export default {
  name: "SearchPanel",

  data: () => ({
    valid: true,
    externalId: "",
    externalIdRules: [
      (v) => (v ? v.length <= 255 || "External ID must be less than 255 characters" : true),
      (v) => (v ? /^\S+$/.test(v) : true) || "External ID must be valid (regex: ^\\S+$)",
    ],
    containerName: "",
    containerNameRules: [(v) => (v ? v.length <= 255 || "Container Name must be less than 255 characters" : true)],
  }),

  methods: {
    validate() {
      this.$refs.form.validate();
      this.$emit("do-search", this.externalId, this.containerName);
    },
    reset() {
      this.$refs.form.reset();
      this.$emit("do-search", this.externalId, this.containerName);
    },
  },
};
</script>
