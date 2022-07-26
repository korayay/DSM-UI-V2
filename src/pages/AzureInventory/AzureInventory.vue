<template>
  <v-app id="inspire" :style="{background: $vuetify.theme.themes.dark.background}">
    <SideBar />
    <NavBar />

    <v-card color="grey">
      <v-card-title>
        Azure Portal Inventory
        <v-spacer></v-spacer>
        <v-text-field v-model="search" append-icon="mdi-magnify" label="Search" single-line hide-details></v-text-field>
      </v-card-title>
      <v-data-table :headers="headers" :items="filterInventories" :items-per-page="10" :footer-props="{
        'items-per-page-options': [20, 50, 100, 200]
      }" class="elevation-1 table-cursor" :search="search" :custom-filter="customSearch">
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
        <template v-slot:item.details="{item}">
          <v-btn depressed rounded text color="teal" @click="showBindings(item)">
            <v-icon>mdi-eye</v-icon>Show Bindings
          </v-btn>
        </template>
      </v-data-table>
      <v-dialog v-model="dialogdetail">
        <v-card>
          <v-toolbar dark color="primary">
            <v-btn icon dark @click="dialogdetail = false">
              <v-icon>mdi-close</v-icon>
            </v-btn>
            <v-toolbar-title class="flex text-center text-h5">DETAILS</v-toolbar-title>
          </v-toolbar>
          <v-container>
            <template>
              <v-tabs color="primary" vertical>
                <v-tab>Bindings</v-tab>
                <v-tab-item>
                  <v-card color="primary">
                    <v-card-title>
                      Binding Details
                      <v-spacer></v-spacer>
                      <v-text-field v-model="search" append-icon="mdi-magnify" label="Search" single-line hide-details color="black">
                      </v-text-field>
                    </v-card-title>
                    <v-data-table :headers="bindingheaders" :items="bindings" :items-per-page="10" :footer-props="{
                      'items-per-page-options': [5, 10, 20, 50]
                    }" class="elevation-1" :search="search">
                    </v-data-table>
                  </v-card>
                </v-tab-item>
              </v-tabs>

            </template>
          </v-container>
        </v-card>
      </v-dialog>
    </v-card>
  </v-app>
</template>
  
<script>
import SideBar from '@/components/SideBar.vue'
import NavBar from '@/components/NavBar.vue'
import { mapGetters, mapActions } from "vuex";
export default {
  name: 'AzureInventory',
  data() {
    return {
      filters: { siteName: [], defaultHostName: [], outboundIpAddresses: [], resourceGroup: [], subscriptionName:[]},
      dialog: false,
      loader: null,
      loading: false,
      loading2: false,
      loading3: false,
      loading4: false,
      loading5: false,
      loading6: false,
      search: '',
      headers: [
        { text: 'Site Name', align: 'start', sortable: false, value: 'siteName' , width: '200px' },
        { text: 'Bindings', value: 'details', },
        { text: 'Host Name', value: 'defaultHostName' },
        { text: 'Ip Addresses', value: 'outboundIpAddresses' },
        { text: 'Resource Group', value: 'resourceGroup' },
        { text: 'Subs. Name', value: 'subscriptionName'}
      ],
      bindingheaders: [
        { text: 'Bindings', value: 'bindings' },
      ],
      inventories: [],
      bindings: [],
      dialogdetail: false
    }
  },
  computed: {
    ...mapGetters('inventory', ['getInventoryList', 'getSiteBindings']),
  },
  methods: {
    ...mapActions('inventory', ['setInventories', 'setSiteBindings']),
    async showBindings(item) {
      this.details = item
      this.bindings = item.bindings.map(binding => {return {bindings: binding}});
      this.dialogdetail = true
    },
    columnValueList(val) {
      return this.inventories.map((d) => d[val]);
    },
    async GetInventoryList() {
      this.inventories = await this.setSiteBindings();
    },
    customSearch (value, search, item) {
          return Object.values(item).some(v=>v&&v.toString().toLowerCase().includes(search))
      }
  },
  created() {
    this.GetInventoryList();
  },

  watch: {
    loader() {
      const l = this.loader
      this[l] = !this[l]

      setTimeout(() => (this[l] = false), 2000)

      this.loader = null
    },
  },
  computed: {
    filterInventories() {
      return this.inventories.filter((d) => {
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