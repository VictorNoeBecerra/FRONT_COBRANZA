<!-- eslint-disable vue/multi-word-component-names -->
<!-- eslint-disable no-unused-vars -->
<!-- eslint-disable prettier/prettier -->
<!-- eslint-disable prettier/prettier -->
<script setup>
import { onMounted, reactive, ref, watch } from 'vue';
import ProductService from '@/service/ProductService';
import { useLayout } from '@/layout/composables/layout';
import { useToast } from "primevue/usetoast";
import { useDashboardStore } from '@/stores'
import { getCurrentColumns } from "@/shared/control";
// import moment from 'moment';
// import moment from "moment";

const store = useDashboardStore()
const toast = useToast();

let documentStyle = getComputedStyle(document.documentElement);

let textColor = documentStyle.getPropertyValue('--text-color');
const { isDarkTheme } = useLayout();

const products = ref(null);

const value3 = ref(null);

const semanaAtras = ref(0);

const _f = 'LL'


// moment.locale('es-mx')


const setData = (param) => {
  store.setOperationsVentas(moment().format('YYYY-MM-DD'), semanaAtras.value)
  // store.setOperationsCierres()
  store.setProductsTOP(moment().format('YYYY-MM-DD'), semanaAtras.value)
  // store.getAll(moment().subtract(1, 'days').format('YYYY-MM-DD'));
  store.getAll(moment().format('YYYY-MM-DD'));
  // console.log'setCurrentP-> ', store.getOperations)
}

onMounted(() => {
  // console.logmoment().startOf('week').format('LLLL'))
  // chartData.value = setChartData();
  // chartOptions.value = setChartOptions();

  setData(moment().format('YYYY-MM-DD'))

});

const formatCurrency = (value) => {
  return value?.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
};

const handleChange = (add = false) => {
  if (!!add) {
    if (semanaAtras > 0)
      semanaAtras.value--
    else{
      return
    }

  } else {
    semanaAtras.value++
  }
  store.setOperationsVentas(moment().format('YYYY-MM-DD'), semanaAtras.value)
  store.setProductsTOP(moment().format('YYYY-MM-DD'), semanaAtras.value)

};

const getTextSem = (dias) => {
  if (!dias)
    return 'Semana actual'

  const baseDate = moment().subtract((Number(dias) || 0), 'weeks')
  const start = baseDate.startOf('week').format(_f), end = baseDate.endOf('week').format(_f);
  // console.logstart, end)
  return start
}
const getTextProds = (dias) => {
  const baseDate = moment().subtract((Number(dias) || 0), 'weeks')
  let start = baseDate.startOf('week').format(_f), end = baseDate.endOf('week').format(_f);
  if (!dias) {
    baseDate.hours(3)
    start = baseDate.startOf('week').hours(15).calendar()
    return `desde ${start}.`
  }
  // console.logstart, end)
  return `del ${start} al ${end}.`
}



</script>

