<script setup>
import { ref } from 'vue';
import axios from 'axios';

const nombre = ref("");
const color = ref("");
const capacidad = ref("");
const idProducto = ref("");
const productos = ref([]);
const mostrarTabla = ref(false);
const mostrarModal = ref(false);
const mensajeError = ref("");

//verifica que todos los campos contengan datos, y si estan vacios enviara un mensaje de campos obligatorios
const validarFormulario = () => {
    if (!nombre.value || !color.value || !capacidad.value || !idProducto.value) {
        mensajeError.value = "Campos obligatorios";
        mostrarModal.value = true;
        return false;
    }
    mostrarModal.value = false;
    return true;
};

async function consultarDatos() {
    if (!validarFormulario()) {
        return;
    }

    try {
        const response = await axios.get("https://api.restful-api.dev/objects/1");

        if (response.data) {
            const nuevoProducto = {
                idProducto: response.data.id,
                nombre: response.data.name,
                color: response.data.data.color,
                capacidad: response.data.data.capacity
            };

            productos.value = [nuevoProducto];
            mostrarTabla.value = true;
        } else {
            console.error("La respuesta no tiene la estructura esperada");
        }
    } catch (error) {
        console.error("Error al consultar el API:", error);
    }
};
</script>

<template>
    <div class="container mx-auto p-4 max-w-[60rem]">
        <h2 class="text-xl font-semibold mb-4">Consumir API REST Producto</h2>

        <!-- Mensaje de error si falta algún campo -->
        <div v-if="mostrarModal" class="bg-red-100 text-red-700 p-4 rounded mb-4">
            {{ mensajeError }}
        </div>

        <div class="grid gap-4">
            <div>
                <label for="nombre">Nombre del Producto:</label>
                <input type="text" v-model="nombre" id="nombre" class="border p-2 w-full rounded-md"
                    placeholder="Nombre del producto" required />
            </div>

            <div>
                <label for="color">Color:</label>
                <input type="text" v-model="color" id="color" class="border p-2 w-full rounded-md"
                    placeholder="Color" />
            </div>

            <div>
                <label for="capacidad">Capacidad:</label>
                <input type="text" v-model="capacidad" id="capacidad" class="border p-2 w-full rounded-md"
                    placeholder="Capacidad" />
            </div>

            <div>
                <label for="idProducto">Id Producto:</label>
                <input type="text" v-model="idProducto" id="idProducto" class="border p-2 w-full rounded-md" />
            </div>

            <button @click="consultarDatos"
                class="bg-green-500 text-white py-2 px-4 rounded flex items-center relative mt-4 hover:bg-green-600 transition duration-200">
                <i class="fas fa-search mr-2"></i>
                <span class="absolute left-1/2 transform -translate-x-1/2">Consultar Datos</span>
            </button>

            <div v-if="mostrarTabla" class="col-12 mt-4">
                <div class="card">
                    <h5>Lista de Productos</h5>
                    <DataTable :value="productos" sortMode="single" class="p-datatable-gridlines" rows="7"
                        :paginator="true" responsiveLayout="scroll">
                        <Column field="nombre" header="Nombre del Producto"></Column>
                        <Column field="color" header="Color" style="min-width:200px"></Column>
                        <Column field="capacidad" header="Capacidad" style="min-width:200px"></Column>
                        <Column field="idProducto" header="ID del Producto" style="min-width:150px"></Column>
                    </DataTable>
                </div>
            </div>
        </div>
    </div>
</template>
