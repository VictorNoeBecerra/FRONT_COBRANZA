<!-- eslint-disable vue/multi-word-component-names -->
<script setup>
import { getCurrentColumns } from "../shared/control";
import { useCatalogosStore } from "@/stores";
import { FilterMatchMode, FilterOperator } from "primevue/api";

const frmat = (txt) => capitalize(txt).replaceAll("_", " ");
import { capitalize, onBeforeMount, onMounted, ref, watch } from "vue";
import { defineProps } from "vue";
import { useRoute } from "vue-router";

const deleteProductDialog = ref(false);
const deleteProductsDialog = ref(false);

const selectedProducts = ref(null);

const filters = ref({
  global: { value: null, matchMode: FilterMatchMode.CONTAINS },
  precio_lista: {
    operator: FilterOperator.AND,
    constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }],
  },
  precio_costo: {
    operator: FilterOperator.AND,
    constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }],
  },
  comision: {
    operator: FilterOperator.AND,
    constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }],
  },
  content: {
    operator: FilterOperator.AND,
    constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }],
  },
});
const statuses = ref([
  { label: "INSTOCK", value: "instock" },
  { label: "LOWSTOCK", value: "lowstock" },
  { label: "OUTOFSTOCK", value: "outofstock" },
]);

const dt = ref(null);
const productDialog = ref(false);
const product = ref({});
const submitted = ref(false);
const isEditing = ref(false);

const route = useRoute();
const store = useCatalogosStore();

let currentP = {};
const genders = ref([
  { label: "Hombre", value: "H" },
  { label: "Mujer", value: "M" },
  { label: "Otro", value: "O" },
]);
let columns = [
  { field: "code", header: "Code" },
  { field: "name", header: "Name" },
  { field: "category", header: "Category" },
  { field: "quantity", header: "Quantity" },
];
const setCurrentP = (param) => {
  console.log("\t\tsetCurrentP", param);
  const { routeApi, properties } = getCurrentColumns(frmat(param))[0];
  currentP = {
    key: param,
    title: frmat(param),
    routeApi,
  };
  columns = properties;
  currentP.routeApi = routeApi;
  store.getAll(routeApi);
  console.log("\t\tsetCurrentP-> ", currentP, columns);
};
const props = defineProps({
  indx: { type: String, default: "ss" },
});

const formatCurrency = (value) => {
  return value?.toLocaleString("es-MX", { style: "currency", currency: "MXN" });
};
const formatDate = (value) => {
  return value.toLocaleDateString("en-US", {
    day: "2-digit",
    month: "2-digit",
    year: "numeric",
  });
};
// const formatCurrency = (value) => {
//     return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
// };

onBeforeMount(() => {
  console.count("onBeforeMount()");
  console.log("beforeMount");
  console.log("setCurrentP->1 ", props.indx);
  setCurrentP(props.indx);
  store.getSecondaryData();
});

onMounted(() => {
  console.log("Mounted");
});

const editProduct = (editProduct) => {
  product.value = { ...editProduct };
  isEditing.value = true;
  console.log("product.value: ", product.value);
  productDialog.value = true;
};

const exportCSV = () => {
  dt.value.exportCSV();
};

const confirmDeleteProduct = (editProduct) => {
  console.log("confirmDeleteProduct");
  product.value = editProduct;
  deleteProductDialog.value = true;
};
const openNew = () => {
  isEditing.value = false;
  product.value = {};
  submitted.value = false;
  productDialog.value = true;
};
const hideDialog = () => {
  productDialog.value = false;
  submitted.value = false;
};

