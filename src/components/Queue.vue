<template>
  <div>
    <v-data-table
      :headers="headers"
      :items="records"
      :options.sync="options"
      :server-items-length="totalRecordCount"
      :loading="loading"
      :footer-props="{
        showFirstLastPage: true,
        showCurrentPage: true,
      }"
      class="elevation-1"
    >
      <template v-slot:item.resourcePath="{ item }">
        <v-btn @click="$emit('download-resource', item.resourcePath)" target="_blank" text>
          <v-icon>mdi-open-in-new</v-icon>
        </v-btn>
      </template>

      <template v-slot:item.created="{ item }">
        <span v-text="getFormatedDate(item.created)" />
      </template>

      <template v-slot:item.modified="{ item }">
        <span v-text="getFormatedDate(item.modified)" />
      </template>
    </v-data-table>
  </div>
</template>

<script>
export default {
  name: "Queue",

  props: {
    records: Array,
    loading: Boolean,
    totalRecordCount: Number,

    extPageNumber: Number, //for setting the page number outside.
  },

  data() {
    return {
      options: {},
      headers: [
        {
          text: "DMS ID",
          align: "start",
          sortable: true,
          value: "dmsId",
        },
        { text: "External ID", value: "externalId" },
        { text: "Container Name", value: "containerName" },
        { text: "Created", value: "created" },
        { text: "Modified", value: "modified" },
        { text: "File Type", value: "fileType" },
        { text: "Resource", value: "resourcePath", sortable: false },
      ],
    };
  },

  watch: {
    options: {
      handler() {
        this.$emit("do-page", this.options);
      },
      deep: true,
    },
    extPageNumber: {
      handler() {
        if (this.extPageNumber !== this.options.page) {
          this.options.page = this.extPageNumber;
        }
      },
    },
  },

  methods: {
    getFormatedDate(dateString) {
      return Intl.DateTimeFormat(navigator.language, {
        dateStyle: "short",
        timeStyle: "medium",
      }).format(Date.parse(dateString));
    },
  },
};
</script>
