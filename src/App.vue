<script setup>
import { ref, computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('servicios-barberia', [])

const mostrarModal = ref(false)
const editando = ref(false)
const idEditando = ref(null)
const error = ref([])
const campoConError = ref('')

const mostrarModalCalificar = ref(false)
const servicioCalificando = ref(null)
const calificacionSeleccionada = ref(0)

const mostrarModalEliminar = ref(false)
const idAEliminar = ref(null)

function fechaHoy() {
  const hoy = new Date()
  const anio = hoy.getFullYear()
  const mes = String(hoy.getMonth() + 1).padStart(2, '0')
  const dia = String(hoy.getDate()).padStart(2, '0')
  return `${anio}-${mes}-${dia}`
}

function horaActual() {
  const ahora = new Date()
  const horas = String(ahora.getHours()).padStart(2, '0')
  const minutos = String(ahora.getMinutes()).padStart(2, '0')
  return `${horas}:${minutos}`
}

function fechaHoraSeleccionadaEsPasada() {
  if (formulario.value.fecha === '' || formulario.value.hora === '') {
    return false
  }

  const seleccionada = new Date(`${formulario.value.fecha}T${formulario.value.hora}`)
  const ahora = new Date()

  return seleccionada <= ahora
}



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

const fechaMinima = fechaHoy()

const horaMinima = computed(() => {
  if (formulario.value.fecha === fechaHoy()) {
    return horaActual()
  }
  return '06:00'
})

function abrirModal() {
  editando.value = false
  idEditando.value = null
  error.value = ''
  campoConError.value = ''

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
  campoConError.value = ''
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
    campoConError.value = 'nombre'
    return false
  }

  if (/\d/.test(formulario.value.nombre)) {
    error.value = 'El nombre no puede contener números.'
    campoConError.value = 'nombre'
    return false
  }

  if (formulario.value.servicio === '') {
    error.value = 'Seleccione el tipo de servicio.'
    campoConError.value = 'servicio'
    return false
  }

  if (formulario.value.barbero === '') {
    error.value = 'Seleccione el barbero.'
    campoConError.value = 'barbero'
    return false
  }

  if (formulario.value.fecha === '') {
    error.value = 'Seleccione una fecha.'
    campoConError.value = 'fecha'
    return false
  }

  if (formulario.value.fecha < fechaHoy()) {
    error.value = 'La fecha no puede ser anterior a hoy.'
    campoConError.value = 'fecha'
    return false
  }

  if (formulario.value.hora === '') {
    error.value = 'Seleccione una hora.'
    campoConError.value = 'hora'
    return false
  }

  if (formulario.value.precio === '') {
    error.value = 'Ingrese el precio del servicio.'
    campoConError.value = 'precio'
    return false
  }

  if (Number(formulario.value.precio) <= 0) {
    error.value = 'El precio debe ser mayor que 0.'
    campoConError.value = 'precio'
    return false
  }

  if (formulario.value.metodoPago === '') {
    error.value = 'Seleccione el método de pago.'
    campoConError.value = 'metodoPago'
    return false
  }

  if (formulario.value.estadoPago === '') {
    error.value = 'Seleccione el estado del pago.'
    campoConError.value = 'estadoPago'
    return false
  }

  if (
    formulario.value.hora < '06:00' ||
    formulario.value.hora > '20:00'
  ) {
    error.value = 'La hora debe estar entre las 6:00 AM y las 8:00 PM.'
    campoConError.value = 'hora'
    return false
  }

  if (fechaHoraSeleccionadaEsPasada()) {
    error.value = 'La hora seleccionada ya pasó. Elija una hora posterior a la actual.'
    campoConError.value = 'hora'
    return false
  }

  campoConError.value = ''
  return true
}



function campoInvalido(campo) {
  return campo === campoConError.value
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
  campoConError.value = ''

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

function abrirModalEliminar(id) {
  idAEliminar.value = id
  mostrarModalEliminar.value = true
}

function confirmarEliminacion() {
  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].id === idAEliminar.value) {
      servicios.value.splice(i, 1)
      break
    }
  }

  cerrarModalEliminar()
}

function cerrarModalEliminar() {
  mostrarModalEliminar.value = false
  idAEliminar.value = null
}


function abrirModalCalificar(servicio) {
  servicioCalificando.value = servicio
  calificacionSeleccionada.value = 0
  mostrarModalCalificar.value = true
}

function seleccionarEstrella(numero) {
  calificacionSeleccionada.value = numero
}

function confirmarCalificacion() {
  if (calificacionSeleccionada.value === 0) {
    return
  }

  servicioCalificando.value.calificacion = calificacionSeleccionada.value
  cerrarModalCalificar()
}

