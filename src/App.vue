<script setup>
import { ref, computed, watch } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('servicios-barberia', [])
const serviciosArchivados = useLocalStorage('servicios-barberia-archivados', [])

const serviciosDisponibles = useLocalStorage('catalogo-barberia', [
  { nombre: 'Corte clásico', precio: 15000, icono: '✂️' },
  { nombre: 'Corte con diseños', precio: 18000, icono: '💇' },
  { nombre: 'Barba', precio: 10000, icono: '🧔' },
  { nombre: 'Corte + barba', precio: 25000, icono: '💈' },
  { nombre: 'Cejas', precio: 5000, icono: '👁️' },
  { nombre: 'Tinte', precio: 30000, icono: '🎨' }
])

const conflictos = {
  'Corte + barba': ['Corte clásico', 'Corte con diseños', 'Barba'],
  'Corte clásico': ['Corte + barba', 'Corte con diseños'],
  'Corte con diseños': ['Corte + barba', 'Corte clásico'],
  'Barba': ['Corte + barba']
}

const barberos = ['Don Ramiro', 'Carlos', 'Miguel']
const porcentajesComision = {
  'Don Ramiro': 40,
  'Carlos': 40,
  'Miguel': 40
}

const mostrarModal = ref(false)
const editando = ref(false)
const idEditando = ref(null)
const error = ref('')
const campoConError = ref('')

const mostrarModalCalificar = ref(false)
const servicioCalificando = ref(null)
const calificacionSeleccionada = ref(0)

const mostrarModalEliminar = ref(false)
const idAEliminar = ref(null)

const mostrarModalCaja = ref(false)
const mostrarModalCatalogo = ref(false)
const mostrarModalHistorial = ref(false)

const ordenActual = ref('fecha')
const ordenAscendente = ref(false)
const clienteBuscado = ref('')

const nuevoServicioCatalogo = ref({
  nombre: '',
  precio: '',
  icono: '✂️'
})
const editandoCatalogo = ref(false)
const nombreCatalogoEditando = ref(null)

const formulario = ref({
  nombre: '',
  servicios: [],
  barbero: '',
  fecha: '',
  hora: '',
  precio: '',
  propina: '',
  metodoPago: '',
  estadoPago: '',
  calificacion: '',
  observaciones: '',
  fotoAntes: '',
  fotoDespues: ''
})

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

const fechaMinima = fechaHoy()

const horaMinima = computed(() => {
  if (formulario.value.fecha === fechaHoy()) {
    return horaActual()
  }
  return '06:00'
})

function fechaHoraSeleccionadaEsPasada() {
  if (formulario.value.fecha === '' || formulario.value.hora === '') {
    return false
  }

  const seleccionada = new Date(`${formulario.value.fecha}T${formulario.value.hora}`)
  const ahora = new Date()

  return seleccionada <= ahora
}

function servicioYaTermino(servicio) {
  if (!servicio.fecha || !servicio.hora) {
    return false

  
  }

  const fechaServicio = new Date(`${servicio.fecha}T${servicio.hora}`)
  const ahora = new Date()

  return fechaServicio <= ahora
}


function estaDeshabilitado(nombreServicio) {
  for (let i = 0; i < formulario.value.servicios.length; i++) {
    const seleccionado = formulario.value.servicios[i]

    if (seleccionado === nombreServicio) {
      continue
    }

    const listaConflictos = conflictos[seleccionado] || []

    if (listaConflictos.includes(nombreServicio)) {
      return true
    }
  }

  return false
}

function abrirModal() {
  editando.value = false
  idEditando.value = null
  error.value = ''
  campoConError.value = ''

  formulario.value = {
    nombre: '',
    servicios: [],
    barbero: '',
    fecha: fechaHoy(),
    hora: '',
    precio: '',
    propina: '',
    metodoPago: '',
    estadoPago: '',
    calificacion: '',
    observaciones: '',
    fotoAntes: '',
    fotoDespues: ''
  }

  mostrarModal.value = true
}

function cerrarModal() {
  mostrarModal.value = false
  error.value = ''
  campoConError.value = ''
}

function actualizarPrecioTotal() {
  let total = 0

  for (let i = 0; i < formulario.value.servicios.length; i++) {
    const nombreServicio = formulario.value.servicios[i]
    const encontrado = serviciosDisponibles.value.find(s => s.nombre === nombreServicio)

    if (encontrado) {
      total = total + Number(encontrado.precio)
    }
  }

  if (clienteFrecuente.value && total > 0) {
    total = total * 0.90
  }

  formulario.value.precio = total > 0 ? total : ''
}

watch(
  () => formulario.value.servicios,
  () => {
    actualizarPrecioTotal()
  },
  { deep: true }
)