<template>
  <!--  {{store.getDataFor}}-->
  <div class="grid">
    <Toast />

    <div class="col-12 pb-0">
      <p class="mb-0 p-0">Resúmen del día de hoy</p>
    </div>
    <div class="col-6  lg:col-6 xl:col-3">
      <div class="card mb-0 px-3 py-3 ">
        <div class="flex  justify-content-between">
          <div class="px-3">
            <div>
              <div class="text-900 font-medium text-xl">{{ formatCurrency(store.getCobrosDia | 0) }}</div>
            </div>
            <span class="text-600 font-medium mb-3">Cobros</span>
            <span class="text-500"> del día</span>
          </div>
          <div class="flex ms-4 align-items-center justify-content-center bg-red-100 border-round"
            style="width: 1.4rem; height: 1.4rem">
            <i class="pi pi-shopping-cart text-red-500 text-xl"></i>
          </div>
        </div>
      </div>
    </div>
    <div class="col-6  lg:col-6 xl:col-3">
      <div class="card mb-0 px-3 py-3 ">
        <div class="flex  justify-content-between">

          <div class="px-3">
            <div>
              <div class="text-900 font-medium text-xl">{{ formatCurrency(store.getUtilidadDia | 0) }}</div>
            </div>
            <span class="text-600 font-medium mb-3">Utilidad</span>
            <span class="text-500"> del día</span>
          </div>
          <div class="flex ms-4 align-items-center justify-content-center bg-cyan-100 border-round"
            style="width: 1.4rem; height: 1.4rem">
            <i class="pi pi-shopping-cart text-cyan-500 text-xl"></i>
          </div>
        </div>
      </div>
    </div>
    <div class="col-6  lg:col-6 xl:col-3">
      <div class="card mb-0 px-3 py-3 ">
        <div class="flex  justify-content-between">

          <div class="px-3">
            <div>
              <div class="text-900 font-medium text-xl">{{ formatCurrency(store.getComisionDia | 0) }}</div>
            </div>
            <span class="text-600 font-medium mb-3">Comisiones</span>
            <span class="text-500"> del día</span>
          </div>
          <div class="flex ms-4 align-items-center justify-content-center bg-green-100 border-round"
            style="width: 1.4rem; height: 1.4rem">
            <i class="pi pi-shopping-cart text-green-500 text-xl"></i>
          </div>
        </div>
      </div>
    </div>
    <div class="col-6  lg:col-6 xl:col-3">
      <div class="card mb-0 px-3 py-3 ">
        <div class="flex  justify-content-between">

          <div class="px-3">
            <div>
              <div class="text-900 font-medium text-xl">{{ store.getKltsDia || 0 }}<small class="text-500">Klts</small>
              </div>
            </div>
            <span class="text-600 font-medium mb-3">Kilolitros</span>
            <span class="text-500"> del día</span>
          </div>
          <div class="flex ms-4 align-items-center justify-content-center bg-orange-100 border-round"
            style="width: 1.4rem; height: 1.4rem">
            <i class="pi pi-shopping-cart text-orange-500 text-xl"></i>
          </div>
        </div>
      </div>
    </div>

    <div class="col-12 mt-3">
      <div class="ctrl-box gap-5">
        <button :disabled="store.getLoading === true" @click="(e) => handleChange(false)" class="btn-dire flex-1">
          <template v-if="store.getLoading === true">
            <i class="pi pi-spinner loading"></i>
          </template>
          <template v-else>
            <i class="pi pi-chevron-left"></i>
          </template>
        </button>
        <div :class="(semanaAtras ? 'text-black-alpha-90 bold' : 'text-muted') + 'flex-1 min-w-200'">
          <template v-if="!!semanaAtras === true">
            <small class="text-light">Semana del</small>
          </template>
          <p class="strong">

            {{ getTextSem(semanaAtras) }}
          </p>
        </div>
        <button :disabled="store.getLoading === true" class="btn-dire flex-1" @click="(e) => handleChange(true)">
          <template v-if="store.getLoading === true">
            <i class="pi pi-spinner loading"></i>
          </template>
          <template v-else>
            <i class="pi pi-chevron-right"></i>
          </template>
        </button>
      </div>
    </div>

    <div class="col-12 xl:col-6">
      <div class="card p-2">
        <div class="card-header p-2 d-inline-flex">

          <h5 class="mb-1">Ventas <small class="text-500">de la semana en curso</small></h5>
          <b class="text-primary">cobrado MXN$</b>

        </div>
        <template v-if="store.getLoading === true">
          <div class="card flex bg-light justify-content-center">

            <Image src="demo/images/Vanilla-1s-234px.gif" alt="Image" width="250" />
          </div>
        </template>
        <template v-else>
          <Chart id="34" type="line" :data="store.getLineData" :options="store.getLineOptions" class="h-25rem" />
        </template>

      </div>

    </div>
    <div class="col-12 xl:col-6">
      <div class="card p-2">
        <div class="card-header p-2 d-inline-flex">

          <h5 class="mb-1">Cierres <small class="text-500">de la semana en curso</small></h5>
          <b class="text-primary"> Kilo-Litros</b>


        </div>
        <div class="ldBar" data-value="50">
        </div>
        <template v-if="store.getLoading === true">
          <div class="card flex bg-light justify-content-center">
            <img :src="'demo/images/Vanilla-1s-234px.gif'" :alt="'Cargando'" class="product-image" />
          </div>
        </template>
        <template v-else>
          <Chart id="456" type="bar" :data="store.getBarData" :options="store.getBarOptions" class="h-25rem" />
        </template>
        <!-- <div class="ctrl-box">
          <button @click="(e) => {
                semanaAtras = semanaAtras + 1
                store.setOperationsVentas(moment().format('YYYY-MM-DD'), semanaAtras)
              }" class="btn-dire">
            <i class="pi pi-chevron-left"></i>
          </button>
          <div :class="semanaAtras ? 'text-black-alpha-90 bold' : 'text-muted'">
            {{ getTextSem(semanaAtras) }}
          </div>
          <button class="btn-dire" @click="(e) => {
                semanaAtras > 0 ? semanaAtras = semanaAtras - 1 : null
                store.setOperationsVentas(moment().format('YYYY-MM-DD'), semanaAtras)
              }">

            <i class="pi pi-chevron-right"></i>
          </button>
        </div> -->

      </div>
    </div>

    <div class="col-12 xl:col-6">
      <!-- <div class="card">
          <h5>Recent Sales</h5>
          <DataTable :value="products" :rows="5" :paginator="true" responsiveLayout="scroll">
              <Column style="width: 15%">
                  <template #header> Image </template>
                  <template #body="slotProps">
                      <img :src="'demo/images/product/' + slotProps.data.image" :alt="slotProps.data.image" width="50" class="shadow-2" />
                  </template>
              </Column>
              <Column field="name" header="Name" :sortable="true" style="width: 35%"></Column>
              <Column field="price" header="Price" :sortable="true" style="width: 35%">
                  <template #body="slotProps">
                      {{ formatCurrency(slotProps.data.price) }}
                  </template>
              </Column>
              <Column style="width: 15%">
                  <template #header> View </template>
                  <template #body>
                      <Button icon="pi pi-search" type="button" class="p-button-text"></Button>
                  </template>
              </Column>
          </DataTable>
      </div> -->
      <div class="card">
        <div class="flex justify-content-between align-items-center mb-0">
          <h5 class="m-0">Productos más vendidos</h5>
        </div>
        <p class="font-light">Detalle de venta en los productos más vendidos {{ getTextProds(semanaAtras) }}</p>
        <ul class="list-none p-0 m-0">
          <li v-for="(item, index) in store.getProdTop"
            class="flex flex-column md:flex-row md:align-items-center md:justify-content-between mb-2 border-div">
            <div>
              <b class="text-muted">{{ index }}.- </b>
              <span class="mt-1 text-600">{{ item.description }} </span>
              <span class="text-red-600 ml-1 font-semibold text-xs"> [{{ item.code }}]</span>
            </div>
            <div class="mt-2 md:mt-0 flex align-items-center">
              <span class="text-400 ml-3 font-light">{{ item.vendido }}<small>Pzs.</small></span>
              <span class="text-red-600 ml-3 font-medium">{{ formatCurrency(item.vendido * item.precio_lista) }}</span>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <!--    <div class="col-12 xl:col-6">-->
    <!--      <div class="card">-->
    <!--        <div class="flex align-items-center justify-content-between mb-4">-->
    <!--          <h5>Notifications</h5>-->
    <!--          <div>-->
    <!--            <Button icon="pi pi-ellipsis-v" class="p-button-text p-button-plain p-button-rounded"-->
    <!--                    @click="$refs.menu1.toggle($event)"></Button>-->
    <!--            <Menu ref="menu1" :popup="true" :model="items"></Menu>-->
    <!--          </div>-->
    <!--        </div>-->

    <!--        <span class="block text-600 font-medium mb-3">TODAY</span>-->
    <!--        <ul class="p-0 mx-0 mt-0 mb-4 list-none">-->
    <!--          <li class="flex align-items-center py-2 border-bottom-1 surface-border">-->
    <!--            <div-->
    <!--                class="w-3rem h-3rem flex align-items-center justify-content-center bg-blue-100 border-circle mr-3 flex-shrink-0">-->
    <!--              <i class="pi pi-dollar text-xl text-blue-500"></i>-->
    <!--            </div>-->
    <!--            <span class="text-900 line-height-3"-->
    <!--            >Richard Jones-->
    <!--                            <span class="text-700">has purchased a blue t-shirt for <span-->
    <!--                                class="text-blue-500">79$</span></span>-->
    <!--                        </span>-->
    <!--          </li>-->
    <!--          <li class="flex align-items-center py-2">-->
    <!--            <div-->
    <!--                class="w-3rem h-3rem flex align-items-center justify-content-center bg-orange-100 border-circle mr-3 flex-shrink-0">-->
    <!--              <i class="pi pi-download text-xl text-orange-500"></i>-->
    <!--            </div>-->
    <!--            <span class="text-700 line-height-3">Your request for withdrawal of <span class="text-blue-500 font-medium">2500$</span> has been initiated.</span>-->
    <!--          </li>-->
    <!--        </ul>-->

    <!--        <span class="block text-600 font-medium mb-3">YESTERDAY</span>-->
    <!--        <ul class="p-0 m-0 list-none">-->
    <!--          <li class="flex align-items-center py-2 border-bottom-1 surface-border">-->
    <!--            <div-->
    <!--                class="w-3rem h-3rem flex align-items-center justify-content-center bg-blue-100 border-circle mr-3 flex-shrink-0">-->
    <!--              <i class="pi pi-dollar text-xl text-blue-500"></i>-->
    <!--            </div>-->
    <!--            <span class="text-900 line-height-3"-->
    <!--            >Keyser Wick-->
    <!--                            <span class="text-700">has purchased a black jacket for <span-->
    <!--                                class="text-blue-500">59$</span></span>-->
    <!--                        </span>-->
    <!--          </li>-->
    <!--          <li class="flex align-items-center py-2 border-bottom-1 surface-border">-->
    <!--            <div-->
    <!--                class="w-3rem h-3rem flex align-items-center justify-content-center bg-pink-100 border-circle mr-3 flex-shrink-0">-->
    <!--              <i class="pi pi-question text-xl text-pink-500"></i>-->
    <!--            </div>-->
    <!--            <span class="text-900 line-height-3"-->
    <!--            >Jane Davis-->
    <!--                            <span class="text-700">has posted a new questions about your product.</span>-->
    <!--                        </span>-->
    <!--          </li>-->
    <!--        </ul>-->
    <!--      </div>-->
    <!--    </div>-->

  </div>
