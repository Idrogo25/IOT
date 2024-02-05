<script setup>
import { deleteData, getDataChanged, updateData, getDataDocument} from "@/firebase";
import { ref } from "vue";

const nombreEjecutor = ref("");
const nombreSensor = ref("");
const espacios = ref([]);
const sensores = ref([]);


const generateUniqueId = () => {
  return new Date().getTime(); // Utilizando la marca de tiempo actual como ID único
};

const callback = (ids) => {
  espacios.value = [];
  sensores.value = [];
  ids.forEach((doc) => {
    espacios.value.push({
      id: doc.id,
      ...doc.data(),
    });
  });
};

getDataChanged("espacios", callback);

const eliminarEspacio = (espacio) => {
  try {
    deleteData(espacio, "espacios");
  } catch (error) {
    console.error(error);
  }
};

const agregaSensor = async (espacio) => {
  try {
    if (!nombreSensor.value) {
      alert("¡El nombre del Sensor es obligatorio");
      return;
    }

    const documento = await getDataDocument(espacio, "espacios");
    const sensoresActuales = documento.exists() ? (documento.data().Sensores || []) : [];

    const nuevoSensor = {
      id: generateUniqueId(),
      nombre: nombreSensor.value,
      estado: "Sin asignar",
    };

    sensoresActuales.push(nuevoSensor);

    await updateData(espacio, "espacios", { Sensores: sensoresActuales });
    nombreSensor.value = "";
    alert("!Se ha agregado correctamente el sensor!");
  } catch (error) {
    console.error(error);
  }
};

