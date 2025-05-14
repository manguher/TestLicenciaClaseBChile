<template>
  <div class="max-w-3xl mx-auto">
    <!-- Pantalla de inicio -->
    <div v-if="!testIniciado && !testFinalizado" class="text-center py-8">
      <h1 class="text-3xl font-bold mb-6">Test Teórico Licencia Clase B</h1>
      <p class="mb-4">Este test contiene 35 preguntas seleccionadas aleatoriamente.</p>
      <p class="mb-6">Tienes que responder correctamente al menos 30 preguntas para aprobar.</p>
      <Button type="button" btnType="secondary" @click="iniciarTest">Comenzar Test</Button>
    </div>

    <!-- Pantalla del test -->
    <div v-else-if="testIniciado && !testFinalizado" class="py-4">
      <div class="flex justify-between items-center mb-4">
        <h2 class="text-xl font-bold">Pregunta {{ actual + 1 }} de {{ preguntasTest.length }}</h2>
        <div class="text-sm">
          <span>Respondidas: {{ respuestasUsuario.filter(r => r !== null).length }}</span>
          <span class="mx-2">|</span>
          <Button 
            type="button" 
            btnType="secondary" 
            @click="finalizarTest" 
            class="text-sm py-1 px-3"
          >
            Finalizar Test
          </Button>
        </div>
      </div>

      <div class="bg-white rounded-lg shadow-md p-6 mb-4">
        <p class="text-lg mb-4">{{ preguntaActual.pregunta }}</p>
        
        <div v-if="preguntaActual.imagen" class="mb-4 flex justify-center">
          <img :src="preguntaActual.imagen" alt="Imagen pregunta" class="max-w-full h-auto max-h-64" />
        </div>
        
        <div class="space-y-3">
          <div 
            v-for="(alt, idx) in preguntaActual.alternativas" 
            :key="idx" 
            class="border rounded-md p-3 hover:bg-gray-50 cursor-pointer"
            :class="{
              'border-green-500 bg-green-50': respondido && alt.correcta,
              'border-red-500 bg-red-50': respondido && !alt.correcta && seleccionadasIncluye(idx)
            }"
            @click="!respondido && seleccionarAlternativa(idx)"
          >
            <label class="flex items-start cursor-pointer w-full">
              <input
                :type="esMultiple ? 'checkbox' : 'radio'"
                :name="'alternativa-' + preguntaActual.id"
                :value="idx"
                :checked="seleccionadasIncluye(idx)"
                :disabled="respondido"
                class="mt-1 mr-3"
                @change="seleccionarAlternativa(idx)"
              />
              <span>{{ alt.alternativa }}</span>
            </label>
          </div>
        </div>
      </div>

      <div class="flex justify-between items-center">
        <Button 
          type="button" 
          btnType="secondary" 
          @click="preguntaAnterior" 
          :disabled="actual === 0"
          class="px-4"
        >
          Anterior
        </Button>

        <div v-if="respondido" class="text-center">
          <p v-if="esCorrecta" class="text-green-600 font-bold">¡Correcto!</p>
          <p v-else class="text-red-600 font-bold">Incorrecto</p>
        </div>
        <div v-else class="text-center">
          <Button 
            type="button" 
            btnType="secondary" 
            @click="verificarRespuesta" 
            :disabled="seleccionadas.length === 0"
          >
            Responder
          </Button>
        </div>

        <Button 
          type="button" 
          btnType="secondary" 
          @click="siguientePregunta" 
          :disabled="actual >= preguntasTest.length - 1"
          class="px-4"
        >
          Siguiente
        </Button>
      </div>

      <!-- Navegación rápida -->
      <div class="mt-8">
        <p class="text-sm mb-2">Navegación rápida:</p>
        <div class="flex flex-wrap gap-2">
          <button 
            v-for="(respuesta, index) in respuestasUsuario" 
            :key="index"
            @click="irAPregunta(index)"
            class="w-8 h-8 rounded-full flex items-center justify-center text-sm"
            :class="{
              'bg-green-500 text-white': respuesta === true,
              'bg-red-500 text-white': respuesta === false,
              'bg-gray-200': respuesta === null,
              'ring-2 ring-blue-500': index === actual
            }"
          >
            {{ index + 1 }}
          </button>
        </div>
      </div>
    </div>

    <!-- Pantalla de resultados -->
    <div v-else-if="testFinalizado" class="py-8">
      <h2 class="text-2xl font-bold mb-6 text-center">Resultados del Test</h2>
      
      <div class="bg-white rounded-lg shadow-md p-6 mb-6">
        <div class="text-center mb-6">
          <p class="text-3xl font-bold mb-2">
            {{ respuestasCorrectas }} / {{ preguntasTest.length }}
          </p>
          <p class="text-lg">
            {{ respuestasCorrectas >= 30 ? '¡Aprobado!' : 'No aprobado' }}
          </p>
          <p class="text-sm text-gray-600 mt-2">
            Necesitas al menos 30 respuestas correctas para aprobar
          </p>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div class="bg-green-50 p-4 rounded-md">
            <p class="font-bold text-green-800">Correctas: {{ respuestasCorrectas }}</p>
          </div>
          <div class="bg-red-50 p-4 rounded-md">
            <p class="font-bold text-red-800">Incorrectas: {{ preguntasTest.length - respuestasCorrectas }}</p>
          </div>
        </div>
      </div>

      <div class="flex justify-center space-x-4">
        <Button type="button" btnType="secondary" @click="verRevision">Ver revisión</Button>
        <Button type="button" btnType="secondary" @click="reiniciarTest">Nuevo test</Button>
      </div>

      <!-- Revisión de preguntas -->
      <div v-if="mostrarRevision" class="mt-8">
        <h3 class="text-xl font-bold mb-4">Revisión de preguntas</h3>
        
        <div v-for="(pregunta, index) in preguntasTest" :key="pregunta.id" class="bg-white rounded-lg shadow-md p-6 mb-4">
          <div class="flex justify-between items-start mb-2">
            <h4 class="text-lg font-bold">Pregunta {{ index + 1 }}</h4>
            <span 
              class="px-3 py-1 rounded-full text-sm font-bold"
              :class="{
                'bg-green-100 text-green-800': respuestasUsuario[index] === true,
                'bg-red-100 text-red-800': respuestasUsuario[index] === false
              }"
            >
              {{ respuestasUsuario[index] ? 'Correcta' : 'Incorrecta' }}
            </span>
          </div>
          
          <p class="mb-4">{{ pregunta.pregunta }}</p>
          
          <div v-if="pregunta.imagen" class="mb-4 flex justify-center">
            <img :src="pregunta.imagen" alt="Imagen pregunta" class="max-w-full h-auto max-h-48" />
          </div>
          
          <div class="space-y-2">
            <div 
              v-for="(alt, idx) in pregunta.alternativas" 
              :key="idx"
              class="p-3 rounded-md"
              :class="{
                'bg-green-100': alt.correcta,
                'bg-red-100': !alt.correcta && respuestasSeleccionadas[index] && respuestasSeleccionadas[index].includes(idx)
              }"
            >
              <div class="flex items-start">
                <div class="mr-2 mt-1">
                  <svg v-if="alt.correcta" class="w-4 h-4 text-green-600" fill="currentColor" viewBox="0 0 20 20">
                    <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"></path>
                  </svg>
                  <svg v-else-if="respuestasSeleccionadas[index] && respuestasSeleccionadas[index].includes(idx)" class="w-4 h-4 text-red-600" fill="currentColor" viewBox="0 0 20 20">
                    <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd"></path>
                  </svg>
                </div>
                <span>{{ alt.alternativa }}</span>
              </div>
            </div>
          </div>
        </div>
        
        <div class="flex justify-center mt-6">
          <Button type="button" btnType="secondary" @click="reiniciarTest">Nuevo test</Button>
        </div>
      </div>
    </div>

    <!-- Pantalla de carga -->
    <div v-else class="flex justify-center items-center py-16">
      <p>Cargando preguntas...</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import Button from './Button.vue'

