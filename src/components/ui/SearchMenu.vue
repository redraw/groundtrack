<template>
  <div>
    <v-autocomplete
      ref="satellites"
      v-model="tle"
      :loading="loading"
      :items="satellites"
      :search-input.sync="query"
      :filter="filter"
      auto-select-first
      hide-details
      hide-no-data
      solo-inverted
      persistent-hint
      dense
      flat
      label="Satellite..."
      @input="$refs.satellites.blur()"
    >
      <template #item="{ item }">
        {{ item.text }}&nbsp;<small class="grey--text">{{ getCatalogNumber(item.value) }}</small>
      </template>
    </v-autocomplete>
    <div class="d-flex flex-column my-4">
      <v-tooltip right>
        <template #activator="{ on, attrs }">
          <v-btn
            class="my-1"
            fab
            x-small
            v-bind="attrs"
            v-on="on"
            @click="setTerminator(!config.terminator)"
          >
            <v-icon :color="config.terminator ? 'primary': ''">
              mdi-weather-sunny
            </v-icon>
          </v-btn>
        </template>
        <span>Solar terminator</span>
      </v-tooltip>
      <v-tooltip right>
        <template #activator="{ on, attrs }">
          <v-btn
            class="my-1"
            fab
            x-small
            v-bind="attrs"
            v-on="on"
            @click="setFollow(!config.follow)"
          >
            <v-icon :color="config.follow ? 'primary': ''">
              mdi-image-filter-center-focus
            </v-icon>
          </v-btn>
        </template>
        <span>Follow satellite</span>
      </v-tooltip>
      <v-tooltip right>
        <template #activator="{ on, attrs }">
          <v-btn
            class="d-none d-sm-flex my-1"
            fab
            x-small
            v-bind="attrs"
            v-on="on"
            @click="setTelemetry(!config.telemetry)"
          >
            <v-icon :color="config.telemetry ? 'primary': ''">
              mdi-satellite-uplink
            </v-icon>
          </v-btn>
        </template>
        <span>Show telemetry panel</span>
      </v-tooltip>
    </div>
  </div>
</template>

<script>
import { mapState, mapMutations } from "vuex";
import { getSatelliteName, getCatalogNumber } from "tle.js"

const debounce = require("lodash/debounce");

export default {
  data() {
    return {
      query: "",
    };
  },

  computed: {
    tle: {
      get() {
        return this.$store.state.tle;
      },
      set(value) {
        if (value) {
          this.$store.commit("updateSat", value);
        }
      },
    },
    satellites() {
      return this.tles.map(tle => {
        return {text: getSatelliteName(tle), value: tle}
      })
    },
    ...mapState([
      "loading", 
      "config", 
      "tles"
    ]),
  },

  watch: {
    query: debounce(async function () {
      if (this.query?.length > 2 && this.$refs.satellites.filteredItems.length === 0) {
        const _type = Array.from(this.query).some(isNaN) ? "NAME" : "CATNR"
        this.$store.dispatch("fetchTLEs", { [_type]: this.query });
      }
    }, 500)
  },

  methods: {
    filter (item, queryText, itemText) {
      const query = queryText.toLowerCase()
      const a = itemText.toLowerCase().indexOf(query) > -1
      const b = getCatalogNumber(item.value).toString().indexOf(query) > -1
      return a || b
    },
    getSatelliteName,
    getCatalogNumber,
    ...mapMutations([
      "updateSat",
      "setTerminator",
      "setFollow",
      "setTelemetry"
    ])
  }
};
</script>
