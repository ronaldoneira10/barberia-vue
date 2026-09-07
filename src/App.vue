<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('servicios-barberia', [])

const mostrarModal = ref(false)
const editando = ref(false)
const idEditando = ref(null)
const error = ref('')

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
  editando.value = false
  idEditando.value = null
  error.value = ''

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

  mostrarModal.value = true
}

function cerrarModal() {
  mostrarModal.value = false
  error.value = ''
}


function actualizarPrecio() {
  const precios = {
    'Corte clásico': 15000,
    'Corte moderno': 18000,
    'Barba': 10000,
    'Corte + barba': 25000,
    'Cejas': 5000,
    'Tinte': 30000
  }

  formulario.value.precio = precios[formulario.value.servicio] || ''
}


function validarFormulario() {
  if (formulario.value.nombre.trim() === '') {
    error.value = 'Ingrese el nombre del cliente.'
    return false
  }

  if (formulario.value.servicio === '') {
    error.value = 'Seleccione el tipo de servicio.'
    return false
  }

  if (formulario.value.barbero === '') {
    error.value = 'Seleccione el barbero.'
    return false
  }

  if (formulario.value.fecha === '') {
    error.value = 'Seleccione una fecha.'
    return false
  }

  if (formulario.value.hora === '') {
    error.value = 'Seleccione una hora.'
    return false
  }

  if (formulario.value.precio === '') {
    error.value = 'Ingrese el precio del servicio.'
    return false
  }

  if (Number(formulario.value.precio) <= 0) {
    error.value = 'El precio debe ser mayor que 0.'
    return false
  }

  if (formulario.value.metodoPago === '') {
    error.value = 'Seleccione el método de pago.'
    return false
  }

  if (formulario.value.estadoPago === '') {
    error.value = 'Seleccione el estado del pago.'
    return false
  }

  if (
    formulario.value.hora < '06:00' ||
    formulario.value.hora > '20:00'
  ) {
    error.value = 'La hora debe estar entre las 6:00 AM y las 8:00 PM.'
    return false
  }

  return true
}



function campoInvalido(campo) {
  if (error.value === '') {
    return false
  }

  if (campo === 'nombre') {
    return formulario.value.nombre.trim() === ''
  }

  if (campo === 'servicio') {
    return formulario.value.servicio === ''
  }

  if (campo === 'barbero') {
    return formulario.value.barbero === ''
  }

  if (campo === 'fecha') {
    return formulario.value.fecha === ''
  }

  if (campo === 'hora') {
    return formulario.value.hora === ''
  }

  if (campo === 'precio') {
    return formulario.value.precio === ''
  }

  if (campo === 'metodoPago') {
    return formulario.value.metodoPago === ''
  }

  if (campo === 'estadoPago') {
    return formulario.value.estadoPago === ''
  }

  return false
}



function guardarServicio() {
  error.value = ''

  if (!validarFormulario()) {
    return
  }

  if (editando.value === true) {
    for (let i = 0; i < servicios.value.length; i++) {
      if (servicios.value[i].id === idEditando.value) {
        servicios.value[i] = {
          id: idEditando.value,
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
        }

        break
      }
    }
  } else {
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
  }

  cerrarModal()
}

function editarServicio(servicio) {
  editando.value = true
  idEditando.value = servicio.id
  error.value = ''

  formulario.value = {
    nombre: servicio.nombre,
    servicio: servicio.servicio,
    barbero: servicio.barbero,
    fecha: servicio.fecha,
    hora: servicio.hora,
    precio: servicio.precio,
    metodoPago: servicio.metodoPago,
    estadoPago: servicio.estadoPago,
    calificacion: servicio.calificacion,
    observaciones: servicio.observaciones
  }

  mostrarModal.value = true
}

function eliminarServicio(id) {
  const confirmar = confirm(
    '¿Está seguro de que desea eliminar este servicio?'
  )



  if (confirmar === true) {
    for (let i = 0; i < servicios.value.length; i++) {
      if (servicios.value[i].id === id) {
        servicios.value.splice(i, 1)
        break
      }
    }
  }
}


function calificarServicio(servicio) {
  const calificacion = prompt(
    'Califique el servicio de 1 a 5 estrellas:'
  )

  if (calificacion === null) {
    return
  }

  const numero = Number(calificacion)

  if (numero < 1 || numero > 5 || !Number.isInteger(numero)) {
    alert('La calificación debe ser un número entre 1 y 5.')
    return
  }

  servicio.calificacion = numero
}




