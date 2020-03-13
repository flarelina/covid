<template>
  <!--CHART-->
  <div id="chart">
    <apex-chart type="line" height="350" :options="chartOptions" :series="series"></apex-chart>
  </div>
</template>

<script>
  import ApexChart from 'vue-apexcharts'

  export default {
    name: "GlobalGraph",
    props: {
      historyByCountry: {type: Array, default: () => []}
    },
    components: {
      ApexChart
    },
    data: () => ({
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
          height: 700,
          type: 'line',
        },
        stroke: {
          width: 5,
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
            color: 'rgb(56, 63, 107)'
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
          size: 0,
          colors: ["#383f6b"],
          strokeColors: "#fff",
          strokeWidth: 0,
          hover: {
            size: 0,
          }
        },
        yaxis: {
          title: {
            text: 'Cases',
          },
        }
      }
    }),
    mounted() {
      const data = this.historyByCountry;

      // generate the dates
      const dates = data[0].confirmed.map(c => c.date);

      const computePerDate = (category) => dates.map((date, i) => {
        return data.map(d => d[category][i].value).reduce((sum, x) => sum + Number(x), 0)
      }).reverse();

      this.chartOptions.xaxis.categories = dates.reverse();
      console.log(this.chartOptions.xaxis.categories);

      this.series[0].data = computePerDate("confirmed");
      this.series[1].data = computePerDate("recovered");
      this.series[2].data = computePerDate("deaths");
    }
  }
</script>

<style scoped>

</style>