// Estados del test
const todasLasPreguntas = ref([])  // Todas las preguntas del JSON
const preguntasTest = ref([])      // 35 preguntas seleccionadas para el test
const actual = ref(0)              // Índice de la pregunta actual
const seleccionadas = ref([])      // Alternativas seleccionadas por el usuario
const respuestasUsuario = ref([])  // Array con las respuestas del usuario (true/false/null)
const respuestasSeleccionadas = ref([]) // Array con los índices seleccionados por el usuario
const respondido = ref(false)      // Si la pregunta actual ha sido respondida
const esCorrecta = ref(false)      // Si la respuesta actual es correcta

// Estados de navegación
const testIniciado = ref(false)    // Si el test ha iniciado
const testFinalizado = ref(false)  // Si el test ha finalizado
const mostrarRevision = ref(false) // Si se muestra la revisión de preguntas

// URL del JSON
const preguntasURL = '/vue-landing-page/preguntas_licencia_b.json'

// Computados
const preguntaActual = computed(() => preguntasTest.value[actual.value] || {})

const esMultiple = computed(() => {
  if (!preguntaActual.value.alternativas) return false
  return preguntaActual.value.alternativas.filter(a => a.correcta).length > 1
})

const respuestasCorrectas = computed(() => {
  return respuestasUsuario.value.filter(r => r === true).length
})

