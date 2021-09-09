<template>
  <v-app>
    <AppBar :version="appConfig.version" />

    <v-main>
      <SearchPanel @do-search="onDoSearch" />
      <Queue
        :records="searchResult.records"
        :totalRecordCount="searchResult.totalRecords"
        :loading="searchResult.loading"
        :extPageNumber="searchParam.extPageNumber"
        @do-page="onDoPage"
        @download-resource="onDownloadResource"
      />
    </v-main>

    <Footer
      :serverURL="appConfig.serverURL"
      :totalRecords="searchResult.totalRecords"
      :errorMessage="searchResult.errorMessage"
    />
  </v-app>
</template>

<script>
const FileSaver = require("file-saver");
const axios = require("axios");

import AppBar from "./components/AppBar";
import SearchPanel from "./components/SearchPanel";
import Queue from "./components/Queue";
import Footer from "./components/Footer";

import { SERVER_URL, X_API_KEY, TIMEOUT_SEC, SEARCH_ENDPOINT } from "./config.js";

export default {
  name: "App",

  components: {
    AppBar,
    SearchPanel,
    Queue,
    Footer,
  },

  data: () => ({
    appConfig: {
      serverURL: SERVER_URL,
      x_api_key: X_API_KEY,
      searchEndpoint: SEARCH_ENDPOINT,
      timeout_sec: TIMEOUT_SEC,
      version: "0.1.0", //TODO: get version from package.json
    },
    searchParam: {
      externalId: "",
      containerName: "",
      sortBy: "",
      sortDesc: "",
      page: 1,
      itemsPerPage: 10,
      extPageNumber: 1,
    },
    searchResult: {
      records: [],
      totalRecords: 0,
      loading: true,
      errorMessage: "",
    },
  }),

  methods: {
    onDoSearch(externalId, containerName) {
      this.searchParam.externalId = externalId;
      this.searchParam.containerName = containerName;
      console.log(`onDoSearch: externalId=${externalId}, containerName=${containerName}`);

      if (this.searchParam.extPageNumber === 1) {
        this.getDataFromApi(); //if currently in the first page, trigger the search here.
      } else {
        this.searchParam.extPageNumber = 1; //new search, set the page to 1, let datatable trigger the search through onDoPage.
      }
    },

    onDoPage(options) {
      this.searchParam.extPageNumber = options.page;
      this.searchParam.sortBy = options.sortBy;
      this.searchParam.sortDesc = options.sortDesc;
      this.searchParam.page = options.page;
      this.searchParam.itemsPerPage = options.itemsPerPage;

      console.log(
        `onDoPage: sortBy(${options.sortBy[0]}), desc(${options.sortDesc[0]}), page(${options.page}), itemsPerPage(${options.itemsPerPage})`
      );

      this.getDataFromApi();
    },

    getDataFromApi() {
      this.searchResult.loading = true;

      let url = this.buildSearchUrl();

      axios
        .get(url, {
          headers: {
            "X-API-KEY": this.appConfig.x_api_key,
          },
          timeout: this.appConfig.timeout_sec * 1000,
        })
        .then((response) => {
          this.searchResult.records = response.data.content;
          this.searchResult.totalRecords = response.data.totalElements;
          this.searchResult.errorMessage = "";
          this.searchResult.loading = false;
        })
        .catch((err) => {
          console.error(`${err}`);
          this.searchResult.errorMessage = err.message;
          this.searchResult.loading = false;
        });
    },

    buildSearchUrl() {
      //example:
      //https://localhost:8443/api/record?page=1&size=10&sort=modified%2CDESC&externalId=ext&containerName=con&fileName=fn&markedForDeletion=true
      let url = `${this.appConfig.serverURL}${this.appConfig.searchEndpoint}?page=${this.searchParam.page - 1}&size=${
        this.searchParam.itemsPerPage
      }`;
      if (this.searchParam.sortBy.length === 1 && this.searchParam.sortDesc.length === 1) {
        let sortBy = this.searchParam.sortBy[0] === "dmsId" ? "id" : this.searchParam.sortBy[0];
        url += `&sort=${sortBy}${this.searchParam.sortDesc[0] ? "%2CDESC" : ""}`;
      }
      if (this.searchParam.externalId) {
        url += `&externalId=${encodeURIComponent(this.searchParam.externalId)}`;
      }
      if (this.searchParam.containerName) {
        url += `&containerName=${encodeURIComponent(this.searchParam.containerName)}`;
      }

      console.log(`buildUrl: ${url}`);
      return url;
    },

    onDownloadResource(resourcePath) {
      console.log(`onDownloadResource: ${resourcePath}`);

      let url = this.appConfig.serverURL + resourcePath;

      axios
        .get(url, {
          headers: {
            "X-API-KEY": this.appConfig.x_api_key,
          },
          responseType: "blob",
          timeout: this.appConfig.timeout_sec * 1000,
        })
        .then((response) => {
          let filename = this.getFileNameFromHeader(response.headers);
          console.log(`filename: ${filename}`);
          FileSaver.saveAs(new Blob([response.data]), filename);
        })
        .catch((err) => {
          console.error(`${err}`);
          this.searchResult.errorMessage = err.message;
        });
    },

    getFileNameFromHeader(headers) {
      let filename = "";
      let disposition = headers["content-disposition"];
      if (disposition && disposition.indexOf("attachment") !== -1) {
        let filenameRegex = /filename[^;=\n]*=((['"]).*?\2|[^;\n]*)/;
        let matches = filenameRegex.exec(disposition);
        if (matches != null && matches[1]) {
          filename = matches[1].replace(/['"]/g, "");
        }
      }

      return filename;
    },

  },

  mounted() {
    console.log("app started.");
    console.log(`server: ${this.appConfig.serverURL}`);

    //this.getDataFromApi();
  },
};
</script>