function mostrarEstrellas(calificacion) {
  let estrellas = ''

  for (let i = 1; i <= 5; i++) {
    if (i <= Number(calificacion)) {
      estrellas = estrellas + '★'
    } else {
      estrellas = estrellas + '☆'
    }
  }

  return estrellas
}

function iconoPago(metodo) {
  if (metodo === 'Efectivo') {
    return '💵'
  } else if (metodo === 'Transferencia') {
    return '📱'
  } else {
    return '💳'
  }
}

function clasePago(estado) {
  if (estado === 'Pagado') {
    return 'pago-pagado'
  } else {
    return 'pago-pendiente'
  }
}

function claseCalificacion(calificacion) {
  if (Number(calificacion) <= 2) {
    return 'calificacion-baja'
  } else if (Number(calificacion) === 3) {
    return 'calificacion-media'
  } else {
    return 'calificacion-alta'
  }
}
</script>

<template>
  <div class="app">

    <header class="encabezado">
      <div>
        <h1>✂️ Barbería Don Ramiro</h1>
        <p>Registro de servicios</p>
      </div>

      <button
        class="boton-nuevo"
        @click="abrirModal"
      >
        + Nuevo servicio
      </button>
    </header>

    <section class="informacion">

      <div class="tarjeta-info">
        <span>📋</span>

        <div>
          <strong>{{ servicios.length }}</strong>
          <p>Servicios registrados</p>
        </div>
      </div>

      <div class="tarjeta-info">
        <span>💈</span>

        <div>
          <strong>3</strong>
          <p>Barberos</p>
        </div>
      </div>

      <div class="tarjeta-info">
        <span>⭐</span>

        <div>
          <strong>1 - 5</strong>
          <p>Calificación</p>
        </div>
      </div>

    </section>

    <div class="contenido">

      <div
        v-if="servicios.length === 0"
        class="sin-servicios"
      >
        <div class="icono-vacio">💈</div>

        <h2>No hay servicios registrados</h2>

        <p>
          Agregue el primer servicio de la barbería.
        </p>

        <button
          class="boton-nuevo-2"
          @click="abrirModal"
        >
          + Registrar servicio
        </button>
      </div>

      <div
        v-else
        class="lista-servicios"
      >

        <article
          v-for="servicio in servicios"
          :key="servicio.id"
          class="servicio-card"
          :class="clasePago(servicio.estadoPago)"
        >

          <div class="card-cabecera">

            <div>
              <h2>{{ servicio.nombre }}</h2>

              <p class="tipo-servicio">
                {{ servicio.servicio }}
              </p>
            </div>

            <div
              v-if="servicio.estadoPago === 'Pendiente'"
              class="alerta-pago"
            >
              ⚠️ Pendiente
            </div>

            <div
              v-else
              class="pago-ok"
            >
              ✓ Pagado
            </div>

          </div>

          <div class="datos">

            <div class="dato">
              <span>💈 Barbero</span>
              <strong>{{ servicio.barbero }}</strong>
            </div>

            <div class="dato">
              <span>📅 Fecha</span>
              <strong>{{ servicio.fecha }}</strong>
            </div>

            <div class="dato">
              <span>🕐 Hora</span>
              <strong>{{ servicio.hora }}</strong>
            </div>

            <div class="dato">
              <span>💵 Precio</span>

              <strong>
                ${{ Number(servicio.precio).toLocaleString() }}
              </strong>
            </div>

            <div class="dato">
              <span>💳 Pago</span>

              <strong>
                {{ iconoPago(servicio.metodoPago) }}
                {{ servicio.metodoPago }}
              </strong>
            </div>

          </div>

          <div
            class="calificacion"
            :class="claseCalificacion(servicio.calificacion)"
          >
            <span>Calificación:</span>

            <strong>
              {{ mostrarEstrellas(servicio.calificacion) }}
            </strong>

            <span>
              {{ servicio.calificacion }}/5
            </span>

            <span
              v-if="Number(servicio.calificacion) <= 2"
              class="texto-baja"
            >
              ⚠️ Calificación baja
            </span>

            <span
              v-else-if="Number(servicio.calificacion) === 3"
              class="texto-media"
            >
              Regular
            </span>

            <span
              v-else
              class="texto-alta"
            >
              Excelente
            </span>
          </div>

          <div
            v-if="servicio.observaciones !== ''"
            class="observaciones"
          >
            <strong>📝 Observaciones:</strong>

            <p>
              {{ servicio.observaciones }}
            </p>
          </div>

        <div class="acciones">

              <button
                class="boton-editar"
                @click="editarServicio(servicio)"
              >
                ✏️ Editar
              </button>

              <button
                v-if="servicio.calificacion === '' || servicio.calificacion === null"
                class="boton-calificar"
                @click="calificarServicio(servicio)"
              >
                ⭐ Calificar
              </button>

              <button
                class="boton-eliminar"
                @click="eliminarServicio(servicio.id)"
              >
                🗑️ Eliminar
              </button>

            </div>

        </article>

      </div>

    </div>

    <div
      v-show="mostrarModal"
      class="modal-fondo"
    >

      <div class="modal">

        <div class="modal-cabecera">

          <div>
            <h2 v-if="editando">
              ✏️ Editar servicio
            </h2>

            <h2 v-else>
              ✂️ Nuevo servicio
            </h2>

            <p>
              Complete la información del servicio
            </p>
          </div>

          <button
            class="cerrar"
            @click="cerrarModal"
          >
            ×
          </button>

        </div>

        <div
          v-if="error"
          class="mensaje-error"
        >
          ⚠️ {{ error }}
        </div>

        <form @submit.prevent="guardarServicio">

          <div class="form-grid">

            <div class="campo campo-completo">
              <label>Nombre del cliente *</label>

              <input
                v-model="formulario.nombre"
                type="text"
                placeholder="Ej: Juan Pérez"
                :class="{ 'campo-error': campoInvalido('nombre') }"
              >
            </div>

            <div class="campo">
              <label>Tipo de servicio *</label>

              <select
                v-model="formulario.servicio"
                @change="actualizarPrecio"
                :class="{ 'campo-error': campoInvalido('servicio') }"
              >
                <option value="">Seleccione</option>

                <option value="Corte clásico">
                  Corte clásico
                </option>

                <option value="Corte moderno">
                  Corte moderno
                </option>

                <option value="Barba">
                  Barba
                </option>

                <option value="Corte + barba">
                  Corte + barba
                </option>

                <option value="Cejas">
                  Cejas
                </option>

                <option value="Tinte">
                  Tinte
                </option>
              </select>
            </div>

            <div class="campo">
              <label>Barbero *</label>

              <select v-model="formulario.barbero"
              :class="{ 'campo-error': campoInvalido('barbero') }"
              >
                <option value="">Seleccione</option>
                <option value="Don Ramiro">
                  Don Ramiro
                </option>
                <option value="Carlos">
                  Carlos
                </option>
                <option value="Miguel">
                  Miguel
                </option>
              </select>
            </div>

            <div class="campo">
              <label>Fecha *</label>

              <input
                v-model="formulario.fecha"
                type="date"
                :class="{ 'campo-error': campoInvalido('fecha') }"
              >
            </div>

            <div class="campo">
              <label>Hora *</label>

              <input
                v-model="formulario.hora"
                type="time"
                min="06:00"
                max="20:00"
                :class="{ 'campo-error': campoInvalido('hora') }"
              >
            </div>

            <div class="campo">
              <label>Precio cobrado *</label>

              <input
                v-model="formulario.precio"
                type="number"
                min="1"
                placeholder="Ej: 20000"
                :class="{ 'campo-error': campoInvalido('precio') }"
              >
            </div>

            <div class="campo">
              <label>Método de pago *</label>

              <select v-model="formulario.metodoPago"
              :class="{ 'campo-error': campoInvalido('metodoPago') }"
              >
                <option value="">Seleccione</option>
                <option value="Efectivo">
                  💵 Efectivo
                </option>
                <option value="Transferencia">
                  📱 Transferencia
                </option>
                <option value="Tarjeta">
                  💳 Tarjeta
                </option>
              </select>
            </div>

            <div class="campo">
              <label>Estado del pago *</label>

              <select v-model="formulario.estadoPago"
              :class="{ 'campo-error': campoInvalido('estadoPago') }">
                <option value="">Seleccione</option>
                <option value="Pagado">
                  Pagado
                </option>
                <option value="Pendiente">
                  Pendiente
                </option>
              </select>
            </div>

           

            <div class="campo campo-completo">
              <label>Observaciones</label>

              <textarea
                v-model="formulario.observaciones"
                rows="3"
                placeholder="Escriba alguna observación..."
              ></textarea>
            </div>

          </div>

          <div class="botones-formulario">

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

              <span v-if="editando">
                Guardar cambios
              </span>

              <span v-else>
                Registrar servicio
              </span>

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
  font-family: 'Work Sans', Arial, sans-serif;
  background: #EFE7D8;
  color: #050505;
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
  width: 100%;
  text-align: left;
}



