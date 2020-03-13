<template>
  <q-page class="bg-primary">
   <div class="loader fixed-center text-center q-pt-lg" v-if="isFetching">
      <q-spinner-ios class="loader__spinner" size="5em" :thickness="5"/>
      <div class="text-white q-pt-lg">
        Retrieving Data ...
      </div>
    </div>
    <div class="dashboard" v-else>
      <!--TITLE-->
      <div class="dashboard__title">
        COVID-19 CASES
      </div>

      <!--FIRST SECTION-->
      <div class="dashboard__section1">
        <!--TOTAL REPORT-->
        <div  :class="`total row ${$q.platform.is.mobile ? '' : 'q-gutter-lg'}`">
          <q-card flat :class="`total--item col-12 col-md text-center ${$q.platform.is.mobile ? 'q-mb-lg' : 'q-mb-lg'}`"
                  v-for="item in total" :key="item.label">
            <q-card-section>
              <div :style="{color: item.color}">
                <q-icon class="q-pb-xs q-pr-sm" dense name="fiber_manual_record" />
                <b>{{item.label.toUpperCase()}}</b>
              </div>
              <div class="text-h4 q-pt-md">{{FormatNumber(item.num)}}</div>
            </q-card-section>
          </q-card>
        </div>
      </div>

      <div class="dashboard__section2">
        <!--COUNTRIES-->
        <div :class="`row ${$q.platform.is.mobile ? '' : 'q-gutter-lg'}`">
          <q-card flat class="graph col-12 col-md text-center q-mb-lg">
            <q-card-section>
              <global-graph :historyByCountry="historyByCountry"></global-graph>
            </q-card-section>
          </q-card>

          <q-card flat class="table col-12 col-md text-center q-mb-lg">
            <q-card-section>
              <q-input stack-label filled dark v-model="filter.search" label="Search" />

              <!--LIST OF COUNTRIES-->
              <q-scroll-area bordered  class="q-mt-lg" style="height: 450px;">
                <q-list bordered separator>
                  <q-item clickable v-ripple
                          v-for="confirm in FilteredCountries" :key="confirm.key"
                          @click="ShowCountryModal(confirm.key)">
                    <q-item-section>{{confirm.country}}</q-item-section>
                    <q-item-section>
                      <span style="color: #008FFB">Confirmed: {{FormatNumber(confirm.num)}}</span>
                      <span style="color: #15ff15">Recovered: {{FormatNumber(confirm.recovered)}}</span>
                      <span style="color: #ff9800">Deaths: {{FormatNumber(confirm.deaths)}}</span>
                    </q-item-section>
                  </q-item>
                </q-list>
              </q-scroll-area>
            </q-card-section>
          </q-card>
        </div>
      </div>
    </div>

    <!--MODAL-->
    <country-modal ref="countryModalRef"></country-modal>
  </q-page>
</template>

<script>
  import "./IndexStyle.scss"

  import {date} from "quasar"

  import CountryModal from "./CountryModal"
  import GlobalGraph  from "./GlobalGraph"

  export default {
    name: 'PageIndex',
    components: {CountryModal, GlobalGraph},
    data: () => ({
      isFetching: true,
      filter: {
        search: ''
      },
      total: [
        {label: 'Confirmed Cases', color: '#008FFB'  , num: "0"},
        {label: 'Recovered'      , color: '#15ff15' , num: "0"},
        {label: 'Deaths'         , color: '#ff9800'   , num: "0"},
      ],
      confirmedByCountry: [],
      historyByCountry: [],
      allData: []
    }),
    computed: {
      FilteredCountries() {
        return this.confirmedByCountry.filter(confirm => {
          return confirm.country.toLowerCase().includes(this.filter.search.toLowerCase())
        })
      }
    },
    methods: {
      FormatNumber(num){
        const num_str  = String(num).split(".");
        const real_num = num_str[0].replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1,');
        const dec_num  = num_str[1] ? num_str[1] : '';
        return [real_num, dec_num].join('')
      },
      async LoadAllData() {
        this.isFetching = true;

        const allData = await this.axios.get('https://coronavirus-tracker-api.herokuapp.com/all')
          .then(res => res.data)
          .catch(error=>{
            console.log(error)
          });

        // Total Confirmed, Recovered and Deaths
        this.total[0].num = this.FormatNumber(allData.confirmed.latest);
        this.total[1].num = this.FormatNumber(allData.recovered.latest);
        this.total[2].num = this.FormatNumber(allData.deaths.latest);

        // Latest Confirmed Cases Data [{key, country, num}]
        const countryCodes = [...new Set(allData.confirmed.locations.map(l => l.country_code))];

        const confirmedByCountry = countryCodes.map(code => {
          const locations = allData.confirmed.locations;
          const index = locations.findIndex(i => i.country_code === code);

          const getNum = (arr) => arr
            .filter(i => i.country_code === code)
            .map(i => i.latest)
            .reduce((a, v) => a + Number(v) , 0);

          return {
            key       : code,
            country   : locations[index].country,
            num       : getNum(locations),
            recovered : getNum(allData.recovered.locations),
            deaths    : getNum(allData.deaths.locations)
          }
        });

        // Get Data History By Country
        const historyByCountry = countryCodes.map(code => {
          const index = allData.confirmed.locations.findIndex(i => i.country_code === code);

          const getHistory = (arr) => {
            const collectedHistories = {};

            const histories = arr.filter(i => i.country_code === code).map(i => i.history);

            // Initialize Available Dates
            const dates = Object.keys(histories[0]);
            dates.forEach(date => {
              collectedHistories[date] = 0;
            });

            // Summarize data
            histories.forEach(history => {
              dates.forEach(date => {
                collectedHistories[date] += Number(history[date])
              })
            });

            // Convert object to array and sort
            const historyArray = Object.keys(collectedHistories).map(_date => {
              return {
                dateId : date.formatDate(_date, 'YYYYMMDD'),
                date   : date.formatDate(_date, 'MM/DD'),
                value  : collectedHistories[_date],
              }
            });

            return historyArray.sort((a, b) => Number(b.dateId) - Number(a.dateId));
          };

          return {
            key       : code,
            country   : allData.confirmed.locations[index].country,
            confirmed : getHistory(allData.confirmed.locations),
            recovered : getHistory(allData.recovered.locations),
            deaths    : getHistory(allData.deaths.locations)
          }
        });

        this.confirmedByCountry = confirmedByCountry.sort((a ,b) => Number(b.num) - Number(a.num));
        this.historyByCountry   = historyByCountry;

        this.isFetching = false;
      },
      ShowCountryModal(countryCode) {
        const data = this.historyByCountry.filter(h => h.key === countryCode)[0];
        this.$refs.countryModalRef.ShowModal(data)
      }
    },
    mounted() {
      this.LoadAllData();
    }
  }
</script>
