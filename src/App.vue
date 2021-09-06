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
import AppBar from "./components/AppBar";
import SearchPanel from "./components/SearchPanel";
import Queue from "./components/Queue";
import Footer from "./components/Footer";

import {
  SERVER_URL,
  X_API_KEY,
  TIMEOUT_SEC,
  SEARCH_ENDPOINT,
} from "./config.js";

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
    onDoSearch: function(externalId, containerName) {
      this.searchParam.externalId = externalId;
      this.searchParam.containerName = containerName;
      console.log(
        `onDoSearch: externalId=${externalId}, containerName=${containerName}`
      );

      if (this.searchParam.extPageNumber === 1) {
        this.getDataFromApi(); //if currently in the first page, trigger the search here.
      } else {
        this.searchParam.extPageNumber = 1; //new search, set the page to 1, let datatable trigger the search through onDoPage.
      }
    },

    onDoPage: function(options) {
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

      let url = this.buildUrl();

      this.remoteApiCall(url)
        .then((data) => {
          this.searchResult.records = data.content;
          this.searchResult.totalRecords = data.totalElements;
          this.searchResult.errorMessage = "";
          this.searchResult.loading = false;
        })
        .catch((err) => {
          this.searchResult.loading = false;
          console.error(`${err}`);
          this.searchResult.errorMessage = err.message;
        });
    },

    buildUrl() {
      //example:
      //https://localhost:8443/api/record?page=1&size=10&sort=modified&sort=string&externalId=ext&containerName=con&fileName=fn&markedForDeletion=true
      let url = `${this.appConfig.serverURL}${
        this.appConfig.searchEndpoint
      }?page=${this.searchParam.page - 1}&size=${
        this.searchParam.itemsPerPage
      }`;
      if (
        this.searchParam.sortBy.length === 1 &&
        this.searchParam.sortDesc.length === 1
      ) {
        let sortBy =
          this.searchParam.sortBy[0] === "dmsId"
            ? "id"
            : this.searchParam.sortBy[0];
        url += `&sort=${sortBy}${
          this.searchParam.sortDesc[0] ? "%2CDESC" : ""
        }`;
      }
      if (this.searchParam.externalId) {
        url += `&externalId=${this.searchParam.externalId}`;
      }
      if (this.searchParam.containerName) {
        url += `&containerName=${this.searchParam.containerName}`;
      }

      console.log(`buildUrl: ${url}`);
      return url;
    },

    async remoteApiCall(url) {
      const fetchPro = fetch(url, {
        method: "GET",
        headers: {
          accept: "application/json",
          "X-API-KEY": this.appConfig.x_api_key,
        },
      });

      const res = await Promise.race([
        fetchPro,
        this.timeout(this.appConfig.timeout_sec),
      ]);
      const data = await res.json();

      if (!res.ok) {
        throw new Error(`${data.message} (${res.status})`);
      }
      return data;
    },

    timeout(s) {
      return new Promise(function(_, reject) {
        setTimeout(function() {
          reject(new Error(`Timeout after ${s} seconds.`));
        }, s * 1000);
      });
    },

    fakeApiCall() {
      return new Promise((resolve /*, reject*/) => {
        const {
          sortBy,
          sortDesc,
          page,
          itemsPerPage,
          externalId,
          containerName,
        } = this.searchParam;

        console.log(
          `getData: sortBy(${sortBy[0]}), desc(${sortDesc[0]}), page(${page}), itemsPerPage(${itemsPerPage}), extID(${externalId}), containerName(${containerName})`
        );

        let result = this.getFakeSearchResult();
        let content = result.content;
        const totalElements = result.totalElements;

        if (sortBy.length === 1 && sortDesc.length === 1) {
          content = content.sort((a, b) => {
            const sortA = a[sortBy[0]];
            const sortB = b[sortBy[0]];

            if (sortDesc[0]) {
              if (sortA < sortB) return 1;
              if (sortA > sortB) return -1;
              return 0;
            } else {
              if (sortA < sortB) return -1;
              if (sortA > sortB) return 1;
              return 0;
            }
          });
        }

        if (itemsPerPage > 0) {
          content = content.slice(
            (page - 1) * itemsPerPage,
            page * itemsPerPage
          );
        }

        setTimeout(() => {
          resolve({
            content,
            totalElements,
          });
        }, 1000);
      });
    },

    getFakeSearchResult() {
      return {
        content: [
          {
            externalId: "FT-PT5251331",
            fileName:
              "20210825_01661U_FT-PT5251331_NAME-Patient_Request_For_Clinical_Report_Fillable_1__Google_Docs_2.pdf",
            fileType: "pdf",
            containerName: "Default",
            md5sum: "0b964bf03a298f59fad09972699627e8",
            dmsId: "DMS202108256138685",
            created: "2021-08-25T16:46:33.760978Z",
            modified: "2021-08-25T16:46:33.760978Z",
            resourcePath: "/api/resource/DMS202108256138685",
          },
          {
            externalId: "FT-PT5251331",
            fileName: "20210825_01660U_FT-PT5251331_NAME-image0.jpeg",
            fileType: "jpeg",
            containerName: "Default",
            md5sum: "2eb4e97330a58c598e7087aa182b9ec4",
            dmsId: "DMS202108256138684",
            created: "2021-08-25T16:46:24.358114Z",
            modified: "2021-08-25T16:46:24.358114Z",
            resourcePath: "/api/resource/DMS202108256138684",
          },
          {
            externalId: "FH-TS3119580",
            fileName:
              "Filtered_Variants_FH-TS3119580_ACT21H2104740_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "7c4b95fe2ac0acb8b584a9a9722e1a24",
            dmsId: "DMS202108256138683",
            created: "2021-08-25T16:45:36.045461Z",
            modified: "2021-08-25T16:45:36.045461Z",
            resourcePath: "/api/resource/DMS202108256138683",
          },
          {
            externalId: "FT-PT5251331",
            fileName: "20210825_01659U_FT-PT5251331_NAME-image0.jpeg",
            fileType: "jpeg",
            containerName: "Default",
            md5sum: "2eb4e97330a58c598e7087aa182b9ec4",
            dmsId: "DMS202108256138682",
            created: "2021-08-25T16:45:31.143455Z",
            modified: "2021-08-25T16:45:31.143455Z",
            resourcePath: "/api/resource/DMS202108256138682",
          },
          {
            externalId: "FT-TS7822879",
            fileName:
              "Filtered_Variants_FT-TS7822879_ACT21H2192936_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "2bf44262da75502214f57b2f43bbe9ba",
            dmsId: "DMS202108256138681",
            created: "2021-08-25T16:42:02.742474Z",
            modified: "2021-08-25T16:42:02.742474Z",
            resourcePath: "/api/resource/DMS202108256138681",
          },
          {
            externalId: "FT-TS7822866",
            fileName:
              "Filtered_Variants_FT-TS7822866_ACT21H2193733_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "3194796d1576d54de898f2a8eaf5ce2f",
            dmsId: "DMS202108256138680",
            created: "2021-08-25T16:42:02.703225Z",
            modified: "2021-08-25T16:42:02.703225Z",
            resourcePath: "/api/resource/DMS202108256138680",
          },
          {
            externalId: "FT-TS7821426",
            fileName:
              "Filtered_Variants_FT-TS7821426_ACT21H2190148_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "1bff3abf725be09f6835eb36b1f86339",
            dmsId: "DMS202108256138678",
            created: "2021-08-25T16:42:01.733977Z",
            modified: "2021-08-25T16:42:01.733977Z",
            resourcePath: "/api/resource/DMS202108256138678",
          },
          {
            externalId: "FT-TS7821644",
            fileName:
              "Filtered_Variants_FT-TS7821644_ACT21H2190110_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "6ad159d04da0b2d13e3bc9278f96388f",
            dmsId: "DMS202108256138679",
            created: "2021-08-25T16:42:01.733373Z",
            modified: "2021-08-25T16:42:01.733373Z",
            resourcePath: "/api/resource/DMS202108256138679",
          },
          {
            externalId: "FT-TS7821409",
            fileName:
              "Filtered_Variants_FT-TS7821409_ACT21H2188740_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "20a16eba03e51586611420129d983ad9",
            dmsId: "DMS202108256138676",
            created: "2021-08-25T16:42:00.655061Z",
            modified: "2021-08-25T16:42:00.655061Z",
            resourcePath: "/api/resource/DMS202108256138676",
          },
          {
            externalId: "FT-TS7821415",
            fileName:
              "Filtered_Variants_FT-TS7821415_ACT21H2188738_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "e4146df9bd4eb22d55e97aee54390b45",
            dmsId: "DMS202108256138677",
            created: "2021-08-25T16:42:00.648673Z",
            modified: "2021-08-25T16:42:00.648673Z",
            resourcePath: "/api/resource/DMS202108256138677",
          },
          {
            externalId: "FH-TS3122151",
            fileName:
              "Filtered_Variants_FH-TS3122151_ACT21H2183067_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "048ac01dcf0fbe72f3c2a468baa23adf",
            dmsId: "DMS202108256138675",
            created: "2021-08-25T16:41:59.655928Z",
            modified: "2021-08-25T16:41:59.655928Z",
            resourcePath: "/api/resource/DMS202108256138675",
          },
          {
            externalId: "FH-TS3119614",
            fileName:
              "Filtered_Variants_FH-TS3119614_ACT21H2104635_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "40f83cfeb30b0dcb8a596fd70daeaaba",
            dmsId: "DMS202108256138674",
            created: "2021-08-25T16:41:59.651787Z",
            modified: "2021-08-25T16:41:59.651787Z",
            resourcePath: "/api/resource/DMS202108256138674",
          },
          {
            externalId: "FT-PT5159059",
            fileName: "20210825_01658U_FT-PT5159059_NAME-Natera_4936333.png",
            fileType: "png",
            containerName: "Default",
            md5sum: "13cb5f1e8722ef240d1dd5c2c225f177",
            dmsId: "DMS202108256138673",
            created: "2021-08-25T16:39:03.360468Z",
            modified: "2021-08-25T16:39:03.360468Z",
            resourcePath: "/api/resource/DMS202108256138673",
          },
          {
            externalId: "FT-PT4885561",
            fileName: "Isaacs,Donald(4892454)_FT-TS7247846AA.tsv",
            fileType: "tsv",
            containerName: "Reports",
            md5sum: "a21137a32fcfbd264a744986b516d604",
            dmsId: "DMS202108256138672",
            created: "2021-08-25T16:38:10.707194Z",
            modified: "2021-08-25T16:38:10.707194Z",
            resourcePath: "/api/resource/DMS202108256138672",
          },
          {
            externalId: "FT-TS7826983",
            fileName:
              "Filtered_Variants_FT-TS7826983_ACT21H2221524_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "51696fc30efca5689508c6fff8319ada",
            dmsId: "DMS202108256138671",
            created: "2021-08-25T16:38:10.569243Z",
            modified: "2021-08-25T16:38:10.569243Z",
            resourcePath: "/api/resource/DMS202108256138671",
          },
          {
            externalId: "FT-TS7826269",
            fileName:
              "Filtered_Variants_FT-TS7826269_ACT21H2218164_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "d6136223fb1ffbf70ae522899a929424",
            dmsId: "DMS202108256138670",
            created: "2021-08-25T16:38:10.524363Z",
            modified: "2021-08-25T16:38:10.524363Z",
            resourcePath: "/api/resource/DMS202108256138670",
          },
          {
            externalId: "FT-TS7825285",
            fileName:
              "Filtered_Variants_FT-TS7825285_ACT21H2210728_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "59761f227e5ee0ef0e6dbab206c69b6e",
            dmsId: "DMS202108256138669",
            created: "2021-08-25T16:38:09.609158Z",
            modified: "2021-08-25T16:38:09.609158Z",
            resourcePath: "/api/resource/DMS202108256138669",
          },
          {
            externalId: "FT-TS7825164",
            fileName:
              "Filtered_Variants_FT-TS7825164_ACT21H2205729_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "679bfe90b9ae962fab5bfd53931ba70c",
            dmsId: "DMS202108256138668",
            created: "2021-08-25T16:38:09.544436Z",
            modified: "2021-08-25T16:38:09.544436Z",
            resourcePath: "/api/resource/DMS202108256138668",
          },
          {
            externalId: "FT-TS7825163",
            fileName:
              "Filtered_Variants_FT-TS7825163_ACT21H2207358_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "232d414af267b7966f75a3f460d2f9c0",
            dmsId: "DMS202108256138667",
            created: "2021-08-25T16:38:08.582811Z",
            modified: "2021-08-25T16:38:08.582811Z",
            resourcePath: "/api/resource/DMS202108256138667",
          },
          {
            externalId: "FT-TS7825162",
            fileName:
              "Filtered_Variants_FT-TS7825162_ACT21H2205806_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "a137dadf19e5256d83999a3dc2ce5480",
            dmsId: "DMS202108256138666",
            created: "2021-08-25T16:38:08.582516Z",
            modified: "2021-08-25T16:38:08.582516Z",
            resourcePath: "/api/resource/DMS202108256138666",
          },
          {
            externalId: "FT-TS7825161",
            fileName:
              "Filtered_Variants_FT-TS7825161_ACT21H2207373_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "53fbca5deaa92c4d80a4899afc23a076",
            dmsId: "DMS202108256138665",
            created: "2021-08-25T16:38:07.621989Z",
            modified: "2021-08-25T16:38:07.621989Z",
            resourcePath: "/api/resource/DMS202108256138665",
          },
          {
            externalId: "FT-TS7825154",
            fileName:
              "Filtered_Variants_FT-TS7825154_ACT21H2205719_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "98823f4b7f38cb177f95601895ffe9bc",
            dmsId: "DMS202108256138664",
            created: "2021-08-25T16:38:07.609775Z",
            modified: "2021-08-25T16:38:07.609775Z",
            resourcePath: "/api/resource/DMS202108256138664",
          },
          {
            externalId: "FT-TS7825153",
            fileName:
              "Filtered_Variants_FT-TS7825153_ACT21H2205778_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "542dde68be2d02d3783f0d0b6d6a6950",
            dmsId: "DMS202108256138663",
            created: "2021-08-25T16:38:06.668552Z",
            modified: "2021-08-25T16:38:06.668552Z",
            resourcePath: "/api/resource/DMS202108256138663",
          },
          {
            externalId: "FT-TS7825152",
            fileName:
              "Filtered_Variants_FT-TS7825152_ACT21H2205735_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "517e3f700e408f5a70f94ff32bb6f2f5",
            dmsId: "DMS202108256138662",
            created: "2021-08-25T16:38:06.641439Z",
            modified: "2021-08-25T16:38:06.641439Z",
            resourcePath: "/api/resource/DMS202108256138662",
          },
          {
            externalId: "FT-TS7825151",
            fileName:
              "Filtered_Variants_FT-TS7825151_ACT21H2207359_multisource.db",
            fileType: "db",
            containerName: "NGSAnalysis",
            md5sum: "8bfc101108459805f7951a58399c4677",
            dmsId: "DMS202108256138661",
            created: "2021-08-25T16:38:05.715360Z",
            modified: "2021-08-25T16:38:05.715360Z",
            resourcePath: "/api/resource/DMS202108256138661",
          },
        ],
        pageable: {
          sort: {
            unsorted: false,
            sorted: true,
            empty: false,
          },
          offset: 0,
          pageNumber: 0,
          pageSize: 25,
          paged: true,
          unpaged: false,
        },
        last: false,
        totalPages: 1346194,
        totalElements: 25,
        size: 25,
        number: 0,
        sort: {
          unsorted: false,
          sorted: true,
          empty: false,
        },
        first: true,
        numberOfElements: 25,
        empty: false,
      };
    },
  },

  mounted() {
    console.log("app started.");
    console.log(`server: ${this.appConfig.serverURL}`);

    //this.getDataFromApi();
  },
};
</script>