const totalFormulario = computed(() => {
  const precio = Number(formulario.value.precio) || 0
  const propina = Number(formulario.value.propina) || 0
  return precio + propina
})

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

  if (formulario.value.servicios.length === 0) {
    error.value = 'Seleccione al menos un tipo de servicio.'
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

  if (formulario.value.precio === '' || Number(formulario.value.precio) <= 0) {
    error.value = 'El precio debe ser mayor que 0.'
    campoConError.value = 'precio'
    return false
  }

  if (Number(formulario.value.propina) < 0) {
    error.value = 'La propina no puede ser negativa.'
    campoConError.value = 'propina'
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

  if (formulario.value.hora < '06:00' || formulario.value.hora > '20:00') {
    error.value = 'La hora debe estar entre las 6:00 AM y las 8:00 PM.'
    campoConError.value = 'hora'
    return false
  }

  if (fechaHoraSeleccionadaEsPasada() && !editando.value) {
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

function crearObjetoServicio(id) {
  return {
    id,
    nombre: formulario.value.nombre.trim(),
    servicios: [...formulario.value.servicios],
    barbero: formulario.value.barbero,
    fecha: formulario.value.fecha,
    hora: formulario.value.hora,
    precio: Number(formulario.value.precio),
    propina: Number(formulario.value.propina) || 0,
    total: totalFormulario.value,
    metodoPago: formulario.value.metodoPago,
    estadoPago: formulario.value.estadoPago,
    calificacion: formulario.value.calificacion,
    observaciones: formulario.value.observaciones,
    fotoAntes: formulario.value.fotoAntes || '',
    fotoDespues: formulario.value.fotoDespues || '',
    archivado: false
  }
}

function guardarServicio() {
  error.value = ''

  if (!validarFormulario()) {
    return
  }

  if (editando.value === true) {
    for (let i = 0; i < servicios.value.length; i++) {
      if (servicios.value[i].id === idEditando.value) {
        servicios.value[i] = crearObjetoServicio(idEditando.value)
        break
      }
    }
  } else {
    servicios.value.push(crearObjetoServicio(Date.now()))
  }

  cerrarModal()
}

function editarServicio(servicio) {
  const estadoPago = String(servicio.estadoPago || '').trim()

  if (servicio.calificacion) {
    return
  }

  if (
    servicioYaTermino(servicio) &&
    estadoPago !== 'Pendiente' &&
    estadoPago !== 'Fiado'
  ) {
    return
  }

  
  editando.value = true
  idEditando.value = servicio.id
  error.value = ''
  campoConError.value = ''

  formulario.value = {
    nombre: servicio.nombre,
    servicios: Array.isArray(servicio.servicios) ? [...servicio.servicios] : [],
    barbero: servicio.barbero,
    fecha: servicio.fecha,
    hora: servicio.hora,
    precio: servicio.precio,
    propina: servicio.propina || 0,
    metodoPago: servicio.metodoPago,
    estadoPago: servicio.estadoPago,
    calificacion: servicio.calificacion,
    observaciones: servicio.observaciones || '',
    fotoAntes: servicio.fotoAntes || '',
    fotoDespues: servicio.fotoDespues || ''
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
  calificacionSeleccionada.value = Number(servicio.calificacion) || 0
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
    estrellas += i <= Number(calificacion) ? '★' : '☆'
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
  if (metodo === 'Efectivo') return '💵'
  if (metodo === 'Transferencia') return '📱'
  if (metodo === 'Tarjeta') return '💳'
  return '💰'
}

function clasePago(estado) {
   if (estado === 'Pagado') {
    return 'pago-pagado'
  }

  if (estado === 'Pendiente') {
    return 'pago-pendiente'
  }

  if (estado === 'Fiado') {
    return 'pago-fiado'
  }
}

function claseCalificacion(calificacion) {
  if (!calificacion) return 'calificacion-media'
  if (Number(calificacion) <= 2) return 'calificacion-baja'
  if (Number(calificacion) === 3) return 'calificacion-media'
  return 'calificacion-alta'
}

function textoServicios(listaServicios) {
  if (!Array.isArray(listaServicios) || listaServicios.length === 0) {
    return ''
  }

  return listaServicios.join(', ')
}

function dinero(valor) {
  return Number(valor || 0).toLocaleString('es-CO')
}

function totalCobrado(servicio) {
  return Number(servicio.total || (Number(servicio.precio) + Number(servicio.propina || 0)))
}

function ordenarPor(campo) {
  if (ordenActual.value === campo) {
    ordenAscendente.value = !ordenAscendente.value
  } else {
    ordenActual.value = campo
    ordenAscendente.value = false
  }
}

const serviciosOrdenados = computed(() => {
  const copia = [...servicios.value]

  copia.sort((a, b) => {
    let valorA = a[ordenActual.value]
    let valorB = b[ordenActual.value]

    if (ordenActual.value === 'fecha') {
      valorA = `${a.fecha} ${a.hora}`
      valorB = `${b.fecha} ${b.hora}`
    }

    if (ordenActual.value === 'precio') {
      valorA = totalCobrado(a)
      valorB = totalCobrado(b)
    }

    if (ordenActual.value === 'calificacion') {
      valorA = Number(a.calificacion) || 0
      valorB = Number(b.calificacion) || 0
    }

    if (valorA < valorB) return ordenAscendente.value ? -1 : 1
    if (valorA > valorB) return ordenAscendente.value ? 1 : -1
    return 0
  })

  return copia
})

function grupoHorario(hora) {
  if (!hora) return 'Noche'

  const horas = Number(hora.split(':')[0])

  if (horas < 12) return 'Mañana'
  if (horas < 18) return 'Tarde'
  return 'Noche'
}

function serviciosDelGrupo(grupo) {
  return serviciosOrdenados.value.filter(servicio => grupoHorario(servicio.hora) === grupo)
}

const todosLosServicios = computed(() => [
  ...servicios.value,
  ...serviciosArchivados.value
])

const serviciosDeHoy = computed(() =>
  servicios.value.filter(servicio => servicio.fecha === fechaHoy())
)

const totalVendido = computed(() =>
  servicios.value
    .filter(servicio => servicio.estadoPago === 'Pagado')
    .reduce((total, servicio) => total + totalCobrado(servicio), 0)
)

const promedioCalificacion = computed(() => {
  const calificados = servicios.value.filter(s => Number(s.calificacion) > 0)

  if (calificados.length === 0) return 0

  const suma = calificados.reduce((total, servicio) => total + Number(servicio.calificacion), 0)
  return (suma / calificados.length).toFixed(1)
})

function cantidadCortesBarbero(barbero) {
  return serviciosDeHoy.value.filter(servicio =>
    servicio.barbero === barbero &&
    Array.isArray(servicio.servicios) &&
    servicio.servicios.some(nombre => nombre.toLowerCase().includes('corte'))
  ).length
}

const barberoConMasCortes = computed(() => {
  let ganador = 'Sin cortes'

  let mayor = 0

  for (const barbero of barberos) {
    const cantidad = cantidadCortesBarbero(barbero)

    if (cantidad > mayor) {
      mayor = cantidad
      ganador = barbero
    }
  }

  return mayor > 0 ? `${ganador} (${mayor})` : 'Sin cortes'
})

const historialCliente = computed(() => {
  const nombre = clienteBuscado.value.trim().toLowerCase()

  if (nombre === '') {
    return {
      veces: 0,
      gastado: 0
    }
  }

  const coincidencias = todosLosServicios.value.filter(servicio =>
    servicio.nombre.toLowerCase() === nombre
  )

  return {
    veces: coincidencias.length,
    gastado: coincidencias.reduce((total, servicio) => total + totalCobrado(servicio), 0)
  }
})

const clienteFrecuente = computed(() => {
  const nombre = formulario.value.nombre.trim().toLowerCase()

  if (nombre === '') return false

  const visitas = todosLosServicios.value.filter(servicio =>
    servicio.nombre.toLowerCase() === nombre
  ).length

  return visitas >= 5
})

function totalMetodoPago(metodo) {
  return serviciosDeHoy.value
    .filter(servicio => servicio.metodoPago === metodo && servicio.estadoPago === 'Pagado')
    .reduce((total, servicio) => total + totalCobrado(servicio), 0)
}

const totalPendiente = computed(() =>
  serviciosDeHoy.value
    .filter(servicio => servicio.estadoPago !== 'Pagado')
    .reduce((total, servicio) => total + totalCobrado(servicio), 0)
)

const deudas = computed(() => {
  const resultado = {}

  todosLosServicios.value
    .filter(servicio => servicio.estadoPago === 'Fiado')
    .forEach(servicio => {
      if (!resultado[servicio.nombre]) {
        resultado[servicio.nombre] = 0
      }

      resultado[servicio.nombre] += totalCobrado(servicio)
    })

  return Object.entries(resultado).map(([nombre, total]) => ({
    nombre,
    total
  }))
})

const comisiones = computed(() =>
  barberos.map(barbero => {
    const serviciosBarbero = serviciosDeHoy.value.filter(servicio => servicio.barbero === barbero)
    const base = serviciosBarbero.reduce((total, servicio) => total + Number(servicio.precio || 0), 0)
    const porcentaje = porcentajesComision[barbero] || 0

    return {
      barbero,
      porcentaje,
      base,
      comision: base * porcentaje / 100
    }
  })
)

function cerrarCaja() {
  mostrarModalCaja.value = true
}

function archivarServiciosDelDia() {
  const delDia = servicios.value.filter(servicio => servicio.fecha === fechaHoy())

  delDia.forEach(servicio => {
    serviciosArchivados.value.push({
      ...servicio,
      archivado: true
    })
  })

  servicios.value = servicios.value.filter(servicio => servicio.fecha !== fechaHoy())
  mostrarModalCaja.value = false
}

function editarFoto(evento, campo) {
  const archivo = evento.target.files[0]

  if (!archivo) return

  if (!archivo.type.startsWith('image/')) {
    error.value = 'Seleccione un archivo de imagen.'
    return
  }

  if (archivo.size > 1.5 * 1024 * 1024) {
    error.value = 'La imagen es muy pesada. Use una imagen menor a 1.5 MB.'
    return
  }

  const lector = new FileReader()

  lector.onload = () => {
    formulario.value[campo] = lector.result
  }

  lector.readAsDataURL(archivo)
}

function eliminarFoto(campo) {
  formulario.value[campo] = ''
}

function abrirCatalogo() {
  editandoCatalogo.value = false
  nombreCatalogoEditando.value = null
  nuevoServicioCatalogo.value = {
    nombre: '',
    precio: '',
    icono: '✂️'
  }
  mostrarModalCatalogo.value = true
}

function editarServicioCatalogo(servicio) {
  editandoCatalogo.value = true
  nombreCatalogoEditando.value = servicio.nombre
  nuevoServicioCatalogo.value = {
    nombre: servicio.nombre,
    precio: servicio.precio,
    icono: servicio.icono
  }
}

function guardarServicioCatalogo() {
  const nombre = nuevoServicioCatalogo.value.nombre.trim()
  const precio = Number(nuevoServicioCatalogo.value.precio)

  if (nombre === '' || precio <= 0) {
    return
  }

  if (editandoCatalogo.value) {
    const encontrado = serviciosDisponibles.value.find(
      servicio => servicio.nombre === nombreCatalogoEditando.value
    )

    if (encontrado) {
      encontrado.nombre = nombre
      encontrado.precio = precio
      encontrado.icono = nuevoServicioCatalogo.value.icono
    }

    for (const servicio of servicios.value) {
      if (Array.isArray(servicio.servicios)) {
        servicio.servicios = servicio.servicios.map(nombreServicio =>
          nombreServicio === nombreCatalogoEditando.value ? nombre : nombreServicio
        )
      }
    }
  } else {
    const existe = serviciosDisponibles.value.some(
      servicio => servicio.nombre.toLowerCase() === nombre.toLowerCase()
    )

    if (existe) return

    serviciosDisponibles.value.push({
      nombre,
      precio,
      icono: nuevoServicioCatalogo.value.icono
    })
  }

  nuevoServicioCatalogo.value = {
    nombre: '',
    precio: '',
    icono: '✂️'
  }
  editandoCatalogo.value = false
  nombreCatalogoEditando.value = null
}

function eliminarServicioCatalogo(nombre) {
  const estaUsado = todosLosServicios.value.some(servicio =>
    Array.isArray(servicio.servicios) && servicio.servicios.includes(nombre)
  )

  if (estaUsado) {
    return
  }

  serviciosDisponibles.value = serviciosDisponibles.value.filter(
    servicio => servicio.nombre !== nombre
  )
}
</script>

<template>
  <div class="app">

    <header class="encabezado">
      <div>
        <p>Registro de servicios</p>
        <h1>✂️ Barbería Don Ramiro</h1>
      </div>

      <div class="boton-con-texto">
        <button class="boton-nuevo" @click="abrirModal">
          + Nuevo servicio
        </button>
        <p>agenda tu servicio para poder tener el gusto de atenderte</p>
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
        <span>💰</span>
        <div>
          <strong>${{ dinero(totalVendido) }}</strong>
          <p>Total vendido</p>
        </div>
      </div>

      <div class="tarjeta-info">
        <span>⭐</span>
        <div>
          <strong>{{ promedioCalificacion }}/5</strong>
          <p>Promedio de calificación</p>
        </div>
      </div>

      <div class="tarjeta-info">
        <span>✂️</span>
        <div>
          <strong>{{ barberoConMasCortes }}</strong>
          <p>Más cortes de hoy</p>
        </div>
      </div>
    </section>

    <section class="herramientas">
      <div class="ordenamiento">
        <strong>Ordenar:</strong>
        <button @click="ordenarPor('fecha')">
          📅 Fecha {{ ordenActual === 'fecha' ? (ordenAscendente ? '↑' : '↓') : '' }}
        </button>
        <button @click="ordenarPor('precio')">
          💰 Precio {{ ordenActual === 'precio' ? (ordenAscendente ? '↑' : '↓') : '' }}
        </button>
        <button @click="ordenarPor('calificacion')">
          ⭐ Calificación {{ ordenActual === 'calificacion' ? (ordenAscendente ? '↑' : '↓') : '' }}
        </button>
      </div>

      <div class="botones-extra">
        <button @click="mostrarModalHistorial = true">👤 Historial cliente</button>
        <button @click="abrirCatalogo">✂️ Catálogo</button>
        <button @click="cerrarCaja">💵 Cerrar caja</button>
      </div>
    </section>

    <section class="panel-alerta" v-if="deudas.length > 0">
      <h3>⚠️ Recordatorios de deudas</h3>
      <div class="lista-deudas">
        <div v-for="deuda in deudas" :key="deuda.nombre">
          <strong>{{ deuda.nombre }}</strong>
          <span>Debe ${{ dinero(deuda.total) }}</span>
        </div>
      </div>
    </section>

    <section class="panel-comisiones">
      <h3>💈 Comisiones del día</h3>
      <div class="comisiones-grid">
        <div v-for="comision in comisiones" :key="comision.barbero" class="comision-card">
          <strong>{{ comision.barbero }}</strong>
          <span>{{ comision.porcentaje }}% de comisión</span>
          <small>Base: ${{ dinero(comision.base) }}</small>
          <b>${{ dinero(comision.comision) }}</b>
        </div>
      </div>
    </section>

    <div class="contenido">

      <div v-if="servicios.length === 0" class="sin-servicios">
        <div class="icono-vacio">💈</div>
        <h2>No hay servicios registrados</h2>
        <p>Agregue el primer servicio de la barbería.</p>
        <button class="boton-nuevo-2" @click="abrirModal">
          + Registrar servicio
        </button>
      </div>

      <div v-else class="lista-turnos">

        <template v-for="grupo in ['Mañana', 'Tarde', 'Noche']" :key="grupo">
          <div v-if="serviciosDelGrupo(grupo).length > 0" class="separador-turno">
            <h2>{{ grupo }}</h2>
            <span>{{ serviciosDelGrupo(grupo).length }} servicio(s)</span>
          </div>

          <div v-if="serviciosDelGrupo(grupo).length > 0" class="lista-servicios">
            <article
              v-for="servicio in serviciosDelGrupo(grupo)"
              :key="servicio.id"
              class="servicio-card"
              :class="clasePago(servicio.estadoPago)"
            >
              <div class="card-cabecera">
                <div>
                  <h2>{{ servicio.nombre }}</h2>
                  <p class="tipo-servicio">{{ textoServicios(servicio.servicios) }}</p>
                </div>

                <div v-if="servicio.estadoPago !== 'Pagado'" class="alerta-pago">
                  ⚠️ {{ servicio.estadoPago }}
                </div>

                <div v-else class="pago-ok">✓ Pagado</div>
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
                  <strong>${{ dinero(servicio.precio) }}</strong>
                </div>

                <div class="dato">
                  <span>💳 Pago</span>
                  <strong>{{ iconoPago(servicio.metodoPago) }} {{ servicio.metodoPago }}</strong>
                </div>

                <div class="dato">
                  <span>💰 Total</span>
                  <strong>
                    ${{ dinero(servicio.precio) }}
                    <span v-if="Number(servicio.propina) > 0">
                      + ${{ dinero(servicio.propina) }} propina
                    </span>
                  </strong>
                </div>
              </div>

              <div class="fotos-card" v-if="servicio.fotoAntes || servicio.fotoDespues">
                <div v-if="servicio.fotoAntes">
                  <small>Antes</small>
                  <img :src="servicio.fotoAntes" alt="Foto antes">
                </div>
                <div v-if="servicio.fotoDespues">
                  <small>Después</small>
                  <img :src="servicio.fotoDespues" alt="Foto después">
                </div>
              </div>

              <div class="calificacion" :class="claseCalificacion(servicio.calificacion)">
                <span>Calificación:</span>
                <strong>{{ mostrarEstrellas(servicio.calificacion) }}</strong>
                <span>{{ servicio.calificacion || 0 }}/5</span>

                <span v-if="Number(servicio.calificacion) <= 2 && servicio.calificacion" class="texto-baja">
                  ⚠️ Calificación baja
                </span>

                <span v-else-if="Number(servicio.calificacion) === 3" class="texto-media">
                  Regular
                </span>

                <span v-else-if="Number(servicio.calificacion) > 3" class="texto-alta">
                  Excelente
                </span>
              </div>

              <div v-if="servicio.observaciones !== ''" class="observaciones">
                <strong>📝 Observaciones:</strong>
                <p>{{ servicio.observaciones }}</p>
              </div>

              <div class="acciones">

                <template v-if="!servicio.calificacion">

                  <button
                    v-if="
                      !servicioYaTermino(servicio) ||
                      String(servicio.estadoPago).trim() === 'Pendiente' ||
                      String(servicio.estadoPago).trim() === 'Fiado'
                    "
                    class="boton-editar"
                    @click="editarServicio(servicio)"
                  >
                    ✏️ Editar
                  </button>

                  <button
                    v-if="!servicioYaTermino(servicio)"
                    class="boton-eliminar"
                    @click="abrirModalEliminar(servicio.id)"
                  >
                    🗑️ Eliminar
                  </button>

                  <button
                    v-if="servicioYaTermino(servicio) && !servicio.calificacion"
                    class="boton-calificar"
                    @click="abrirModalCalificar(servicio)"
                  >
                    ⭐ Calificar
                  </button>

                </template>

              </div>

            </article>
          </div>
        </template>

      </div>
    </div>

    <!-- Modal principal -->
    <div v-show="mostrarModal" class="modal-fondo">
      <div class="modal">

        <div class="modal-cabecera">
          <div>
            <h2 v-if="editando">✏️ Editar servicio</h2>
            <h2 v-else>💈✂️ Nuevo servicio</h2>
            <p>Complete la información del servicio</p>
          </div>

          <button class="cerrar" @click="cerrarModal">×</button>
        </div>

        <div v-if="error" class="mensaje-error">
          ⚠️ {{ error }}
        </div>

        <div v-if="clienteFrecuente" class="alerta-fidelidad">
          🎉 ¡Cliente frecuente, aplica 10% de descuento!
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

            <div class="campo campo-completo">
              <label>Tipo(s) de servicio *</label>

              <div class="grupo-tarjetas" :class="{ 'campo-error': campoInvalido('servicio') }">
                <label
                  v-for="s in serviciosDisponibles"
                  :key="s.nombre"
                  class="tarjeta-servicio"
                  :class="{
                    'tarjeta-seleccionada': formulario.servicios.includes(s.nombre),
                    'tarjeta-deshabilitada': estaDeshabilitado(s.nombre)
                  }"
                >
                  <input
                    type="checkbox"
                    :value="s.nombre"
                    v-model="formulario.servicios"
                    :disabled="estaDeshabilitado(s.nombre)"
                    hidden
                  >

                  <span class="check-servicio">✓</span>
                  <span class="icono-servicio">{{ s.icono }}</span>
                  <span class="nombre-servicio">{{ s.nombre }}</span>
                  <span class="precio-servicio">${{ dinero(s.precio) }}</span>

                  <span v-if="estaDeshabilitado(s.nombre)" class="nota-conflicto">
                    ya incluido
                  </span>
                </label>
              </div>
            </div>

            <div class="campo">
              <label>Barbero *</label>
              <select v-model="formulario.barbero" :class="{ 'campo-error': campoInvalido('barbero') }">
                <option value="">Seleccione</option>
                <option v-for="barbero in barberos" :key="barbero" :value="barbero">
                  {{ barbero }}
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
              <label>Precio base *</label>
              <input v-model="formulario.precio" type="number" min="1" readonly>
            </div>

            <div class="campo">
              <label>Propina (opcional)</label>
              <input
                v-model="formulario.propina"
                type="number"
                min="0"
                placeholder="0"
                :class="{ 'campo-error': campoInvalido('propina') }"
              >
            </div>

            <div class="campo">
              <label>Total a cobrar</label>
              <input :value="totalFormulario" type="number" readonly>
            </div>

            <div class="campo">
              <label>Método de pago *</label>
              <select v-model="formulario.metodoPago" :class="{ 'campo-error': campoInvalido('metodoPago') }">
                <option value="">Seleccione</option>
                <option value="Efectivo">💵 Efectivo</option>
                <option value="Transferencia">📱 Transferencia</option>
                <option value="Tarjeta">💳 Tarjeta</option>
              </select>
            </div>

            <div class="campo">
              <label>Estado del pago *</label>
              <select v-model="formulario.estadoPago" :class="{ 'campo-error': campoInvalido('estadoPago') }">
                <option value="">Seleccione</option>
                <option value="Pagado">Pagado</option>
                <option value="Pendiente">Pendiente</option>
                <option value="Fiado">Fiado</option>
              </select>
            </div>

            <div class="campo campo-completo">
              <label>Foto antes (opcional)</label>
              <input type="file" accept="image/*" @change="editarFoto($event, 'fotoAntes')">

              <div v-if="formulario.fotoAntes" class="vista-foto">
                <img :src="formulario.fotoAntes" alt="Antes">
                <button type="button" @click="eliminarFoto('fotoAntes')">Quitar</button>
              </div>
            </div>

            <div class="campo campo-completo">
              <label>Foto después (opcional)</label>
              <input type="file" accept="image/*" @change="editarFoto($event, 'fotoDespues')">

              <div v-if="formulario.fotoDespues" class="vista-foto">
                <img :src="formulario.fotoDespues" alt="Después">
                <button type="button" @click="eliminarFoto('fotoDespues')">Quitar</button>
              </div>
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
            <button type="button" class="boton-cancelar" @click="cerrarModal">
              Cancelar
            </button>

            <button type="submit" class="boton-guardar">
              <span v-if="editando">Guardar cambios</span>
              <span v-else>Registrar servicio</span>
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Historial por cliente -->
    <div v-show="mostrarModalHistorial" class="modal-fondo">
      <div class="modal modal-pequeno">
        <div class="modal-cabecera">
          <div>
            <h2>👤 Historial por cliente</h2>
            <p>Consulte las visitas y el dinero gastado</p>
          </div>
          <button class="cerrar" @click="mostrarModalHistorial = false">×</button>
        </div>

        <div class="contenido-modal-simple">
          <label>Nombre del cliente</label>
          <input v-model="clienteBuscado" type="text" placeholder="Ej: Ronaldo Rincon">

          <div class="resultado-historial" v-if="clienteBuscado.trim()">
            <strong>{{ historialCliente.veces }}</strong>
            <span>veces ha venido</span>

            <strong>${{ dinero(historialCliente.gastado) }}</strong>
            <span>gastado en total</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Catálogo editable -->
    <div v-show="mostrarModalCatalogo" class="modal-fondo">
      <div class="modal">
        <div class="modal-cabecera">
          <div>
            <h2>✂️ Catálogo de servicios</h2>
            <p>Crear y editar servicios con precio base sugerido</p>
          </div>
          <button class="cerrar" @click="mostrarModalCatalogo = false">×</button>
        </div>

        <div class="contenido-modal-simple">
          <div class="form-grid">
            <div class="campo">
              <label>Nombre</label>
              <input v-model="nuevoServicioCatalogo.nombre" type="text" placeholder="Ej: Lavado">
            </div>

            <div class="campo">
              <label>Precio base sugerido</label>
              <input v-model="nuevoServicioCatalogo.precio" type="number" min="1">
            </div>

            <div class="campo">
              <label>Icono</label>
              <input v-model="nuevoServicioCatalogo.icono" type="text" maxlength="2">
            </div>
          </div>

          <button class="boton-guardar catalogo-guardar" @click="guardarServicioCatalogo">
            {{ editandoCatalogo ? 'Guardar cambios' : 'Agregar servicio' }}
          </button>

          <div class="catalogo-lista">
            <div v-for="s in serviciosDisponibles" :key="s.nombre" class="catalogo-item">
              <span>{{ s.icono }} {{ s.nombre }}</span>
              <strong>${{ dinero(s.precio) }}</strong>

              <div>
                <button class="boton-editar" @click="editarServicioCatalogo(s)">✏️</button>
                <button class="boton-eliminar" @click="eliminarServicioCatalogo(s.nombre)">🗑️</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

   
    <div v-show="mostrarModalCaja" class="modal-fondo">
      <div class="modal modal-pequeno">
        <div class="modal-cabecera">
          <div>
            <h2>💵 Cierre de caja</h2>
            <p>Resumen de los servicios de hoy</p>
          </div>
          <button class="cerrar" @click="mostrarModalCaja = false">×</button>
        </div>

        <div class="contenido-modal-simple">
          <div class="resumen-caja">
            <div>
              <span>💵 Total efectivo</span>
              <strong>${{ dinero(totalMetodoPago('Efectivo')) }}</strong>
            </div>

            <div>
              <span>📱 Total transferencia</span>
              <strong>${{ dinero(totalMetodoPago('Transferencia')) }}</strong>
            </div>

            <div class="dato-caja">
              <span>💳 Total tarjeta</span>
              <strong>${{ dinero(totalMetodoPago('Tarjeta')) }}</strong>
            </div>

            <div>
              <span>⚠️ Pendientes por cobrar</span>
              <strong>${{ dinero(totalPendiente) }}</strong>
            </div>

            <div>
              <span>📋 Servicios de hoy</span>
              <strong>{{ serviciosDeHoy.length }}</strong>
            </div>
          </div>

          <div class="botones-formulario">
            <button class="boton-cancelar" @click="mostrarModalCaja = false">
              Cancelar
            </button>
            <button class="boton-guardar" @click="archivarServiciosDelDia">
              📦 Archivar servicios del día
            </button>
          </div>
        </div>
      </div>
    </div>

 


    <div v-show="mostrarModalCalificar" class="modal-fondo">
      <div class="modal modal-pequeno">
        <div class="modal-cabecera">
          <div>
            <h2>⭐ Calificar servicio</h2>
            <p v-if="servicioCalificando">
              {{ servicioCalificando.nombre }} - {{ textoServicios(servicioCalificando.servicios) }}
            </p>
          </div>
          <button class="cerrar" @click="cerrarModalCalificar">×</button>
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
          <button type="button" class="boton-cancelar" @click="cerrarModalCalificar">
            Cancelar
          </button>
          <button type="button" class="boton-guardar" @click="confirmarCalificacion">
            Confirmar
          </button>
        </div>
      </div>
    </div>



    <div v-show="mostrarModalEliminar" class="modal-fondo">
      <div class="modal modal-pequeno">
        <div class="modal-cabecera">
          <div>
            <h2>🗑️ Eliminar servicio</h2>
            <p>Esta acción no se puede deshacer</p>
          </div>
          <button class="cerrar" @click="cerrarModalEliminar">×</button>
        </div>

        <div class="mensaje-confirmacion">
          ¿Está seguro de que desea eliminar este servicio?
        </div>

        <div class="botones-formulario">
          <button type="button" class="boton-cancelar" @click="cerrarModalEliminar">
            Cancelar
          </button>
          <button type="button" class="boton-eliminar-confirmar" @click="confirmarEliminacion">
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
  color: rgb(252, 250, 250);
}

.informacion {
  max-width: 1200px;
  margin: 25px auto;
  padding: 0 20px;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 15px;
}

.tarjeta-info {
  background: rgb(248, 247, 247);
  padding: 20px;
  border-radius: 12px;
  display: flex;
  gap: 15px;
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
  max-width: 1600px;
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
  padding-top: 20px;
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
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
}

.servicio-card {
  background: white;
  border-radius: 14px;
  padding: 22px;
  border-left: 5px solid #df4747;
  box-shadow: 0 3px 12px rgba(0, 0, 0, 0.07);
}

.pago-pagado {
  border-left-color: #3b9c68;
}

.pago-pendiente {
  border-left-color: #ffaf01;
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

.grupo-tarjetas {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  padding: 2px;
  border-radius: 4px;
  border: 2px solid transparent;
}

.tarjeta-servicio {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  text-align: center;
  background: white;
  border: 2px solid #E2D6BE;
  border-radius: 10px;
  padding: 14px 10px;
  cursor: pointer;
  font-weight: normal;
  transition: border-color 0.15s ease, background 0.15s ease, transform 0.1s ease;
}

.tarjeta-servicio:hover {
  border-color: #C9BCA1;
  transform: translateY(-2px);
}

.check-servicio {
  position: absolute;
  top: 6px;
  right: 8px;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  border: 2px solid #C9BCA1;
  font-size: 11px;
  line-height: 14px;
  color: transparent;
  background: white;
}

.icono-servicio {
  font-size: 24px;
}

.nombre-servicio {
  font-size: 13px;
  font-weight: 600;
  color: #050505;
}

.precio-servicio {
  font-size: 12px;
  color: #a27442;
  font-weight: bold;
}

.tarjeta-seleccionada {
  border-color: #9C6B2E;
  background: #fdf6ea;
}

.tarjeta-seleccionada .check-servicio {
  background: #9C6B2E;
  border-color: #9C6B2E;
  color: white;
}

.tarjeta-deshabilitada {
  cursor: not-allowed;
  opacity: 0.45;
}

.tarjeta-deshabilitada:hover {
  transform: none;
  border-color: #E2D6BE;
}

.nota-conflicto {
  font-size: 10px;
  color: #a53c3c;
  text-transform: uppercase;
  font-weight: bold;
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

.grupo-tarjetas.campo-error {
  border-color: #e53935 !important;
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

  .grupo-tarjetas {
    grid-template-columns: repeat(2, 1fr);
  }

}

.herramientas {
  width: 95%;
  max-width: 1600px;
  margin: 20px auto;
  padding: 15px;
  background: white;
  border-radius: 12px;
  display: flex;
  justify-content: space-between;
  gap: 15px;
  flex-wrap: wrap;
  box-shadow: 0 2px 8px #ddd;
}

.ordenamiento,
.botones-extra {
  display: flex;
  gap: 8px;
  align-items: center;
  flex-wrap: wrap;
}

.herramientas button {
  border: 1px solid #C9BCA1;
  background: #EFE7D8;
  padding: 9px 12px;
  border-radius: 6px;
  cursor: pointer;
}

.herramientas button:hover {
  background: #E2D6BE;
}

.panel-alerta,
.panel-comisiones {
  width: 95%;
  max-width: 1600px;
  margin: 15px auto;
  background: white;
  border-radius: 12px;
  padding: 18px;
  box-shadow: 0 2px 8px #ddd;
}

.panel-alerta {
  border-left: 5px solid #d39a35;
}

.panel-alerta h3,
.panel-comisiones h3 {
  margin-top: 0;
}

.lista-deudas {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.lista-deudas div,
.comision-card {
  background: #fff8e7;
  padding: 12px;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.comisiones-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

.comision-card {
  background: #f7f4f0;
}

.comision-card b {
  font-size: 20px;
}

.lista-turnos {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.separador-turno {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 3px solid #9C6B2E;
  padding: 10px 5px;
  margin-top: 15px;
}

.separador-turno h2 {
  margin: 0;
}

.separador-turno span {
  color: #777;
}

.alerta-fidelidad {
  background: #fff2c9;
  color: #805d00;
  border-left: 4px solid #d79a27;
  padding: 12px;
  margin: 0 25px 18px;
  font-weight: bold;
}

.fotos-card {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  border-top: 1px solid #eee;
  padding-top: 15px;
  margin-top: 5px;
}

.fotos-card div {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.fotos-card img {
  width: 100%;
  height: 150px;
  object-fit: cover;
  border-radius: 8px;
}

.vista-foto {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-top: 8px;
}

.vista-foto img {
  width: 100px;
  height: 80px;
  object-fit: cover;
  border-radius: 6px;
}

.vista-foto button {
  border: none;
  background: #f8dddd;
  color: #a53c3c;
  padding: 7px 10px;
  border-radius: 5px;
}

.contenido-modal-simple {
  padding: 0 25px 25px;
}

.contenido-modal-simple > label {
  display: block;
  font-weight: bold;
  margin-bottom: 7px;
}

.contenido-modal-simple > input {
  width: 100%;
  border: none;
  border-bottom: 2px solid hsl(209, 100%, 56%);
  padding: 10px 4px;
  background: #f5f5f5;
}

.resultado-historial {
  margin-top: 20px;
  background: #f7f4f0;
  border-radius: 8px;
  padding: 20px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
  text-align: center;
}

.resultado-historial strong {
  font-size: 24px;
}

.catalogo-guardar {
  margin-top: 20px;
}

.catalogo-lista {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.catalogo-item {
  display: grid;
  grid-template-columns: 1fr auto auto;
  align-items: center;
  gap: 10px;
  padding: 10px;
  background: #f7f4f0;
  border-radius: 7px;
}

.resumen-caja {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.resumen-caja div {
  background: #f7f4f0;
  padding: 14px;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  gap: 10px;
}

.boton-calificar {
  border: none;
  padding: 9px 14px;
  border-radius: 7px;
  font-weight: bold;
  background: #fff1bf;
  color: #8b6515;
}

@media (max-width: 800px) {
  .lista-deudas,
  .comisiones-grid {
    grid-template-columns: 1fr;
  }

  .herramientas {
    flex-direction: column;
  }
}

@media (max-width: 550px) {
  .fotos-card,
  .resultado-historial {
    grid-template-columns: 1fr;
  }

  .catalogo-item {
    grid-template-columns: 1fr;
  }
}

</style>