.encabezado {
  background-image:
    linear-gradient(rgba(5, 5, 5, 0.26), rgba(5, 5, 5, 0.72)),
    url('./img/image.png');
  background-size: cover;
  background-position: center;
  height: 500px;
  color: #EFE7D8;
  padding: 28px 2%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
  border-bottom: 4px solid #9C6B2E;
}

.encabezado h1 {
  margin: 0;
  font-family: 'Oswald', sans-serif;
  margin-top: 410px;
  font-size: 35px;
  text-transform: uppercase;
}

.encabezado p {
  margin: 6px 0 0;
  color: #E2D6BE;
  font-size: 14px;
}

.boton-nuevo {
  border: 1px solid #fcfcfc;
  background: #9C6B2E;
  color: #EFE7D8;
  padding: 13px 20px;
  border-radius: 3px;
  font-size: 14px;
  margin-top: 380px;
 
 
 
}


.boton-nuevo-2 {
  border: 1px solid #fcfcfc;
  background: #9C6B2E;
  color: #EFE7D8;
  padding: 13px 20px;
  border-radius: 3px;
 
}

.boton-nuevo:hover {
  background: #ff5e01;
  color:rgb(252, 250, 250);
}


.informacion {
 max-width: 1200px;
 margin: 25px auto;
 padding: 0 20px;
 display: grid;
 grid-template-columns: repeat(3, 1fr);
 gap:15px;
}

