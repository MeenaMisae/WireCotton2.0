<template>
  <div>
    <Toast />
    <div class="card">
      <Toolbar class="mb-6">
        <template #start>
          <RouterLink to="/products/create">
            <Button label="Novo" icon="pi pi-plus" class="mr-2" @click="openNew" />
          </RouterLink>
          <Button label="Excluir" icon="pi pi-trash" severity="danger" @click="confirmDeleteSelected"
            :disabled="!selectedProducts || !selectedProducts.length" />
        </template>

        <template #end>
          <InputText v-model="filters['global'].value" placeholder="Pesquisar..." />
          <FileUpload mode="basic" accept="image/*" :maxFileSize="1000000" label="Import" chooseLabel="Import"
            class="mr-2" auto />
          <Button label="Exportar" icon="pi pi-upload" severity="help" @click="exportCSV($event)" />
        </template>
      </Toolbar>

      <DataTable ref="dt" v-model:selection="selectedProducts" :value="products" dataKey="" :paginator="true" :rows="4"
        :filters="filters"
        paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
        :rowsPerPageOptions="[5, 10, 25]"
        currentPageReportTemplate="Mostrando {first} de {last} do total de {totalRecords} produtos">
        <Column selectionMode="multiple" style="width: 3rem" :exportable="false"></Column>
        <Column field="code" header="SKU" sortable style="min-width: 12rem"></Column>
        <Column field="name" header="Nome" sortable style="min-width: 12rem"> </Column>
        <Column header="Imagem" style="min-width: 12rem">
          <template #body="slotProps">
            <img :src="slotProps.data.images[0]" :alt="slotProps.data.image" class="rounded" style="width: 64px" />
          </template>
        </Column>
        <Column field="price" header="Preço" sortable style="min-width: 12rem">
          <template #body="slotProps"> R$ {{ formatCurrency(slotProps.data.price) }} </template>
        </Column>
        <Column field="category" header="Categoria" sortable style="min-width: 12rem"></Column>
        <Column field="quantity" header="Quantidade" sortable style="min-width: 12rem">
          <template #body="slotProps">
            {{ slotProps.data.quantity }}
          </template>
        </Column>
        <Column field="inventoryStatus" header="Status" sortable style="min-width: 12rem">
          <template #body="slotProps">
            <Tag :value="slotProps.data.inventoryStatus" :severity="getStatusLabel(slotProps.data.inventoryStatus)" />
          </template>
        </Column>
        <Column :exportable="false" style="min-width: 11rem">
          <template #body="slotProps">
            <Button icon="pi pi-pencil" outlined rounded class="mr-5" @click="editProduct(slotProps.data)" />
            <Button icon="pi pi-trash" outlined rounded severity="danger"
              @click="confirmDeleteProduct(slotProps.data)" />
          </template>
        </Column>
      </DataTable>
    </div>

    <Dialog v-model:visible="productDialog" :style="{ width: '450px' }" header="Detalhes do produto" :modal="true">
      <div class="flex flex-col gap-6">
        <img v-if="product.images" :src="product.images[0]" :alt="product.image" class="block m-auto rounded h-96" />
        <div>
          <label for="name" class="block mb-3 font-bold">Nome</label>
          <InputText id="name" v-model.trim="product.name" required="true" autofocus
            :invalid="submitted && !product.name" fluid class="w-full" />
          <small v-if="submitted && !product.name" class="text-red-500">Name is required.</small>
        </div>
        <div>
          <label for="description" class="block mb-3 font-bold">Descrição</label>
          <Textarea id="description" v-model="product.description" required="true" rows="5" cols="20" fluid
            class="w-full" />
        </div>
        <div>
          <span class="block mb-4 font-bold">Categoria</span>
          <div class="grid grid-cols-12 gap-4">
            <div class="flex items-center col-span-6 gap-2" v-for="category in categories" :key="category">
              <RadioButton :id="category.id" v-model="product.category" name="category" :value="category.name" />
              <label :for="category.id">{{ category.name }}</label>
            </div>
          </div>
        </div>
        <div>
          <span class="block mb-4 font-bold">Subcategoria</span>
          <div class="grid grid-cols-12 gap-4">
            <div class="flex items-center col-span-6 gap-2" v-for="subcategory in subcategories" :key="subcategory">
              <RadioButton :id="subcategory.id" v-model="product.subcategory" name="subcategory"
                :value="subcategory.name" @change="loadAttributes" />
              <label :for="subcategory.id">{{ subcategory.name }}</label>
            </div>
          </div>
        </div>
        <div>
          <span class="block mb-4 font-bold">Atributos</span>
          <div class="grid grid-cols-12 gap-4">
            <div class="flex items-center col-span-6 gap-2" v-for="attribute in attributes" :key="attribute.id">
              <RadioButton :id="attribute.id" v-model="product.attribute" name="subcategory" :value="product.attribute"
                @change="loadAttributes" />
              <label :for="attribute.id">{{ attribute.name }}</label>
            </div>
          </div>
        </div>
        <div class="w-full space-y-4">
          <div>
            <label for="price" class="block font-bold">Preço</label>
            <InputNumber id="price" v-model="product.price" mode="currency" currency="BRL" locale="pt-BR" fluid />
          </div>
          <div>
            <label for="quantity" class="block font-bold">Quantidade</label>
            <InputNumber id="quantity" v-model="product.quantity" integeronly fluid />
          </div>
        </div>
      </div>
      <template #footer>
        <Button label="Cancelar" icon="pi pi-times" text @click="hideDialog" />
        <Button label="Salvar" icon="pi pi-check" @click="saveProduct" />
      </template>
    </Dialog>

    <Dialog v-model:visible="deleteProductDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
      <div class="flex items-center gap-4">
        <i class="pi pi-exclamation-triangle !text-3xl" />
        <span v-if="product">Você tem certeza que quer excluir o produto <b>{{ product.name }}</b>?</span>
      </div>
      <template #footer>
        <Button label="Não" icon="pi pi-times" text @click="deleteProductDialog = false" />
        <Button label="Sim" icon="pi pi-check" @click="deleteProduct" />
      </template>
    </Dialog>

    <Dialog v-model:visible="deleteProductsDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
      <div class="flex items-center gap-4">
        <i class="pi pi-exclamation-triangle !text-3xl" />
        <span v-if="product">Are you sure you want to delete the selected products?</span>
      </div>
      <template #footer>
        <Button label="No" icon="pi pi-times" text @click="deleteProductsDialog = false" />
        <Button label="Yes" icon="pi pi-check" text @click="deleteSelectedProducts" />
      </template>
    </Dialog>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { FilterMatchMode } from 'primevue/api'
