<script setup>
import { ref } from 'vue';

const nombre = ref('');
const apellidoPaterno = ref('');
const apellidoMaterno = ref('');
const fechaNacimiento = ref('');
const rfc = ref('');

function generarRFC() {
    const nombres = nombre.value.trim().split(' ');
    const primerNombre = nombres[0];
    const apPaterno = normalizarCadena(apellidoPaterno.value.trim());
    const apMaterno = normalizarCadena(apellidoMaterno.value.trim());
    const fecha = fechaNacimiento.value.split('-');

    const iniciales = ((apPaterno[0] || '') + (apPaterno.match(/[aeiou]/i)?.[0] || '') + (apMaterno[0] || '') + (primerNombre[0] || '')).toUpperCase();

    const anio = fecha[0]?.slice(-2) || '';
    const mes = fecha[1] || '';
    const dia = fecha[2] || '';

    const homoclave = generarHomoclave();

    rfc.value = `${iniciales}${anio}${mes}${dia}${homoclave}`;
}

function generarHomoclave() {
    const letras = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
    const numeros = '0123456789';

    const letra1 = letras.charAt(Math.floor(Math.random() * letras.length));
    const letra2 = letras.charAt(Math.floor(Math.random() * letras.length));
    const numero = numeros.charAt(Math.floor(Math.random() * numeros.length));

    return `${letra1}${letra2}${numero}`;
}

function normalizarCadena(cadena) {
    return cadena.normalize('NFD').replace(/\p{Diacritic}/gu, '');
}
</script>

<template>
    <div class="container mx-auto p-4 max-w-[60rem]">
        <h2 class="text-xl font-semibold mb-4">Generador de RFC</h2>

        <div class="grid gap-4">
            <div>
                <label for="nombre">Nombre</label>
                <input type="text" v-model="nombre" id="nombre" class="border p-2 w-full" placeholder="Nombre(s)" />
            </div>

            <div>
                <label for="apellidoPaterno">Apellido Paterno</label>
                <input type="text" v-model="apellidoPaterno" id="apellidoPaterno" class="border p-2 w-full"
                    placeholder="Apellido Paterno" />
            </div>

            <div>
                <label for="apellidoMaterno">Apellido Materno</label>
                <input type="text" v-model="apellidoMaterno" id="apellidoMaterno" class="border p-2 w-full"
                    placeholder="Apellido Materno" />
            </div>

            <div>
                <label for="fechaNacimiento">Fecha de Nacimiento</label>
                <input type="date" v-model="fechaNacimiento" id="fechaNacimiento" class="border p-2 w-full" />
            </div>

            <button @click="generarRFC" class="bg-green-500 text-white py-2 px-4 rounded">Generar RFC</button>
        </div>


        <div v-if="rfc" class="mt-4">
            <h3 class="text-lg font-semibold">RFC Generado:</h3>
            <p class="text-lg">{{ rfc }}</p>
        </div>
    </div>
</template>
