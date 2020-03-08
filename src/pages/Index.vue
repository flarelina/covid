<template>
  <q-page>
    <div class="text-h6 text-center q-pt-lg" v-if="isFetching">
      <q-spinner color="primary" size="3em" :thickness="5"/>
    </div>

    <div class="dashboard row q-col-gutter-lg" v-else>
      <!--FIRST SECTION-->
      <div class="dashboard__section1 col-12 col-md-12">
        <!--TOTAL REPORT-->
        <div :class="`total row ${$q.platform.is.mobile ? '' : 'q-gutter-lg'}`">
          <q-card flat bordered :class="`col-12 col-md text-center ${$q.platform.is.mobile ? 'q-mb-lg' : ''}`"
                  v-for="item in total" :key="item.label">
            <q-card-section>
              <div class="text-grey-7"><b>{{item.label.toUpperCase()}}</b></div>
              <div class="text-h4 q-pt-md" :style="{color: item.color}">{{FormatNumber(item.num)}}</div>
            </q-card-section>
          </q-card>
        </div>

        <!--MAP-->
<!--        <div class="map">
          <q-card flat bordered class="col text-center">
            <q-card-section>
              Gusto ko mapa dito maganda
            </q-card-section>
          </q-card>
        </div>-->
      </div>

      <div class="dashboard__section2 col-12 col-md-12">
        <!--COUNTRIES-->
        <div>
          <q-card flat bordered class="col text-center">
            <q-card-section>
              <q-input stack-label filled v-model="filter.search" label="Search" />

              <!--LIST OF COUNTRIES-->
              <q-scroll-area bordered  class="q-mt-lg" style="height: 450px;">
                <q-list bordered separator>
                  <q-item clickable v-ripple v-for="confirm in FilteredCountries" :key="confirm.key">
                    <q-item-section>{{confirm.country}}</q-item-section>
                    <q-item-section>
                      <span style="color: grey">Confirmed: {{FormatNumber(confirm.num)}}</span>
                      <span style="color: green">Recovered: {{FormatNumber(confirm.recovered)}}</span>
                      <span style="color: red">Deaths: {{FormatNumber(confirm.deaths)}}</span>
                    </q-item-section>
                  </q-item>
                </q-list>
              </q-scroll-area>
            </q-card-section>
          </q-card>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
  import "./IndexStyle.scss"
  import Mapa from "./Mapa"

  export default {
    name: 'PageIndex',
    components: {Mapa},
    data: () => ({
      isFetching: true,
      filter: {
        search: ''
      },
      total: [
        {label: 'Confirmed Cases', color: 'gray'  , num: "0"},
        {label: 'Recovered'      , color: 'green' , num: "0"},
        {label: 'Deaths'         , color: 'red'   , num: "0"},
      ],
      confirmedByCountry: [],
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


        this.confirmedByCountry = confirmedByCountry.sort((a ,b) => Number(b.num) - Number(a.num));

        this.isFetching = false;
      }
    },
    mounted() {
      this.LoadAllData();
    }
  }
</script>
