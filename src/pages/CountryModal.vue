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
        <span class="text-h6">{{country}}</span>
        <q-btn icon="close" class="float-right shadow-0" size="sm" flat @click="HideModal"/>
      </q-card-section>
      <q-separator />

      <!--CONTENT-->
      <q-card-section :style="`max-height: ${$q.platform.is.mobile ? '80vh' : '70vh'}`" class="scroll">
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
              <q-td key="deaths" class="text-red">
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
  export default {
    name: "CountryModal.vue",
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
      ]
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

        this.history = data.confirmed.map((c, i) => {
          return {
            date      : c.date,
            confirmed : this.FormatNumber(data.confirmed[i].value),
            recovered : this.FormatNumber(data.recovered[i].value),
            deaths    : this.FormatNumber(data.deaths[i].value),
          }
        });

        this.isShowModal = true;
      },
      HideModal() {
        this.isShowModal = false;
      }
    },
    columns: [
      { name: 'date'      , label: 'Date'       , field: 'date'      , sortable: true},
      { name: 'confirmed' , label: 'C'  , field: 'confirmed' , sortable: true},
      { name: 'recovered' , label: 'R'  , field: 'recovered' , sortable: true},
      { name: 'deaths'    , label: 'D'     , field: 'deaths'     , sortable: true},
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
