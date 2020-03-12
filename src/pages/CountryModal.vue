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
        <span class="text-h6">Country Here</span>
        <q-btn icon="close" class="float-right shadow-0" size="sm" flat @click="HideModal"/>
      </q-card-section>
      <q-separator />

      <!--CONTENT-->
      <q-card-section :style="`max-height: ${$q.platform.is.mobile ? '80vh' : '70vh'}`" class="scroll">
        luhhh
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script>
  export default {
    name: "CountryModal.vue",
    data: () => ({
      isShowModal: false
    }),
    methods: {
      ShowModal(data) {
        this.isShowModal = true;
      },
      HideModal() {
        this.isShowModal = false;
      }
    }
  }
</script>

<style lang="scss" scoped>
.dialog {
   background-color: $primary;

  .title {
    background-color: $primary;
    color: white;
  }
}
</style>
