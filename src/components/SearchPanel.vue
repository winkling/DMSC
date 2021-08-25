<template>
  <v-card height="150px">
    <v-form ref="form" v-model="valid" lazy-validation>
      <v-container class="grey lighten-5 mb-6">
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
    },
    reset() {
      this.$refs.form.reset();
    },
  },
};
</script>
