<script setup>
import ProgressSpinner from 'primevue/progressspinner';
import {ref, onMounted} from 'vue';
import ProductService from '@/service/ProductService';
import {useReportStore} from "@/stores";

const store = useReportStore();

const dataviewValue = ref(null);
const layout = ref('grid');
const sortKey = ref(null);
const sortOrder = ref(null);
const sortField = ref(null);
const skeleton = ref([1, 2, 3, 4])

const sortOptions = ref([
  {label: 'Price High to Low', value: '!price'},
  {label: 'Price Low to High', value: 'price'}
]);

import {inject} from "vue";
// import moment from "moment";
moment.locale('es-mx');

const dialogRef = inject('dialogRef');
const modal = dialogRef.modal
const closeDialog = (obj) => {
  dialogRef.value.close(obj);
}

onMounted(() => {
  fetchCatalogos()
  console.log(modal)
});
const formatCurrency = (value) => {
  return value?.toLocaleString('en-US', {style: 'currency', currency: 'USD'});
};
const fetchCatalogos = () => {
  store.setOperaciones(moment().subtract(2, "months").format('YYYY-MM-DD'), moment().add(1, 'day').format('YYYY-MM-DD'));
}

const formatLongDate = (date) => {
  return moment(date).format('MMMM Do YYYY, h:mm:ss a')
}
const formatDate = (date) => {
  return moment(date).fromNow()
}
</script>

<template>
  <template v-if="store.getOperaciones.length">
    <DataView :value="store.getOperaciones" :layout="'list'" :paginator="true" :rows=10 :sortOrder="sortOrder"
              :sortField="sortField">
      <template #list="slotProps">
        <div class="col-12 bg-black-alpha-50/500'">
          <div class="flex flex-row align-items-center px-3 py-0 w-full hoverable" role="button"
               @click="closeDialog(slotProps.data)">
            <div class="flex-1 text-left">
              <div class="font-bold text-l">Ruta {{ slotProps.data.no_ruta }} -
                <strong class="text-primary">{{ formatCurrency(slotProps.data.cobro) }} </strong>
              </div>
              <div class="mb-0">
                <span class="text-sm mr-1">
                {{ formatLongDate(slotProps.data.date) }},
                </span>
                <small class="font-bold">
                  {{(formatDate(slotProps.data.date).toUpperCase()) }}
                </small>
              </div>
            </div>
            <div
                class="flex flex-column justify-content-between w-auto align-items-center md:align-items-end mt-5 md:mt-0">
              <Button text icon="pi pi-chevron-right" size="small" label="ver" class="mb-2"></Button>
            </div>
          </div>
        </div>
      </template>
    </DataView>

  </template>
  <template v-else>
    <DataView :value="skeleton" :layout="'list'">

      <template #list>
        <div class="col-12">
          <div class="flex flex-row align-items-center p-3 w-full">
            <div
                class="flex flex-column sm:flex-row justify-content-between align-items-center xl:align-items-start flex-1 gap-4">
              <div class="flex flex-column align-items-center sm:align-items-start gap-3">
                <Skeleton class="w-6rem border-round h-1rem"/>
                <Skeleton class="w-8rem border-round h-1rem"/>
              </div>
              <div class="flex sm:flex-column align-items-center sm:align-items-end gap-3 sm:gap-2">

                <Skeleton shape="circle" class="w-3rem h-3rem"/>
              </div>
            </div>
          </div>
        </div>
      </template>

    </DataView>
  </template>
</template>

<style scoped lang="scss">
.p-dataview .p-dataview-content {
  background: #ffffff;
  color: #574949;
  border: 0 none;
  padding: 0;
  min-height: 270px;
}
</style>
