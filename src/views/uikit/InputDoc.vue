<script setup>
import { useToast } from 'primevue/usetoast';
import { ref, onBeforeMount, onMounted, computed, watch } from 'vue';

const toast = useToast();
const visibleUpdate = ref(false);
const visibleDelete = ref(false);


const nomp = ref('');
const cantidad = ref('');
const precioUnitario = ref('');
const tablaCompras = ref([]);

const productoItem = ref({
    cns: null,
    nomProducto: null,
    cantidad: null,
    precioUnitario: null,
    precioParcial: null
});

// Guarda los productos registrados 
const guardarProductos = () => {
    localStorage.setItem('productos', JSON.stringify(tablaCompras.value));
};

const cargarProductos = () => {
    const productosGuardados = localStorage.getItem('productos');
    if (productosGuardados) {
        tablaCompras.value = JSON.parse(productosGuardados);
    } else {
        // Si no hay nuevos productos registrados, inicializar con productos que ya estan por defectos
        tablaCompras.value = [
            { cns: 1, nomProducto: "Impresora LaserJet Color", cantidad: 2, precioUnitario: 5200.00, precioParcial: 10400.00 },
            { cns: 2, nomProducto: "Monitor LED 31 plg.", cantidad: 3, precioUnitario: 1700.00, precioParcial: 5100.00 }
        ];
    }
};

onMounted(() => {
    cargarProductos();
});

watch([tablaCompras], () => {
    guardarProductos(); // Guarda los datos cada vez que se actualiza tablaCompras
}, { deep: true });

watch([visibleDelete, visibleUpdate], ([newDelete, newUpdate]) => {
    if (!newDelete && !newUpdate) {
        limpiarFormulario();
    }
});



async function editarProducto(row) {
    visibleUpdate.value = true;

    nomp.value = row.nomProducto;
    cantidad.value = row.cantidad;
    precioUnitario.value = row.precioUnitario;


    productoItem.value = row;

}

async function updateProducto() {
    const index = tablaCompras.value.findIndex(producto => producto.cns === productoItem.value.cns);

    if (index !== -1) {

        tablaCompras.value[index] = {
            ...productoItem.value,
            nomProducto: nomp.value,
            cantidad: parseFloat(cantidad.value),
            precioUnitario: parseFloat(precioUnitario.value),
            precioParcial: parseFloat(cantidad.value) * parseFloat(precioUnitario.value)
        };

        toast.add({ severity: 'success', summary: 'Producto Actualizado', detail: 'El producto ha sido actualizado con éxito.', life: 3000 });

        visibleUpdate.value = false;


    } else {
        toast.add({ severity: 'error', summary: 'Error', detail: 'No se pudo actualizar el producto.', life: 3000 });
    }
};

async function eliminarProducto(row) {
    productoItem.value = row;
    visibleDelete.value = true;

}

async function deleteProducto(row) {
    if (productoItem.value) {

        tablaCompras.value = tablaCompras.value.filter(producto => producto.cns !== productoItem.value.cns);


        toast.add({ severity: 'success', summary: 'Producto Eliminado', detail: 'El producto ha sido eliminado con éxito.', life: 3000 });

        visibleDelete.value = false;

    }
};



async function registrarCompra() {
    if (
        productoItem.value.nomProducto.trim() !== '' &&
        productoItem.value.cantidad.trim() !== '' &&
        productoItem.value.precioUnitario.trim() !== ''
    ) {
        tablaCompras.value.push({
            cns: tablaCompras.value.length + 1,
            nomProducto: productoItem.value.nomProducto,
            cantidad: parseFloat(productoItem.value.cantidad),
            precioUnitario: parseFloat(productoItem.value.precioUnitario),
            precioParcial: parseFloat(productoItem.value.cantidad) * parseFloat(productoItem.value.precioUnitario)
        });

        toast.add({ severity: 'success', summary: 'Confirmación', detail: 'Nueva compra registrada', life: 3000 });

        limpiarFormulario();
    } else {

        toast.add({ severity: 'warn', summary: 'Campos incompletos', detail: 'Por favor, completa todos los campos antes de registrar.', life: 3000 });
    }




};


function limpiarFormulario() {
    productoItem.value = {
        nomProducto: '',
        cantidad: '',
        precioUnitario: '',
    };

    nomp.value = '';
    cantidad.value = '';
    precioUnitario.value = '';





}

function formatoMoneda(value) {
    if (value) return value.toLocaleString('es-MX', { style: 'currency', currency: 'MXN' });
    return;
}

const subtotal = computed(() => {
    return tablaCompras.value.reduce((acc, producto) => acc + producto.precioParcial, 0);
});

const iva = computed(() => {
    return subtotal.value * 0.16;
});

