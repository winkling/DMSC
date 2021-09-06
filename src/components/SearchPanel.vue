<template>
  <v-card class="pt-3 pb-2 mb-1 grey lighten-5">
    <v-form ref="form" v-model="valid" lazy-validation>
      <v-container>
        <v-row align="center">
          <v-col cols="12" md="4">
            <v-text-field
              v-model="externalId"
              label="External ID"
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="4">
            <v-text-field
              v-model="containerName"
              label="Container Name"
            ></v-text-field>
          </v-col>

          <v-col cols="12" md="4">
            <v-btn
              :disabled="!valid"
              color="success"
              class="mr-4"
              @click="validate"
            >
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
    containerName: "",
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