const deleteProduct = async () => {
  console.log("delete", product.value);
  let _id = "";
  if (currentP.key === "Rutas") {
    _id = product.value.no_ruta;
  } else if (currentP.key === "Productos") {
    _id = product.value.code;
  } else {
    _id = product.value.id;
  }
  await store.delete(currentP.routeApi, _id);
  deleteProductDialog.value = false;
  console.log("", currentP);
  console.log("setCurrentP->2 ", currentP.key);

  setCurrentP(currentP.key);
};
const saveProduct = async () => {
  submitted.value = true;

  console.log("saveProduct", product.value, currentP);
  if (product.value?.id) {
    let body = {};
    if (currentP?.key === "Repartidores") {
      const { ruta, sexo, nombres, edad } = product.value;
      body = {
        sexo: sexo.value || sexo,
        ruta: ruta.value || ruta,
        nombres,
        edad,
      };
    } else {
      body = product.value;
    }
    await store.update(currentP?.routeApi, product.value?.id, body);
  } else if (currentP?.key === "Productos") {
    const body = product.value;
    console.log("body: ", body);
    const newBody = {
      code: body.code,
      autoservicio: body.autoservicio,
      description: body.description,
      precio_lista: body.precio_lista,
      precio_compra: body.precio_compra,
      cant_caja: body.cant_caja,
      content: body.content,
      comision: body.comision,
      um: body.um.id,
      grupo: body.grupo.id,
    };
    console.log("newBody: ", newBody);
    console.log("----- is Editing: ", isEditing.value, isEditing);
    if (isEditing.value === true) {
      console.log("\tediting", currentP?.routeApi, product.value.code, newBody);
      await store.update(currentP?.routeApi, product.value.code, newBody);
    } else {
      console.log("\tregister", newBody);
      await store.register(currentP?.routeApi, newBody);
    }
  } else if (product.value.no_ruta && currentP?.key === "Rutas") {
    const body = product.value;
    console.log(body);

    if (isEditing.value === true) {
      console.log("editing");
      await store.update(currentP?.routeApi, product.value.no_ruta, body);
    } else {
      console.log("register", body);
      await store.register(currentP?.routeApi, body);
    }
  } else {
    console.log("-----------------------");
    console.log("ELSE BEFORE SWITCH");
    console.log("else: ", currentP?.key, currentP?.value);
    switch (currentP?.key) {
      case "grupos":
        const { familia } = product.value;
        console.log("--", familia);
        product.value.familia = familia.id;
        break;
      case "factores_de_conversión":
        const { um, um_eq } = product.value;
        console.log("--", um, um_eq);
        product.value.um = um.id;
        product.value.um_eq = um_eq.id;
        break;
      case "Repartidores":
        const { ruta, sexo, nombres, edad } = product.value;
        console.log("Repartidores");
        console.log("--", ruta, sexo);


        product.value = {
          sexo: typeof sexo === "object" ? sexo.value : sexo,
          ruta: typeof ruta === "object" ? ruta.no_ruta : ruta,
          nombres,
          edad,
        };
        console.log(product.value);
        break;
      default:
        console.log("default::currentP", currentP?.value);
        break;
    }
    await store.register(currentP?.routeApi, product.value);
  }
  productDialog.value = false;

  await store.getAll(currentP?.routeApi);
  await store.getSecondaryData();
  // setCurrentP(currentP.value.routeApi)

  // if (product.value.name && product.value.name.trim() && product.value.price) {
  //   if (product.value.id) {
  //     product.value.inventoryStatus = product.value.inventoryStatus.value ? product.value.inventoryStatus.value : product.value.inventoryStatus;
  //     products.value[findIndexById(product.value.id)] = product.value;
  //     toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Updated', life: 3000 });
  //   } else {
  //     product.value.id = createId();
  //     product.value.code = createId();
  //     product.value.image = 'product-placeholder.svg';
  //     product.value.inventoryStatus = product.value.inventoryStatus ? product.value.inventoryStatus.value : 'INSTOCK';
  //     products.value.push(product.value);
  //     toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Created', life: 3000 });
  //   }
  //   productDialog.value = false;
  //   product.value = {};
  // }
};
watch(
  () => props.indx,
  (val) => {
    console.log(val);
    console.log("setCurrentP->3", val);
    setCurrentP(val);
  }
);
</script>