.tarjeta-info {
  background: rgb(248, 247, 247);
  padding: 20px;
  border-radius: 12px;
  display: flex;
  gap:15px;
  align-items: center;
  box-shadow: 0 2px 8px #ddd;
}

.tarjeta-info:last-child {
  border-right: none;
}

.tarjeta-info > span {
  font-size: 26px;
}

.tarjeta-info strong {
  font-family: 'Oswald', sans-serif;
  font-size: 24px;
  font-weight: 600;
  color: #050505;
}

.tarjeta-info p {
  margin: 3px 0 0;
  color: #48544E;
  font-size: 13px;
}

/* ===== Contenido ===== */

.contenido {
  width: 95%;
  max-width: 1600PX;
  margin: auto;
  padding-bottom: 150px;
  
 
}

.sin-servicios {
  background: #E2D6BE;
  border: 2px dashed #C9BCA1;
  border-radius: 6px;
  text-align: center;
  height: 300px;
  

 
  
}

.icono-vacio {
  font-size: 46px;
  padding-top:20px ;
  
  
}

.sin-servicios h2 {
  
  font-size: 2rem;
 
}

.sin-servicios p {
  color: #48544E;
  margin-bottom: 25px;
}

.lista-servicios {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 20px;
}





.servicio-card {
  background: white;
  border-radius: 14px;
  padding: 22px;
  border-left: 5px solid #8e8e8e;
  box-shadow: 0 3px 12px rgba(0, 0, 0, 0.07);
}

.pago-pagado {
  border-left-color: #3b9c68;
}

.pago-pendiente {
  border-left-color: #d39a35;
  background: #fffdf7;
}

.card-cabecera {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 15px;
  border-bottom: 1px solid #eee;
  padding-bottom: 15px;
}

.card-cabecera h2 {
  color: #050505;
  margin: 0;
  font-size: 21px;
  text-transform: uppercase;
}

.tipo-servicio {
  margin: 5px 0 0;
  color: #a27442;
  font-weight: bold;
}

.alerta-pago,
.pago-ok {
  padding: 7px 10px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: bold;
  white-space: nowrap;
}

.alerta-pago {
  background: #ffe5b5;
  color: #8b5b12;
}

.pago-ok {
  background: #dff4e7;
  color: #287247;
}

.datos {
  padding: 18px 0;
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
}