// Métodos
function iniciarTest() {
  // Seleccionar 35 preguntas aleatorias
  preguntasTest.value = seleccionarPreguntasAleatorias(todasLasPreguntas.value, 35)
  
  // Inicializar arrays de respuestas
  respuestasUsuario.value = Array(preguntasTest.value.length).fill(null)
  respuestasSeleccionadas.value = Array(preguntasTest.value.length).fill(null)
  
  // Iniciar el test
  actual.value = 0
  seleccionadas.value = []
  respondido.value = false
  testIniciado.value = true
  testFinalizado.value = false
  mostrarRevision.value = false
}

function seleccionarPreguntasAleatorias(preguntas, cantidad) {
  // Hacer una copia para no modificar el original
  const preguntasCopia = [...preguntas]
  
  // Mezclar el array (algoritmo Fisher-Yates)
  for (let i = preguntasCopia.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[preguntasCopia[i], preguntasCopia[j]] = [preguntasCopia[j], preguntasCopia[i]]
  }
  
  // Tomar las primeras 'cantidad' preguntas
  return preguntasCopia.slice(0, cantidad)
}

function seleccionadasIncluye(idx) {
  if (Array.isArray(seleccionadas.value)) {
    return seleccionadas.value.includes(idx)
  }
  return seleccionadas.value === idx
}

function seleccionarAlternativa(idx) {
  if (esMultiple.value) {
    // Para preguntas de selección múltiple
    const index = seleccionadas.value.indexOf(idx)
    if (index === -1) {
      seleccionadas.value.push(idx)
    } else {
      seleccionadas.value.splice(index, 1)
    }
  } else {
    // Para preguntas de selección única
    seleccionadas.value = [idx]
  }
  
  // Guardar las selecciones actuales en respuestasSeleccionadas
  // incluso si no se ha presionado el botón "Responder"
  if (!respuestasSeleccionadas.value[actual.value]) {
    respuestasSeleccionadas.value[actual.value] = []
  }
  respuestasSeleccionadas.value[actual.value] = [...seleccionadas.value]
}

