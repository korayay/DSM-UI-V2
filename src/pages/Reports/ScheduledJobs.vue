<template>
  <v-app id="inspire" :style="{ background: $vuetify.theme.themes.dark.background }">
    <SideBar />

      <NavBar />

    <v-card color="grey">
      <v-card-title>
        Scheduled Jobs
        <v-spacer></v-spacer>
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-btn class="ma-1 white--text" color="primary" :loading="loading2" :disabled="loading2"
              outlined @click="loader = 'loading2',ExportExcel()">
              <v-icon color="primary" dark v-bind="attrs" v-on="on">
                mdi-microsoft-excel </v-icon>
            </v-btn>
          </template>
          <span>Export to Excel</span>
        </v-tooltip>
        <v-spacer></v-spacer>
        <v-text-field v-model="search"  append-icon="mdi-magnify" label="Search" single-line hide-details></v-text-field>
      </v-card-title>
      <v-data-table :headers="headers" :items="filterJobs" :loading="loaderTable" :items-per-page="10" :footer-props="{
        'items-per-page-options': [20, 50, 100, 200]
      }" class="elevation-1 table-cursor" :search="search">
        <template v-for="(col, index) in filters" v-slot:[`header.${index}`]="{ header }">
          {{ header.text }}
          <v-menu :key="index" offset-y :close-on-content-click="false">
            <template v-slot:activator="{ on, attrs }">
              <v-btn icon v-bind="attrs" v-on="on" color="teal">
                <v-icon small :color="filters[header.value].length ? 'red' : ''">
                  mdi-filter-variant
                </v-icon>
              </v-btn>
            </template>
            <div style="background-color: white; width: 280px">
              <v-list>
                <v-list-item>
                  <div v-if="filters.hasOwnProperty(header.value)">
                    <v-autocomplete multiple dense auto-select-first clearable chips small-chips color="teal"
                      :items="columnValueList(header.value)" append-icon="mdi-filter" v-model="filters[header.value]"
                      :label="filters[header.value] ? `${header.text}` : ''" hide-details>
                      <template v-slot:selection="{ item, index }">
                        <v-chip small class="caption" v-if="index < 5">
                          <span>
                            {{ item }}
                          </span>
                        </v-chip>
                        <span v-if="index === 5" class="grey--text caption">
                          (+{{ filters[header.value].length - 5 }} others)
                        </span>
                      </template>
                    </v-autocomplete>
                  </div>
                </v-list-item>
              </v-list>
            </div>
          </v-menu>
        </template>
      </v-data-table>
    </v-card>
  </v-app>
</template>

<script>
import SideBar from '@/components/SideBar.vue'
import NavBar from '@/components/NavBar.vue'
import { mapGetters, mapActions } from "vuex";
import { scheduledjobsService } from '@/services/api/scheduledjobs.service';
// import { debounce } from 'debounce';
export default {
  name: 'jobs',
  jobs: [],
  data() {
    return {
      filters: { jobDescription: [], owner: [], hostType: [], hostName: [], repeatTime: [], jobName: [] },
      dialog: false,
      loader: null,
      loading: false,
      loading2: false,
      loading3: false,
      loading4: false,
      loading5: false,
      loading6: false,
      search: '',
      loaderTable: false,
      // options: {},
      jobFetchCount: 1,
      searchedJobs: [],
      headers: [
        { text: 'Job Description', align: 'start', sortable: false, value: 'jobDescription', width: "200px", fixed: true },
        { text: 'Owner', value: 'owner',width: "200px", fixed: true },
        { text: 'Host Type', value: 'hostType', width: "150px", fixed: true },
        { text: 'Host Name', value: 'hostName', width: "150px", fixed: true },
        { text: 'Repeat Time', value: 'repeatTime',width: "200px", fixed: true },
        { text: 'Job Name', value: 'jobName', width: "200px", fixed: true },
      ],
      jobs: [],
      oldJobs: [],
      dialogdetail: false
    }
  },
  computed: {
    ...mapGetters('scheduledjobs', ['getScheduleJobsList']),
  },
  methods: {
    ...mapActions('scheduledjobs', ['setScheduledJobs', 'setJobsSearch']),
    columnValueList(val) {
      return this.jobs.map((d) => d[val]);
    },
    // async GetScheduleJobsList(count) {
    //   let response = await this.setScheduledJobs(count);
    //     this.jobs = this.jobs.concat(response);
    //     this.oldJobs = this.jobs;
    // },
    async GetScheduledJobsList() {
      this.loaderTable = true;
      let count = 1;
      let temp = [];
      let response = await this.setScheduledJobs(count);
      // this.oldJobs = this.jobs;
      while (response.length > 0) {
        temp = temp.concat(response);
        count++;
        response = await this.setScheduledJobs(count);
      }
      this.loaderTable = false;
      this.jobs = temp;
      temp = [];

    },
    ExportExcel(){
      if(this.search.length > 0){
        scheduledjobsService.getExportSearchList(this.search);
    }else{
      scheduledjobsService.getExportList();
    }
    }
    // debounceInput: debounce(async function (e) {
    //   this.loaderTable = true;
    //   this.jobs = [];
    //   if (e.length > 0) {
    //     this.jobs = await this.setJobsSearch(e);
    //   } else {
    //     this.jobs = this.oldJobs;
    //   }
    //   this.loaderTable = false;
    // }, 1000)
  },
  created() {
    this.GetScheduledJobsList();
  },
  watch: {
    // options: {
    //   handler() {
    //     if (this.search.length == 0) {
    //       this.GetScheduleJobsList(this.jobFetchCount)
    //       this.jobFetchCount++;
    //     }
    //   },
    //     deep: true,
    //   },
    loader() {
      const l = this.loader
      this[l] = !this[l]

      setTimeout(() => (this[l] = false), 2000)

      this.loader = null
    },
  },
  computed: {
    filterJobs() {
      return this.jobs.filter((d) => {
        return Object.keys(this.filters).every((f) => {
          return this.filters[f].length < 1 || this.filters[f].includes(d[f]);
        });
      });
    }
  },
  components: {
    SideBar,
    NavBar,
  },
}
</script>

<style>
.v-btn.withoutupercase {
  text-transform: none !important;
}

.v-btn:not(.v-btn--round).v-size--small {
  min-width: 30px !important;
  padding: 0 5px !important;
}

.custom-loader {
  animation: loader 1s infinite;
  display: flex;
}

@-moz-keyframes loader {
  from {
    transform: rotate(0);
  }

  to {
    transform: rotate(360deg);
  }
}

@-webkit-keyframes loader {
  from {
    transform: rotate(0);
  }

  to {
    transform: rotate(360deg);
  }
}

@-o-keyframes loader {
  from {
    transform: rotate(0);
  }

  to {
    transform: rotate(360deg);
  }
}

@keyframes loader {
  from {
    transform: rotate(0);
  }

  to {
    transform: rotate(360deg);
  }
}
</style>