function cerrarModalCalificar() {
  mostrarModalCalificar.value = false
  servicioCalificando.value = null
  calificacionSeleccionada.value = 0
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

function formatearHora(hora) {
  if (!hora) {
    return ''
  }

  const [horaStr, minutoStr] = hora.split(':')
  let horas = Number(horaStr)
  const minutos = minutoStr

  const sufijo = horas >= 12 ? 'PM' : 'AM'

  horas = horas % 12
  if (horas === 0) {
    horas = 12
  }

  return `${horas}:${minutos} ${sufijo}`
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
        <p>Registro de servicios  </p>
        <h1> ✂️ Barbería Don Ramiro</h1>
        
        
      </div>
       

      <div class="boton-con-texto">

        <button
          class="boton-nuevo"
          @click="abrirModal"
        >
          + Nuevo servicio
        </button>

        <p>agenda tu servicio para poder 
          tener el gusto de atenderte</p>

      </div>
      
      
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
              <strong>{{ formatearHora(servicio.hora) }}</strong>
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
                @click="abrirModalCalificar(servicio)"
              >
                ⭐ Calificar
              </button>

              <button
                class="boton-eliminar"
                @click="abrirModalEliminar(servicio.id)"
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
              💈✂️ Nuevo servicio
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
                placeholder="Ej: Ronaldo Rincon"
                :class="{ 'campo-error': campoInvalido('nombre') }"
                @input="formulario.nombre = formulario.nombre.replace(/[0-9]/g, '')"
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
                :min="fechaMinima"
                :class="{ 'campo-error': campoInvalido('fecha') }"
              >
            </div>

            <div class="campo">
              <label>Hora *</label>

              <input
                v-model="formulario.hora"
                type="time"
                :min="horaMinima"
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
               
                readonly
                
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

    <!-- Modal de calificación -->
    <div
      v-show="mostrarModalCalificar"
      class="modal-fondo"
    >
      <div class="modal modal-pequeno">

        <div class="modal-cabecera">
          <div>
            <h2>⭐ Calificar servicio</h2>
            <p v-if="servicioCalificando">
              {{ servicioCalificando.nombre }} - {{ servicioCalificando.servicio }}
            </p>
          </div>

          <button
            class="cerrar"
            @click="cerrarModalCalificar"
          >
            ×
          </button>
        </div>

        <div class="estrellas-seleccion">
          <button
            v-for="n in 5"
            :key="n"
            type="button"
            class="boton-estrella"
            @click="seleccionarEstrella(n)"
          >
            <span v-if="n <= calificacionSeleccionada">★</span>
            <span v-else>☆</span>
          </button>
        </div>

        <div class="botones-formulario">
          <button
            type="button"
            class="boton-cancelar"
            @click="cerrarModalCalificar"
          >
            Cancelar
          </button>

          <button
            type="button"
            class="boton-guardar"
            @click="confirmarCalificacion"
          >
            Confirmar
          </button>
        </div>

      </div>
    </div>

    <!-- Modal de eliminación -->
    <div
      v-show="mostrarModalEliminar"
      class="modal-fondo"
    >
      <div class="modal modal-pequeno">

        <div class="modal-cabecera">
          <div>
            <h2>🗑️ Eliminar servicio</h2>
            <p>Esta acción no se puede deshacer</p>
          </div>

          <button
            class="cerrar"
            @click="cerrarModalEliminar"
          >
            ×
          </button>
        </div>

        <div class="mensaje-confirmacion">
          ¿Está seguro de que desea eliminar este servicio?
        </div>

        <div class="botones-formulario">
          <button
            type="button"
            class="boton-cancelar"
            @click="cerrarModalEliminar"
          >
            Cancelar
          </button>

          <button
            type="button"
            class="boton-eliminar-confirmar"
            @click="confirmarEliminacion"
          >
            Sí, eliminar
          </button>
        </div>

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

background-image: url('./img/image.png');
background-size: 50% 120%;
background-position: center;
background-repeat: no-repeat;
width: 100%;
height: 160px;
background-color: #0f0a06ee;
border-bottom: 5px solid rgb(228, 157, 5);
display: flex;
justify-content: space-between;
align-items: center;
}

.encabezado h1 {
  margin: 0;
  font-family: 'Oswald', sans-serif;
  color: white;
  width: 400px;
  font-size: 30px;
  text-transform: uppercase;
  margin-left: 20px;
  width: 500px;
 
}

.encabezado p {
  margin: 6px 0 0;
  color: #E2D6BE;
  font-size: 14px;
  margin-left: 18px;
  text-transform: uppercase;
 
}

.boton-con-texto {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-right: 60px;
  
}

.boton-con-texto p {
 
  color: #E2D6BE;
  font-size: 15px;
  margin-left: 1030px;
  width: 300px;
  padding: 10px;
}

.boton-nuevo {
  border: 1px solid #fcfcfc;
  background: #9C6B2E;
  color: #EFE7D8;
  padding: 13px 20px;
  border-radius: 3px;
  font-size: 14px;
  margin-left: 1000px;

 
 
 
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






.modal-fondo {
  position: fixed;
  inset: 0;
  background: rgba(75, 74, 73, 0.72);
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
  background: #0f0f0f;
  color: #df7e10;
  border-bottom: 5px solid #fa9b0c;
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
  background: #fbe9e7d0;
  color: #A23E32;
  border-left: 3px solid #fc0202;
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
  border-bottom: 2px solid hsl(209, 100%, 56%);
  border-radius: 0;
  padding: 10px 4px;
  font-size: 14px;
  background: rgba(228, 227, 227, 0.61);
  color: #0a0a0a;
}

.campo input:focus,
.campo select:focus,
.campo textarea:focus {
  outline: none;
  border-bottom-color: #0a0a0a;
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

.modal-pequeno {
  max-width: 400px;
}

.estrellas-seleccion {
  display: flex;
  justify-content: center;
  gap: 10px;
  padding: 30px 25px;
  font-size: 40px;
}

.boton-estrella {
  border: none;
  background: transparent;
  color: #d79a27;
  padding: 0;
  line-height: 1;
}

.mensaje-confirmacion {
  padding: 20px 25px 0;
  color: #48544E;
  font-size: 15px;
}

.boton-eliminar-confirmar {
  border: none;
  padding: 12px 18px;
  border-radius: 3px;
  font-weight: 600;
  background: #A23E32;
  color: white;
}

.boton-eliminar-confirmar:hover {
  background: #822f26;
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