function verificarRespuesta() {
  // Obtener índices de alternativas correctas
  const correctas = preguntaActual.value.alternativas
    .map((a, i) => a.correcta ? i : null)
    .filter(i => i !== null)
    .sort()
  
  // Ordenar selecciones del usuario
  let seleccion = [...seleccionadas.value].sort()
  
  // Verificar si la respuesta es correcta
  esCorrecta.value = JSON.stringify(correctas) === JSON.stringify(seleccion)
  
  // Guardar resultado y selecciones
  respuestasUsuario.value[actual.value] = esCorrecta.value
  respuestasSeleccionadas.value[actual.value] = [...seleccionadas.value]
  
  // Marcar como respondida
  respondido.value = true
}

function siguientePregunta() {
  if (actual.value < preguntasTest.value.length - 1) {
    actual.value++
    seleccionadas.value = respuestasSeleccionadas.value[actual.value] || []
    respondido.value = respuestasUsuario.value[actual.value] !== null
    esCorrecta.value = respuestasUsuario.value[actual.value] === true
  }
}

function preguntaAnterior() {
  if (actual.value > 0) {
    actual.value--
    seleccionadas.value = respuestasSeleccionadas.value[actual.value] || []
    respondido.value = respuestasUsuario.value[actual.value] !== null
    esCorrecta.value = respuestasUsuario.value[actual.value] === true
  }
}

function irAPregunta(index) {
  actual.value = index
  seleccionadas.value = respuestasSeleccionadas.value[index] || []
  respondido.value = respuestasUsuario.value[index] !== null
  esCorrecta.value = respuestasUsuario.value[index] === true
}

function finalizarTest() {
  // Evaluar todas las respuestas seleccionadas que no han sido verificadas
  evaluarRespuestasPendientes()
  
  // Verificar si hay preguntas sin responder (después de la evaluación automática)
  const preguntasSinResponder = respuestasUsuario.value.filter(r => r === null).length
  
  if (preguntasSinResponder > 0) {
    if (!confirm(`Tienes ${preguntasSinResponder} preguntas sin responder. ¿Seguro que deseas finalizar el test?`)) {
      return
    }
  }
  
  testIniciado.value = false
  testFinalizado.value = true
}

function verRevision() {
  mostrarRevision.value = true
}

function evaluarRespuestasPendientes() {
  // Recorrer todas las preguntas
  for (let i = 0; i < preguntasTest.value.length; i++) {
    // Si la pregunta no ha sido respondida pero tiene selecciones
    if (respuestasUsuario.value[i] === null && respuestasSeleccionadas.value[i] && respuestasSeleccionadas.value[i].length > 0) {
      // Guardar la pregunta actual
      const preguntaActualTemp = actual.value
      const respondidoTemp = respondido.value
      
      // Cambiar a la pregunta que vamos a evaluar
      actual.value = i
      seleccionadas.value = respuestasSeleccionadas.value[i]
      
      // Evaluar la respuesta
      verificarRespuestaAutomatica()
      
      // Restaurar la pregunta actual
      actual.value = preguntaActualTemp
      respondido.value = respondidoTemp
    }
  }
}

function verificarRespuestaAutomatica() {
  // Obtener índices de alternativas correctas
  const correctas = preguntaActual.value.alternativas
    .map((a, i) => a.correcta ? i : null)
    .filter(i => i !== null)
    .sort()
  
  // Ordenar selecciones del usuario
  let seleccion = [...seleccionadas.value].sort()
  
  // Verificar si la respuesta es correcta
  const esCorrectaTemp = JSON.stringify(correctas) === JSON.stringify(seleccion)
  
  // Guardar resultado y selecciones
  respuestasUsuario.value[actual.value] = esCorrectaTemp
  respuestasSeleccionadas.value[actual.value] = [...seleccionadas.value]
}

function reiniciarTest() {
  iniciarTest()
}

onMounted(async () => {
  try {
    const resp = await fetch(preguntasURL)
    todasLasPreguntas.value = await resp.json()
  } catch (error) {
    console.error('Error al cargar preguntas:', error)
  }
})
</script>

<style scoped>
/* Estilos adicionales si los necesitas */
</style>
