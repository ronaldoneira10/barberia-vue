<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('servicios-barberia', [])

const mostrarModal = ref(false)

const formulario = ref({
  nombre: '',
  servicio: '',
  barbero: '',
  fecha: '',
  hora: '',
  precio: '',
  metodoPago: '',
  estadoPago: '',
  calificacion: '',
  observaciones: ''
})

function abrirModal() {
  mostrarModal.value = true
}

function cerrarModal() {
  mostrarModal.value = false
}

function guardarServicio() {
  servicios.value.push({
    id: Date.now(),
    nombre: formulario.value.nombre,
    servicio: formulario.value.servicio,
    barbero: formulario.value.barbero,
    fecha: formulario.value.fecha,
    hora: formulario.value.hora,
    precio: formulario.value.precio,
    metodoPago: formulario.value.metodoPago,
    estadoPago: formulario.value.estadoPago,
    calificacion: formulario.value.calificacion,
    observaciones: formulario.value.observaciones
  })

  formulario.value = {
    nombre: '',
    servicio: '',
    barbero: '',
    fecha: '',
    hora: '',
    precio: '',
    metodoPago: '',
    estadoPago: '',
    calificacion: '',
    observaciones: ''
  }

  cerrarModal()
}
</script>

<template>
  <div class="app">

    <header class="encabezado">
      <div>
        <h1>✂️ Barbería Don Ramiro</h1>
        <p>Registro de servicios</p>
      </div>

      <button class="boton-nuevo" @click="abrirModal">
        + Nuevo servicio
      </button>
    </header>

    <main class="contenido">

      <div v-if="servicios.length === 0" class="sin-servicios">

        <div class="icono">💈</div>

        <h2>No hay servicios registrados</h2>

        <p>Agregue el primer servicio de la barbería.</p>

        <button class="boton-nuevo" @click="abrirModal">
          + Registrar servicio
        </button>

      </div>

      <div v-else class="lista-servicios">

        <div
          v-for="servicio in servicios"
          :key="servicio.id"
          class="servicio-card"
        >

          <h2>{{ servicio.nombre }}</h2>

          <p>
            <strong>Servicio:</strong>
            {{ servicio.servicio }}
          </p>

          <p>
            <strong>Barbero:</strong>
            {{ servicio.barbero }}
          </p>

          <p>
            <strong>Fecha:</strong>
            {{ servicio.fecha }}
          </p>

          <p>
            <strong>Hora:</strong>
            {{ servicio.hora }}
          </p>

          <p>
            <strong>Precio:</strong>
            ${{ servicio.precio }}
          </p>

          <p>
            <strong>Método de pago:</strong>
            {{ servicio.metodoPago }}
          </p>

          <p>
            <strong>Estado del pago:</strong>
            {{ servicio.estadoPago }}
          </p>

          <p>
            <strong>Calificación:</strong>
            {{ servicio.calificacion }}/5
          </p>

          <p v-if="servicio.observaciones !== ''">
            <strong>Observaciones:</strong>
            {{ servicio.observaciones }}
          </p>

        </div>

      </div>

    </main>

    <div v-show="mostrarModal" class="modal-fondo">

      <div class="modal">

        <h2>✂️ Nuevo servicio</h2>

        <form @submit.prevent="guardarServicio">

          <label>Nombre del cliente</label>

          <input
            v-model="formulario.nombre"
            type="text"
            required
          >

          <label>Tipo de servicio</label>

          <select v-model="formulario.servicio" required>
            <option value="">Seleccione</option>
            <option>Corte clásico</option>
            <option>Corte moderno</option>
            <option>Barba</option>
            <option>Corte + barba</option>
            <option>Cejas</option>
            <option>Tinte</option>
          </select>

          <label>Barbero</label>

          <select v-model="formulario.barbero" required>
            <option value="">Seleccione</option>
            <option>Don Ramiro</option>
            <option>Carlos</option>
            <option>Miguel</option>
          </select>

          <label>Fecha</label>

          <input
            v-model="formulario.fecha"
            type="date"
            required
          >

          <label>Hora</label>

          <input
            v-model="formulario.hora"
            type="time"
            required
          >

          <label>Precio cobrado</label>

          <input
            v-model="formulario.precio"
            type="number"
            min="1"
            required
          >

          <label>Método de pago</label>

          <select v-model="formulario.metodoPago" required>
            <option value="">Seleccione</option>
            <option>Efectivo</option>
            <option>Transferencia</option>
            <option>Tarjeta</option>
          </select>

          <label>Estado del pago</label>

          <select v-model="formulario.estadoPago" required>
            <option value="">Seleccione</option>
            <option>Pagado</option>
            <option>Pendiente</option>
            <option>Fiado</option>
          </select>

          <label>Calificación</label>

          <select v-model="formulario.calificacion" required>
            <option value="">Seleccione</option>
            <option value="1">⭐ 1 estrella</option>
            <option value="2">⭐⭐ 2 estrellas</option>
            <option value="3">⭐⭐⭐ 3 estrellas</option>
            <option value="4">⭐⭐⭐⭐ 4 estrellas</option>
            <option value="5">⭐⭐⭐⭐⭐ 5 estrellas</option>
          </select>

          <label>Observaciones</label>

          <textarea
            v-model="formulario.observaciones"
            rows="3"
          ></textarea>

          <div class="botones">

            <button
              type="button"
              class="boton-cancelar"
              @click="cerrarModal"
            >
              Cancelar
            </button>

            <button
              type="submit"
              class="boton-guardar"
            >
              Registrar servicio
            </button>

          </div>

        </form>

      </div>

    </div>

  </div>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #f4f1ed;
  color: #29231f;
}

button,
input,
select,
textarea {
  font-family: inherit;
}

button {
  cursor: pointer;
}

.app {
  min-height: 100vh;
}

.encabezado {
  background: #241b17;
  color: white;
  padding: 25px 7%;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.encabezado h1 {
  margin: 0;
  font-size: 28px;
}

.encabezado p {
  margin: 5px 0 0;
  color: #d8c9bd;
}

.boton-nuevo {
  border: none;
  background: #c89455;
  color: white;
  padding: 12px 18px;
  border-radius: 8px;
  font-weight: bold;
}

.contenido {
  padding: 30px 7%;
}

.sin-servicios {
  background: white;
  padding: 60px 20px;
  text-align: center;
  border-radius: 12px;
}

.icono {
  font-size: 50px;
}

.lista-servicios {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
}

.servicio-card {
  background: white;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.08);
}

.servicio-card h2 {
  margin-top: 0;
}

.modal-fondo {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.modal {
  background: white;
  width: 100%;
  max-width: 650px;
  max-height: 90vh;
  overflow-y: auto;
  padding: 25px;
  border-radius: 12px;
}

.modal h2 {
  margin-top: 0;
}

form {
  display: grid;
  gap: 8px;
}

input,
select,
textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
}

.botones {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 15px;
}

.boton-cancelar {
  background: #777;
}

.boton-guardar {
  background: #241b17;
}

@media (max-width: 700px) {

  .encabezado {
    flex-direction: column;
    gap: 15px;
    align-items: stretch;
  }

  .lista-servicios {
    grid-template-columns: 1fr;
  }

}
</style>