.dato {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.dato span {
  color: #777;
  font-size: 13px;
}

.dato strong {
  font-size: 14px;
}

.calificacion {
  border-top: 1px solid #eee;
  padding: 15px 0;
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.calificacion strong {
  color: #d79a27;
  letter-spacing: 2px;
}

.calificacion-baja {
  background: #fff0f0;
  padding: 10px;
  border-radius: 8px;
}

.calificacion-media {
  background: #fff8e7;
  padding: 10px;
  border-radius: 8px;
}

.calificacion-alta {
  background: #eef9f2;
  padding: 10px;
  border-radius: 8px;
}

.texto-baja {
  color: #b63d3d;
  font-weight: bold;
}

.texto-media {
  color: #a27418;
}

.texto-alta {
  color: #358051;
}

.observaciones {
  background: #f7f4f0;
  padding: 12px;
  border-radius: 8px;
  margin-bottom: 15px;
  font-size: 14px;
}

.observaciones p {
  margin: 7px 0 0;
  color: #666;
}

.acciones {
  display: flex;
  gap: 10px;
  border-top: 1px solid #eee;
  padding-top: 15px;
}

.boton-editar,
.boton-eliminar {
  border: none;
  padding: 9px 14px;
  border-radius: 7px;
  font-weight: bold;
}

.boton-editar {
  background: #eee5da;
  color: #6e4b2d;
}

.boton-eliminar {
  background: #f8dddd;
  color: #a53c3c;
}

/* ===== Modal (estilo boleto) ===== */

.modal-fondo {
  position: fixed;
  inset: 0;
  background: rgba(20, 16, 12, 0.72);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  z-index: 10;
}

.modal {
  background: rgb(255, 255, 255);
  color: #050505;
  width: 100%;
  max-width: 700px;
  max-height: 90vh;
  overflow-y: auto;
  border-radius: 6px;
  padding: 0 0 25px;
  position: relative;
}



.modal-cabecera {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 20px 25px 18px;
  margin-bottom: 20px;
  background: #686868;
  color: #EFE7D8;
  border-bottom: 3px solid #071ac7a8;
}

.modal-cabecera h2 {
  margin: 0;
  font-family: 'Oswald', sans-serif;
  font-weight: 600;
}

.modal-cabecera p {
  color: #fffffe;
  margin: 5px 0 0;
  font-size: 14px;
}

.cerrar {
  border: 1px solid #E2D6BE;
  background: transparent;
  color: #EFE7D8;
  width: 32px;
  height: 32px;
  border-radius: 3px;
  font-size: 20px;
  line-height: 1;
}

.mensaje-error {
  background: #fbe9e7;
  color: #A23E32;
  border-left: 3px solid #A23E32;
  padding: 12px;
  margin: 0 25px 18px;
  font-size: 14px;
}

form {
  padding: 0 25px;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
}

.campo {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.campo-completo {
  grid-column: 1 / -1;
}

.campo label {
  font-weight: 600;
  font-size: 14px;
  color: #050505;
}

.campo input,
.campo select,
.campo textarea {
  width: 100%;
  border: none;
  border-bottom: 2px solid #3c5fd1;
  border-radius: 0;
  padding: 10px 4px;
  font-size: 14px;
  background: rgba(226, 225, 225, 0.726);
  color: #050505;
}

.campo input:focus,
.campo select:focus,
.campo textarea:focus {
  outline: none;
  border-bottom-color: #f80a0a;
}

.campo textarea {
  resize: vertical;
  border: 1px solid #3526c2;
  border-radius: 4px;
  padding: 10px;
}

.campo textarea:focus {
  border-color: #fa0505;
}

.botones-formulario {
  margin-top: 25px;
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.boton-cancelar,
.boton-guardar {
  border: none;
  padding: 12px 18px;
  border-radius: 3px;
  font-weight: 600;
}

.boton-cancelar {
  background: transparent;
  border: 1px solid #C9BCA1;
  color: #48544E;
}

.boton-guardar {
  background: #050505;
  color: #EFE7D8;
}

.boton-guardar:hover {
  background: #0f1613;
}

.campo-error {
  border-bottom-color: #e53935 !important;
  background: #fff0f0 !important;
}

.campo-error:focus {
  border-bottom-color: #e53935 !important;
}

@media (max-width: 800px) {

  .encabezado {
    flex-direction: column;
    align-items: stretch;
  }

  .encabezado h1 {
    font-size: 23px;
  }

  .boton-nuevo {
    width: 100%;
  }

  .informacion {
    flex-direction: column;
  }

  .tarjeta-info {
    border-right: none;
    border-bottom: 1px dashed #48544E;
  }

  .tarjeta-info:last-child {
    border-bottom: none;
  }

  .lista-servicios {
    grid-template-columns: 1fr;
  }

}

@media (max-width: 550px) {

  .encabezado,
  .contenido,
  .informacion {
    padding-left: 20px;
    padding-right: 20px;
  }

  .form-grid {
    grid-template-columns: 1fr;
  }

  .campo-completo {
    grid-column: auto;
  }

  form {
    padding: 0 18px;
  }

  .datos {
    grid-template-columns: 1fr;
  }

  .acciones {
    flex-direction: column;
  }

  .boton-editar,
  .boton-eliminar {
    width: 100%;
  }

}
</style>