const agregaEjecutor = async (espacio) => {
  try {
    if (!nombreEjecutor.value) {
      alert("¡El nombre del Sensor es obligatorio");
      return;
    }

    const documento = await getDataDocument(espacio, "espacios");
    const ejecutoresActuales = documento.exists() ? documento.data().Ejecutores || [] : [];

    const nuevoEjecutor = {
      id: generateUniqueId(),
      nombre: nombreEjecutor.value,
      estado: false,
    };

    ejecutoresActuales.push(nuevoEjecutor);

    await updateData(espacio, "espacios", { Ejecutores: ejecutoresActuales });
    nombreEjecutor.value = "";
    alert("!Se ha agregado correctamente el ejecutor!");
  } catch (error) {
    console.error(error);
  }
};
const toggleEstado = async (espacioId, ejecutor) => {
  try {
    // Obtener el documento actual
    const documento = await getDataDocument(espacioId, "espacios");
    // Obtener el arreglo Ejecutores actual
    const ejecutoresActuales = documento.exists() ? documento.data().Ejecutores || [] : [];
    // Encontrar el ejecutor en el arreglo y actualizar su estado
    const ejecutorIndex = ejecutoresActuales.findIndex(e => e.Nombre === ejecutor.Nombre);
    if (ejecutorIndex !== -1) {
      ejecutoresActuales[ejecutorIndex].estado = !ejecutoresActuales[ejecutorIndex].estado;
      // Actualizar el documento con el nuevo arreglo de Ejecutores
      await updateData(espacioId, "espacios", { Ejecutores: ejecutoresActuales });
    }
  } catch (error) {
    console.error(error);
  }
};
//Función que modifica el estado del sensor 
const modificarNombreSensor = async (espacioId, sensorNombre, nuevoNombre) => {
  try {
  
    const documento = await getDataDocument(espacioId, "espacios");
    const sensoresActuales = documento.exists() ? documento.data().Sensores || [] : [];
    const sensorIndex = sensoresActuales.findIndex(el => el.nombre === sensorNombre);

    if (sensorIndex !== -1) {
      sensoresActuales[sensorIndex].nombre = nuevoNombre;
      await updateData(espacioId, "espacios", { Sensores: sensoresActuales });
    }
  } catch (error) {
    console.error(error);
  }
};
const modificarNombreEjecutor = async(espacioId, ejecutorNombre, nuevoNombre)=>{
  try{
    const documento = await getDataDocument(espacioId,"espacios")
    const ejecutoresActuales = documento.exists() ? documento.data().Ejecutores || [] : []
    const sensorIndex = ejecutoresActuales.findIndex(el => el.nombre === ejecutorNombre);
    if (sensorIndex !== -1) {
      ejecutoresActuales[sensorIndex].nombre = nuevoNombre;
      await updateData(espacioId, "espacios", { Ejecutores: ejecutoresActuales});
    }
  }catch(e){
    console.log(e)
  }
}
//Función que elimina ejecutor
const eliminarEjecutor = async (espacioId, ejecutorNombre) => {
  try {
    // Obtener el documento actual
    const documento = await getDataDocument(espacioId, "espacios");

    // Obtener el arreglo de ejecutores actual
    const ejecutoresActuales = documento.exists() ? documento.data().Ejecutores || [] : [];

    // Filtrar el ejecutor a eliminar
    const nuevosEjecutores = ejecutoresActuales.filter(e => e.nombre !== ejecutorNombre);

    // Actualizar el documento con el nuevo arreglo de ejecutores
    await updateData(espacioId, "espacios", { Ejecutores: nuevosEjecutores });
  } catch (error) {
    console.error(error);
  }
};
//Función que elimina sensor 
const eliminarSensor = async (espacioId, sensorNombre) => {
  try {
    const documento = await getDataDocument(espacioId, "espacios");
    const sensoresActuales = documento.exists() ? documento.data().Sensores || [] : [];

    // Filtrar el sensor a eliminar
    const nuevosSensores = sensoresActuales.filter(s => s.nombre !== sensorNombre);

    // Actualizar el documento con el nuevo arreglo de sensores
    await updateData(espacioId, "espacios", { Sensores: nuevosSensores });
  } catch (error) {
    console.error(error);
  }
};
</script>
<template>
  <div>
    <ul class="espacios">
      <li v-for="espacio in espacios" :key="espacio">
        <nav class="navbar bg-body-tertiary">
          <div class="container">
            <a class="navbar-brand">
              <h4>{{ espacio.id }}</h4>
            </a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarText"
              aria-controls="navbarText" aria-expanded="true" aria-label="Toggle navigation">
              <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse show" id="navbarText">
              <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                <li class="nav-item">
                  <a class="nav-link active" aria-current="page">
                    <h5>Sensores</h5>
                    <p v-for="sensor in espacio.Sensores" :key="sensor">
                      Sensor: {{ sensor.nombre }} / Estado: {{ sensor.estado }}
                      <input v-if="espacio.Sensores && espacio.Sensores.length !== 0" type="text" v-model="estadoSensor"
                        placeholder="Modificar Nombre">
                      <button class="btn btn-primary"
                        @click="modificarNombreSensor(espacio.id, sensor.nombre, estadoSensor)">
                        Modificar Nombre
                      </button>
                      <button class="btn btn-danger btn-sm" @click="eliminarSensor(espacio.id, sensor.nombre)">
                        Eliminar Sensor
                      </button>
                    </p>
                  </a>
                  <a class="nav-link active" aria-current="page">
                    <h5>Ejecutores</h5>
                    <div v-for="ejecutor in espacio.Ejecutores" :key="ejecutor">
                      <div> {{ ejecutor.nombre }}: {{ ejecutor.estado ? "Encendido" : "Apagado" }}
                        <div class="form-check form-switch">
                          <input class="form-check-input" type="checkbox" role="switch" id="flexSwitchCheckDefault"
                            :checked="ejecutor.estado" @change="() => toggleEstado(espacio.id, ejecutor)" />
                          <label class="form-check-label" for="flexSwitchCheckDefault">
                            {{ ejecutor.estado ? 'Apagar' : 'Encender' }}
                          </label>
                        </div>
                        <button class="btn btn-danger btn-sm" @click="eliminarEjecutor(espacio.id, ejecutor.Nombre)">
                          Eliminar Ejecutor
                        </button>
                        <input type="text" v-model="nuevoNombreEjecutor">
                        <button  class="btn btn-primary" @click="modificarNombreEjecutor(espacio.id, ejecutor.nombre, nuevoNombreEjecutor)">
                          Nuevo nombre
                        </button>
                      </div>
                    </div>
                  </a>
                </li>
              </ul>
            </div>
          </div>
        </nav>
        <button class="btn btn-danger" @click="eliminarEspacio(espacio.id)">Eliminar</button>
        <button class="btn btn-primary" @click="agregaEjecutor(espacio.id)">Agregar ejecutor</button>
        <button class="btn btn-primary" @click="agregaSensor(espacio.id)">Agregar sensor</button>
        <hr>
      </li>
      <p>Escriba aquí el nombre del componente y pulsa el botón donde quieras agregar el componente.</p>
      <div class="inputBox">
        <input placeholder="Sensor" type="text" v-model="nombreSensor" required="">
        <span>Nombre Sensor:</span>
      </div>
      <div class="inputBox">
        <input placeholder="Ejecutor" type="text" v-model="nombreEjecutor" required="">
        <span>Nombre Ejecutor:</span>
      </div>
    </ul>
  </div>
</template>

<style lang="css" scoped>
.espacios {
  list-style: none;
  font-size: larger;

}

.inputBox {
  position: relative;
}

.inputBox input {
  padding: 15px 20px;
  outline: none;
  background: transparent;
  border-radius: 5px;
  color: #212121;
  border: 1px solid#212121;
  font-size: 1em;
}

.inputBox span {
  position: absolute;
  left: 0;
  font-size: 0.7em;
  transform: translateX(14px) translateY(-7.5px);
  padding: 0 6px 1px 5px;
  border-radius: 2px;
  background: #e8e8e8;
  letter-spacing: 1px;
  border: 1px solid #212121;
  color: #212121;
}
</style>