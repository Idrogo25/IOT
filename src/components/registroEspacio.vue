<script setup>
import { ref } from "vue"
import { sabeID } from "@/firebase";
const nombre = ref("")
function guardar() {
    //Verificamos que no está vacío el nombre del espacio
    if (!nombre.value) {
        alert("Debes poner el nombre del espacio")
        return;
    }
    sabeID(nombre.value, "espacios", {})
    alert("Se ha agregado correctamente el espacio")
}
</script>
<template>
    <div>
        <form class="p-1">
            <input type="text" v-model="nombre" placeholder="Nombre">
            <button class="btn btn-success" @click.prevent="guardar">Agregar espacios</button>
        </form>
    </div>
</template>

<style lang="scss" scoped></style>