</template>

<style>
.w-50 {
  width: 50%;
}

.w-3rem {
  width: 3rem
}

.d-inline-flex {
  display: inline-flex;
  width: 100%;
  justify-content: space-between;
  align-items: baseline;
}

.btn-dire {
  color: grey;
  border: none;
  cursor: pointer;
  background: #8585851f;
  border-radius: 6px;
  padding: .5rem 2rem;
  transition: all .2s ease-out;
}

.btn-dire:hover {
  background: rgba(45, 37, 37, 0.12);

}

.ctrl-box {
  background: #f7f7f736;
  width: 100%;
  display: inline-flex;
  /* grid-auto-flow: column; */
  align-items: baseline;
  margin-top: 0rem;
  padding: 0 .4rem;
  flex-direction: row;
  border-top: 1px solid #f7f7f7;
  align-content: center;
  align-items: center;
  justify-content: space-evenly;
}

.min-w-200 {
  background-color: #f7f7f7;
  text-align: center;
  font-weight: 700;
  min-width: 200px;
}

.text-light {
  font-weight: 300;
}

.loading {
  animation-name: rotate;
  animation-duration: 1s;
  animation-fill-mode: forwards;
  animation-timing-function: ease;
  animation-direction: normal;
  animation-iteration-count: infinite;
}

.border-div {
  border-top: 1px solid rgba(128, 128, 128, 0.312);
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}
</style>