const totalTotal = computed(() => {
    return subtotal.value + iva.value;
});
</script>

<template>
    <div class="p-grid">
        <Toast />
        <div class="p-col-12">
            <div class="card">
                <Panel header="PUNTO DE VENTA (POS)" style="height: 100%">
                    <Toolbar class="p-mb-4">
                        <template v-slot:start>
                            <InputText type="text" size="40" placeholder="Nombre del producto..."
                                v-model="productoItem.nomProducto" />
                            <InputText class="ml-8" type="text" placeholder="Cant" v-model="productoItem.cantidad" />
                            <InputText class="ml-8" type="text" placeholder="$ Precio U."
                                v-model="productoItem.precioUnitario" />
                        </template>
                        <template v-slot:end>
                            <Button label="Registrar" icon="pi pi-plus" class="p-button-success p-mr-2"
                                @click="registrarCompra" />
                        </template>
                    </Toolbar>
                    <br>

                    <DataTable :value="tablaCompras" v-model:selection="productoItem" class="p-datatable-gridlines"
                        dataKey="cns" :rows="5"
                        paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                        :rowsPerPageOptions="[5, 10, 25]"
                        currentPageReportTemplate="Visualizando {last} de {totalRecords} productos"
                        style="margin-bottom: 20px" :paginator="true" responsiveLayout="scroll">
                        <Column field="cns" header="Cns" :sortable="true" style="width:50px"></Column>
                        <Column field="nomProducto" header="Nombre del Producto" style="width:370px"></Column>
                        <Column field="precioUnitario" header="Precio U." dataType="numeric" style="width:80px">
                            <template #body="slotProps">
                                {{ formatoMoneda(slotProps.data.precioUnitario) }}
                            </template>
                        </Column>
                        <Column field="cantidad" header="Cant." style="width:60px;text-align:center"></Column>
                        <Column field="precioParcial" header="Precio P." style="width:80px">
                            <template #body="slotProps">
                                {{ formatoMoneda(slotProps.data.precioParcial) }}
                            </template>
                        </Column>
                        <Column style="width:140px">
                            <template #header>
                                Acciones
                            </template>
                            <template #body="slotProps">
                                <Button icon="pi pi-pencil" type="button" class="p-button-success p-mr-2 p-mb-1"
                                    @click="editarProducto(slotProps.data)" />
                                <Dialog v-model:visible="visibleUpdate" modal header="Actualizar datos de un producto"
                                    :style="{ width: '45vw' }">
                                    <p>
                                    <div class="flex gap-2">
                                        <div class="flex-auto">
                                            <label for="nomp"><strong>Nombre del producto: </strong></label>
                                            <InputText class="ml-2" id="nomp" v-model="nomp" />
                                        </div>
                                        <div class="flex-auto">
                                            <label for="precioUnitario"><strong>Precio Unitario: </strong></label>
                                            <InputText class="ml-2" id="precioUnitario" v-model="precioUnitario" />
                                        </div>
                                        <div class="flex-auto">
                                            <label for="cantidad"><strong>Cantidad: </strong></label>
                                            <InputText class="ml-2" id="cantidad" v-model="cantidad" />
                                        </div>
                                    </div>
                                    <br>
                                    </p>
                                    <template #footer>
                                        <Button label="Actualizar" icon="pi pi-replay" @click="updateProducto" />
                                    </template>
                                </Dialog>
                                <Button icon="pi pi-trash" type="button" class="p-button-danger p-mb-1"
                                    @click="eliminarProducto(slotProps.data)" />
                                <Dialog v-model:visible="visibleDelete" modal
                                    header="Estas seguro que quieres borrar este producto?" :style="{ width: '30vw' }">
                                    <p>
                                        <br>
                                    </p>
                                    <template #footer>
                                        <Button label="Eliminar" severity="warning" icon="pi pi-check"
                                            @click="deleteProducto(slotProps.data)" autofocus />
                                    </template>
                                </Dialog>
                            </template>
                        </Column>
                    </DataTable>
                    <br>
                    <Toolbar class="p-mb-4">
                        <template v-slot:start></template>
                        <template v-slot:end>
                            <label for="subtotal">Subtotal: </label>
                            <InputText type="text" placeholder="$ " :value="formatoMoneda(subtotal)" disabled />
                            <label for="iva">IVA (16%)</label>
                            <InputText type="text" placeholder="$ " :value="formatoMoneda(iva)" disabled />
                            <label for="total">Total: </label>
                            <InputText type="text" placeholder="$ " :value="formatoMoneda(totalTotal)" disabled />
                        </template>
                    </Toolbar>
                </Panel>
            </div>
        </div>
    </div>
</template>
