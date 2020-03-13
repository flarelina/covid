<template>
  <!--MODAL-->
  <q-dialog transition-show="slide-up"
            transition-hide="slide-down"
            :maximized="$q.platform.is.mobile"
            v-model="isShowModal">
    <q-card class="dialog"
            :style="!$q.platform.is.mobile ? 'min-width: 60%; max-width: 80vw;' : ''">
      <!--TITLE-->
      <q-card-section class="title">
        <span class="text-h6">COVID-19 CASES</span>
        <q-btn icon="close" class="float-right shadow-0" size="sm" flat @click="HideModal"/>
      </q-card-section>
      <q-separator />

      <!--CONTENT-->
      <q-card-section :style="`max-height: ${$q.platform.is.mobile ? '80vh' : '70vh'}`" class="scroll">
        <!--CHART-->
        <div id="chart">
          <apex-chart type="line" height="350" :options="chartOptions" :series="series"></apex-chart>
        </div>

        <!--TABLE--->
        <q-table card-class="table text-white"
                 flat
                 :data="history"
                 :columns="$options.columns"
                 row-key="name">
          <template v-slot:body="props">
            <q-tr>
              <q-td key="date">
                {{ props.row.date }}
              </q-td>
              <q-td key="confirmed" class="text-grey">
                {{ props.row.confirmed }}
              </q-td>
              <q-td key="recovered" class="text-green">
                {{ props.row.recovered }}
              </q-td>
              <q-td key="deaths" style="color: #ff9800">
                {{ props.row.deaths }}
              </q-td>
            </q-tr>
          </template>
        </q-table>
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script>
  import ApexChart from 'vue-apexcharts'

  export default {
    name: "CountryModal.vue",
    components: {
      ApexChart
    },
    data: () => ({
      isShowModal: false,
      country: '',
      history: [
        {
          date: '1/1/2020',
          confirmed: 12,
          recovered: 34,
          deaths: 43,
        }
      ],
      series: [
        {
          name: 'Confirmed',
          data: []
        },
        {
          name: 'Recovered',
          data: []
        },
        {
          name: 'Deaths',
          data: []
        }
      ],
      chartOptions: {
        chart: {
          height: 350,
          type: 'line',
        },
        stroke: {
          width: 7,
          curve: 'smooth'
        },
        xaxis: {
          type: 'Date',
          categories: [],
        },
        title: {
          text: '',
          align: 'left',
          style: {
            fontSize: "16px",
            color: '#666'
          }
        },
        fill: {
          type: 'gradient',
          gradient: {
            shade: 'dark',
            gradientToColors: [ '#FDD835'],
            shadeIntensity: 1,
            type: 'horizontal',
            opacityFrom: 1,
            opacityTo: 1,
            stops: [0, 100, 100, 100]
          },
        },
        markers: {
          size: 4,
          colors: ["#FFA41B"],
          strokeColors: "#fff",
          strokeWidth: 2,
          hover: {
            size: 7,
          }
        },
        yaxis: {
          title: {
            text: 'Cases',
          },
        }
      }
    }),
    methods: {
      FormatNumber(num){
        const num_str  = String(num).split(".");
        const real_num = num_str[0].replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1,');
        const dec_num  = num_str[1] ? num_str[1] : '';
        return [real_num, dec_num].join('')
      },
      ShowModal(data) {
        this.country = data.country;

        // Table Data
        this.history = data.confirmed.map((c, i) => {
          return {
            date      : c.date,
            confirmed : this.FormatNumber(data.confirmed[i].value),
            recovered : this.FormatNumber(data.recovered[i].value),
            deaths    : this.FormatNumber(data.deaths[i].value),
          }
        });

        // Chart Data
        this.chartOptions.title.text = data.country;

        this.chartOptions.xaxis.categories = data.confirmed.map(c => c.date).reverse();
        this.series[0].data = data.confirmed.map((d, i) => data.confirmed[i].value).reverse();
        this.series[1].data = data.confirmed.map((d, i) => data.recovered[i].value).reverse();
        this.series[2].data = data.confirmed.map((d, i) => data.deaths[i].value).reverse();


        this.isShowModal = true;
      },
      HideModal() {
        this.isShowModal = false;
      }
    },
    columns: [
      { name: 'date'      , label: 'Date' , field: 'date'      , sortable: true},
      { name: 'confirmed' , label: 'Confirmed'    , field: 'confirmed' , sortable: true},
      { name: 'recovered' , label: 'Recovered'    , field: 'recovered' , sortable: true},
      { name: 'deaths'    , label: 'Deaths'    , field: 'deaths'     , sortable: true},
    ]
  }
</script>

<style lang="scss" scoped>
.dialog {
   background-color: $primary;

  .title {
    background-color: $primary;
    color: white;
  }

  .table {
    background-color: rgb(38, 43, 73)
  }
}
</style>