<template>
  <div class="grid">
    <div class="col-12">
      <div class="card p-2 py-3">
        <h5 class="px-4">{{ currentP?.title }}</h5>

        <DataTable
          v-model:filters="filters"
          :value="store.dataCatalog"
          :key="currentP?.key"
          :loading="store.isLoading"
          :paginator="true"
          :rowHover="true"
          class="p-data-table-sm"
          :rows="10"
          dataKey="id"
          filterDisplay="menu"
          :filters="filters"
          responsiveLayout="scroll"
          :globalFilterFields="columns.map((c) => c.field)"
        >
          <template #header>
            <div class="flex flex-wrap justify-content-between gap-2">
              <span class="p-input-icon-left">
                <i class="pi pi-search" />
                <InputText v-model="filters['global'].value" placeholder="Busqueda" />
              </span>
              <div class="formgroup-inline align-items-baseline">
                <span class="p-buttonset">
                  <Button
                    label="Registrar"
                    size="small"
                    :loading="store.isLoading"
                    @click="openNew"
                    icon="pi pi-plus"
                  />
                  <Button
                    label="Guardar"
                    size="small"
                    :loading="store.isLoading"
                    @click="exportCSV($event)"
                    icon="pi pi-file-excel"
                  />
                </span>
              </div>
            </div>
          </template>
          <template #empty> {{ currentP?.title }} no tiene registros.</template>
          <template #loading> Cargando la información..</template>
          <Column
            v-for="col of columns"
            :filterField="col.field"
            :key="col.field"
            :field="col.field"
            :header="col.header"
          >
            <template v-if="'precio_lista' === col.field" #body="slotProps">
              <b> {{ formatCurrency(slotProps.data.precio_lista) }}</b>
            </template>
            <template v-if="'precio_compra' === col.field" #body="slotProps">
              <b> {{ formatCurrency(slotProps.data.precio_compra) }}</b>
            </template>
            <template v-if="'autoservicio' === col.field" #body="slotProps">
              <!--              <Checkbox :value="slotProps.data.autoservicio" readonly :binary="true"/>-->
              <b> {{ slotProps.data.autoservicio ? "Sí" : "No" }}</b>
            </template>
            <template
              v-if="
                ['precio_lista', 'precio_compra', 'comision', 'content'].includes(
                  col.field
                )
              "
              #filter="{ filterModel }"
            >
              <InputNumber
                v-model="filterModel.value"
                mode="currency"
                currency="USD"
                locale="en-US"
              />
            </template>
          </Column>
          <Column headerStyle="width:7em; min-width:fit-content;" class="btns-col">
            <template #body="slotProps">
              <Button text icon="pi pi-file-edit" @click="editProduct(slotProps.data)" />
              <Button
                text
                icon="pi pi-trash"
                @click="confirmDeleteProduct(slotProps.data)"
                severity="danger"
              />
            </template>
          </Column>
          <template #footer>
            Total de elementos: {{ store.dataCatalog.length || 0 }}.</template
          >
        </DataTable>

        <Dialog
          v-model:visible="productDialog"
          :style="{ width: '450px' }"
          :header="product.id || product.no_ruta ? 'Editar' : 'Registrar nuevo'"
          :modal="true"
          class="p-fluid"
        >
          <img
            :src="'demo/images/product/' + product.image"
            :alt="product.image"
            v-if="product.image"
            width="150"
            class="mt-0 mx-auto mb-5 block shadow-2"
          />
          <div class="field" v-if="['Rutas'].includes(currentP?.key)">
            <label for="no_ruta"># de Ruta</label>
            <InputText
              id="no_ruta"
              v-model.trim="product.no_ruta"
              required="true"
              autofocus
              :class="{ 'p-invalid': submitted && !product.no_ruta }"
            />
            <small class="p-invalid" v-if="submitted && !product.no_ruta"
              >Ingresa el # ruta.</small
            >
          </div>
          <div class="formgrid grid" v-if="['Productos'].includes(currentP?.key)">
            <div class="field col">
              <label for="code">Codigo</label>
              <InputText
                id="code"
                v-model.trim="product.code"
                required="true"
                :class="{ 'p-invalid': submitted && !product.code }"
              />
              <small class="p-invalid" v-if="submitted && !product.code"
                >code is required.</small
              >
            </div>
            <div class="field col check-center">
              <label for="autoservicio">Autoservicio</label>
              <Checkbox
                id="autoservicio"
                v-model.trim="product.autoservicio"
                :binary="true"
              />
            </div>
          </div>
          <div class="field" v-if="currentP?.key === 'Repartidores'">
            <label for="nombres">Nombres</label>
            <InputText
              id="nombres"
              v-model.trim="product.nombres"
              required="true"
              :class="{ 'p-invalid': submitted && !product.nombres }"
            />
            <small class="p-invalid" v-if="submitted && !product.nombres"
              >Name is required.</small
            >
          </div>

          <div class="field" v-if="['repartidores', 'grupos'].includes(currentP?.key)">
            <label for="nombre">Nombre</label>
            <InputText
              id="nombre"
              v-model.trim="product.nombre"
              required="true"
              :class="{ 'p-invalid': submitted && !product.nombre }"
            />
            <small class="p-invalid" v-if="submitted && !product.nombre"
              >Name is required.</small
            >
          </div>
          <div
            class="field"
            v-if="['Rutas', 'unidades_de_medida', 'familias'].includes(currentP?.key)"
          >
            <label for="descripcion">Descripcion</label>
            <InputText
              id="descripcion"
              v-model.trim="product.descripcion"
              required="true"
              :class="{ 'p-invalid': submitted && !product.descripcion }"
            />
            <small class="p-invalid" v-if="submitted && !product.descripcion"
              >descripcion is required.</small
            >
          </div>
          <div
            class="field"
            v-if="
              ['grupos', 'factores_de_conversión', 'Productos'].includes(currentP?.key)
            "
          >
            <label for="description">Descripcion</label>
            <InputText
              id="description"
              v-model.trim="product.description"
              required="true"
              :class="{ 'p-invalid': submitted && !product.description }"
            />
            <small class="p-invalid" v-if="submitted && !product.description"
              >description is required.</small
            >
          </div>

          <div
            class="field"
            v-if="['unidades_de_medida', 'familias'].includes(currentP?.key)"
          >
            <label for="codigo">Codigo</label>
            <InputText
              id="codigo"
              v-model.trim="product.code"
              required="true"
              autofocus
              :class="{ 'p-invalid': submitted && !product.code }"
            />
            <small class="p-invalid" v-if="submitted && !product.code"
              >Name is required.</small
            >
          </div>

          <div class="field" v-if="['grupos'].includes(currentP?.key)">
            <label for="inventoryStatus" class="mb-3">Familia</label>
            <Dropdown
              id="inventoryStatus"
              v-model="product.familia"
              :options="store.getFamilias"
              optionLabel="label"
              placeholder="Selecciona la familia"
            >
              <template #value="slotProps">
                {{ slotProps }}
                <div v-if="slotProps.value && slotProps.value.value">
                  <span :class="'product-badge status-' + slotProps.value.value">{{
                    slotProps.value.label
                  }}</span>
                </div>
                <div v-else-if="slotProps.value && !slotProps.value.value">
                  <span
                    :class="
                      'product-badge status-' + slotProps.value.toString().toLowerCase()
                    "
                    >{{
                      store.getFamilias.filter((f) => f.id === slotProps.value)[0].label
                    }}</span
                  >
                </div>
                <span v-else>
                  {{ slotProps.placeholder }}
                </span>
              </template>
            </Dropdown>
          </div>

          <div class="formgrid grid" v-if="['Productos'].includes(currentP?.key)">
            <div class="field col">
              <label for="precio">$ Compra</label>
              <InputNumber
                id="precio"
                v-model="product.precio_compra"
                mode="currency"
                currency="MXN"
                lolocale="es-MX"
                console="es-MX"
                :class="{ 'p-invalid': submitted && !product.precio_compra }"
                :required="true"
              />
              <small class="p-invalid" v-if="submitted && !product.precio_compra"
                >Price is required.</small
              >
            </div>
            <div class="field col">
              <label for="costo">$ Venta</label>
              <InputNumber
                id="costo"
                v-model="product.precio_lista"
                mode="currency"
                currency="MXN"
                lolocale="es-MX"
                console="es-MX"
                :class="{ 'p-invalid': submitted && !product.precio_lista }"
                :required="true"
              />
              <small class="p-invalid" v-if="submitted && !product.precio_lista"
                >Price is required.</small
              >
            </div>
            <div class="field col">
              <label for="comision">$ comision</label>
              <InputNumber
                id="comision"
                v-model="product.comision"
                mode="currency"
                :min-fraction-digits="3"
                currency="MXN"
                lolocale="es-MX"
                console="es-MX"
                :class="{ 'p-invalid': submitted && !product.comision }"
                :required="true"
              />
              <small class="p-invalid" v-if="submitted && !product.comision"
                >Price is required.</small
              >
              <small class="p-invalid" v-if="submitted && !product.comision"
                >Price is required.</small
              >
            </div>
          </div>
          <div class="formgrid grid" v-if="['Repartidores'].includes(currentP?.key)">
            <div class="field col">
              <label for="sexo">Genero</label>
              <Dropdown
                id="sexo"
                v-model="product.sexo"
                :options="genders"
                optionLabel="value"
                placeholder="sexo"
                option-value="value"
              >
                <template #value="slotProps">
                  <div v-if="slotProps.value && slotProps.value.value">
                    <span :class="'product-badge status-' + slotProps.value.value">{{
                      slotProps.value.label
                    }}</span>
                  </div>
                  <div v-else-if="slotProps.value && !slotProps.value.value">
                    <span
                      :class="
                        'product-badge status-' + slotProps.value.toString().toLowerCase()
                      "
                      >{{
                        genders.filter((g) => g.value === slotProps.value)[0].label
                      }}</span
                    >
                  </div>
                  <span v-else>
                    {{ slotProps.placeholder }}
                  </span>
                </template>
              </Dropdown>
            </div>
            <div class="field col">
              <label for="edad">Edad</label>
              <InputNumber
                id="edad"
                v-model="product.edad"
                mode="decimal" showButtons :min="0" :max="100"
                :required="true"
                :class="{ 'p-invalid': submitted && !product.edad }"
              />
              <small class="p-invalid" v-if="submitted && !product.edad"
                >Factor is required.</small
              >
            </div>
          </div>
          <div class="field" v-if="['Repartidores'].includes(currentP?.key)">
            <label for="ruta" class="mb-3">Ruta</label>
            <Dropdown
              id="ruta"
              v-model="product.ruta"
              :options="store.getRutas"
              optionLabel="value"
              placeholder="Selecciona la ruta"
            >
              <template #value="slotProps">
                <div v-if="slotProps.value && slotProps.value.value">
                  <span :class="'product-badge status-' + slotProps.value.value">{{
                    slotProps.value.label
                  }}</span>
                </div>
                <div v-else-if="slotProps.value && !slotProps.value.value">
                  <span
                    :class="
                      'product-badge status-' + slotProps.value.toString().toLowerCase()
                    "
                    >{{
                      store.getRutas.find((f) => f.no_ruta === slotProps.value)?.label
                    }}</span
                  >
                </div>
                <span v-else>
                  {{ slotProps.placeholder }}
                </span>
              </template>
            </Dropdown>
          </div>
          <div
            class="formgrid grid"
            v-if="['factores_de_conversión'].includes(currentP?.key)"
          >
            <div class="field col">
              <label for="um">U. origen</label>
              <Dropdown
                id="um"
                v-model="product.um"
                :options="store.getUnidadesMedida"
                optionLabel="label"
                placeholder="Um"
              >
                <template #value="slotProps">
                  <div v-if="slotProps.value && slotProps.value.value">
                    <span :class="'product-badge status-' + slotProps.value.value">{{
                      slotProps.value.label
                    }}</span>
                  </div>
                  <div v-else-if="slotProps.value && !slotProps.value.value">
                    <span
                      :class="
                        'product-badge status-' + slotProps.value.toString().toLowerCase()
                      "
                      >{{
                        // store.getGrupos.filter(f => f.id === slotProps.value)[0].label
                        slotProps.value
                      }}</span
                    >
                  </div>
                  <span v-else>
                    {{ slotProps.placeholder }}
                  </span>
                </template>
              </Dropdown>
            </div>
            <div class="field col">
              <label for="um_eq">U. destino</label>
              <Dropdown
                id="um_eq"
                v-model="product.um_eq"
                :options="store.getUnidadesMedida"
                optionLabel="label"
                placeholder="um_eq"
              >
                <template #value="slotProps">
                  <div v-if="slotProps.value && slotProps.value.value">
                    <span :class="'product-badge status-' + slotProps.value.value">{{
                      slotProps.value.label
                    }}</span>
                  </div>
                  <div v-else-if="slotProps.value && !slotProps.value.value">
                    <span
                      :class="
                        'product-badge status-' + slotProps.value.toString().toLowerCase()
                      "
                      >{{
                        // store.getGrupos.filter(f => f.id === slotProps.value)[0].label
                        slotProps.value
                      }}</span
                    >
                  </div>
                  <span v-else>
                    {{ slotProps.placeholder }}
                  </span>
                </template>
              </Dropdown>
            </div>
            <div class="field col">
              <label for="cantidad">Factor</label>
              <InputNumber
                id="cantidad"
                v-model="product.cantidad"
                mode="decimal"
                :class="{ 'p-invalid': submitted && !product.cantidad }"
                :required="true"
              />
              <small class="p-invalid" v-if="submitted && !product.cantidad"
                >Factor is required.</small
              >
            </div>
          </div>
          <div class="formgrid grid" v-if="['Productos'].includes(currentP?.key)">
            <div class="field col">
              <label for="cont">Cont</label>
              <InputNumber
                id="cont"
                v-model="product.content"
                mode="decimal"
                :class="{ 'p-invalid': submitted && !product.content }"
                :required="true"
              />
              <small class="p-invalid" v-if="submitted && !product.content"
                >Price is required.</small
              >
            </div>
            <div class="field col">
              <label for="um">Ud. medida</label>
              <Dropdown
                id="um"
                v-model="product.um"
                :options="store.getUnidadesMedida"
                optionLabel="label"
                placeholder="U.medida"
              >
                <template #value="slotProps">
                  <div v-if="slotProps.value && slotProps.value.value">
                    <span :class="'product-badge status-' + slotProps.value.value">{{
                      slotProps.value.label
                    }}</span>
                  </div>
                  <div v-else-if="slotProps.value && !slotProps.value.value">
                    <span
                      :class="
                        'product-badge status-' + slotProps.value.toString().toLowerCase()
                      "
                    >
                      {{
                        store.getUnidadesMedida.filter((f) => f.id === slotProps.value)[0]
                          .codigo
                      }}</span
                    >
                  </div>
                  <span v-else>
                    {{ slotProps.placeholder }}
                  </span>
                </template>
              </Dropdown>
              <!--              <label for="um">um</label>-->
              <!--              <InputNumber id="um" v-model="product.um" mode="currency" currency="USD" lolocale="en-US" console="en-US"-->
              <!--                           :class="{ 'p-invalid': submitted && !product.um }" :required="true"/>-->
              <small class="p-invalid" v-if="submitted && !product.um"
                >Price is required.</small
              >
            </div>
            <div class="field col">
              <label for="caja">Unidades caja</label>
              <InputNumber
                id="cont"
                v-model="product.cant_caja"
                mode="decimal"
                :class="{ 'p-invalid': submitted && !product.cant_caja }"
                integeronly
                :required="true"
              />
              <small class="p-invalid" v-if="submitted && !product.cant_caja"
                >Price is required.</small
              >
            </div>
          </div>

          <div class="field" v-if="['Productos'].includes(currentP?.key)">
            <label for="grupos" class="mb-3">Grupo</label>
            <Dropdown
              id="grupos"
              v-model="product.grupo"
              :options="store.getGrupos"
              optionLabel="label"
              placeholder="Selecciona la un grupo"
            >
              <template #value="slotProps">
                <div v-if="slotProps.value && slotProps.value.value">
                  <span :class="'product-badge status-' + slotProps.value.value">{{
                    slotProps.value.label
                  }}</span>
                </div>
                <div v-else-if="slotProps.value && !slotProps.value.value">
                  <span
                    :class="
                      'product-badge status-' + slotProps.value.toString().toLowerCase()
                    "
                  >
                    {{ store.getGrupos.filter((f) => f.id === slotProps.value)[0].label }}
                  </span>
                </div>
                <span v-else>
                  {{ slotProps.placeholder }}
                </span>
              </template>
            </Dropdown>
          </div>

          <template #footer>
            <Button
              label="Cancelar"
              icon="pi pi-times"
              class="p-button-text"
              @click="hideDialog"
            />
            <Button
              label="Guardar"
              icon="pi pi-check"
              class="p-button-text"
              @click="saveProduct"
            />
          </template>
        </Dialog>
        <Dialog
          v-model:visible="deleteProductDialog"
          :style="{ width: '450px' }"
          header="Eliminar"
          :modal="true"
        >
          <div class="flex align-items-center justify-content-center">
            <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
            <span v-if="product"
              >Estás seguro que desear eliminar el elemento
              <b>{{ product.descripcion }}</b
              >?</span
            >
          </div>
          <template #footer>
            <Button
              label="No"
              icon="pi pi-times"
              class="p-button-text"
              @click="deleteProductDialog = false"
            />
            <Button
              label="Yes"
              icon="pi pi-check"
              class="p-button-text"
              @click="deleteProduct"
            />
          </template>
        </Dialog>
        <!--          <div class="field">-->
        <!--            <label class="mb-3">Category</label>-->
        <!--            <div class="formgrid grid">-->
        <!--              <div class="field-radiobutton col-6">-->
        <!--                <RadioButton id="category1" name="category" value="Accessories" v-model="product.category"/>-->
        <!--                <label for="category1">Accessories</label>-->
        <!--              </div>-->
        <!--              <div class="field-radiobutton col-6">-->
        <!--                <RadioButton id="category2" name="category" value="Clothing" v-model="product.category"/>-->
        <!--                <label for="category2">Clothing</label>-->
        <!--              </div>-->
        <!--              <div class="field-radiobutton col-6">-->
        <!--                <RadioButton id="category3" name="category" value="Electronics" v-model="product.category"/>-->
        <!--                <label for="category3">Electronics</label>-->
        <!--              </div>-->
        <!--              <div class="field-radiobutton col-6">-->
        <!--                <RadioButton id="category4" name="category" value="Fitness" v-model="product.category"/>-->
        <!--                <label for="category4">Fitness</label>-->
        <!--              </div>-->
        <!--            </div>-->
        <!--          </div>-->

        <Dialog
          v-model:visible="deleteProductsDialog"
          :style="{ width: '450px' }"
          header="Confirm"
          :modal="true"
        >
          <div class="flex align-items-center justify-content-center p-3">
            <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
            <span v-if="product"
              >Are you sure you want to delete the selected products?</span
            >
          </div>
          <template #footer>
            <Button
              label="No"
              icon="pi pi-times"
              class="p-button-text"
              @click="deleteProductsDialog = false"
            />
            <Button
              label="Si"
              icon="pi pi-check"
              class="p-button-text"
              @click="deleteSelectedProducts"
            />
          </template>
        </Dialog>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
::v-deep(.btns-col) {
  padding-top: 0 !important;
  padding-bottom: 0 !important;
}

::v-deep(.p-datatable .p-datatable-tbody > tr > td) {
  text-align: left;
  border: 1px solid #f4f4f5;
  border-width: 0 0 1px 0;
  padding: 0.5rem 1rem;
}

.check-center {
  display: inline-grid;
  justify-content: center;
  justify-items: center;
  align-items: end;
  align-content: center;
}
</style>