import { useToast } from 'primevue/usetoast'
import DataTable from 'primevue/datatable'
import Tag from 'primevue/tag'
import Column from 'primevue/column'
import Button from 'primevue/button'
import Toolbar from 'primevue/toolbar'
import Dialog from 'primevue/dialog'
import InputText from 'primevue/inputtext'
import axios from 'axios'
import Toast from 'primevue/toast'
import Textarea from 'primevue/textarea'
import RadioButton from 'primevue/radiobutton'
import InputNumber from 'primevue/inputnumber'

onMounted(() => {
  axios
    .get(`${import.meta.env.VITE_ROOT_API}/products`)
    .then((response) => {
      products.value = response.data
    })
    .catch((error) => {
      console.log(error)
    })
})

const toast = useToast()
const dt = ref()
const products = ref()
const productDialog = ref(false)
const deleteProductDialog = ref(false)
const deleteProductsDialog = ref(false)
const product = ref({})
const selectedProducts = ref()
const filters = ref({
  global: { value: null, matchMode: FilterMatchMode.CONTAINS }
})
const submitted = ref(false)
const statuses = ref([
  { label: 'EM ESTOQUE', value: 'instock' },
  { label: 'BAIXO ESTOQUE', value: 'lowstock' },
  { label: 'FORA DE ESTOQUE', value: 'outofstock' }
])
const categories = ref()
const subcategories = ref()
const attributes = ref()

const loadAttributes = (e) => {
  console.log(e.target.value)
  axios.get(
    `${import.meta.env.VITE_ROOT_API}/products/attributes/${selectedSubcategory.value.code}`
  )
}

const formatCurrency = (value) => {
  if (value) {
    return value.toLocaleString('pt-BR', {
      style: 'currency',
      currency: 'BRL'
    })
  }
  return ''
}
const openNew = () => {
  product.value = {}
  submitted.value = false
  productDialog.value = true
}
const hideDialog = () => {
  productDialog.value = false
  submitted.value = false
}
const editProduct = (prod) => {
  product.value = { ...prod }
  productDialog.value = true
  axios
    .get(`${import.meta.env.VITE_ROOT_API}/products/categories`)
    .then((response) => {
      categories.value = response.data.categories.filter((category) => category.parent_id === null)
      subcategories.value = response.data.categories.filter(
        (category) => category.parent_id !== null
      )
    })
    .catch((error) => {
      console.log(error)
    })
  axios
    .get(`${import.meta.env.VITE_ROOT_API}/products/${product.value.id}`)
    .then((response) => {
      // console.log(response)
    })
    .catch((error) => {
      console.log(error)
    })
  axios
    .get(`${import.meta.env.VITE_ROOT_API}/products/attributes/${product.value.subcategory_id}`)
    .then((response) => {
      const res = response.data
      attributes.value = res.attributes.map((attribute) => {
        return {
          ...attribute,
          options: attribute.attribute_options.map((option) => ({
            name: option.value,
            code: option.id
          }))
        }
      })
    })
    .catch((error) => {
      console.log(error.data)
    })
}
const confirmDeleteProduct = (prod) => {
  product.value = prod
  deleteProductDialog.value = true
}
const deleteProduct = () => {
  products.value = products.value.filter((val) => val.id !== product.value.id)
  deleteProductDialog.value = false
  axios
    .delete(`${import.meta.env.VITE_ROOT_API}/products/${product.value.id}`)
    .then((response) => {
      toast.add({
        severity: 'success',
        summary: 'Sucesso!',
        detail: 'Produto apagado',
        life: 3000
      })
    })
    .catch((error) => {
      console.log(error)
    })
}

const exportCSV = () => {
  dt.value.exportCSV()
}
const confirmDeleteSelected = () => {
  deleteProductsDialog.value = true
}
const deleteSelectedProducts = () => {
  products.value = products.value.filter((val) => !selectedProducts.value.includes(val))
  deleteProductsDialog.value = false
  selectedProducts.value = null
}

const getStatusLabel = (status) => {
  switch (status) {
    case 'EM ESTOQUE':
      return 'success'

    case 'BAIXO ESTOQUE':
      return 'warn'

    case 'FORA DE ESTOQUE':
      return 'danger'

    default:
      return null
  }
}
</script>
