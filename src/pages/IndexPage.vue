<template>
  <div class="row justify-evenly">
    <q-card class="col-6">
      <q-card-section class="column items-center">
        <div class="text-h5">Gasto total al año en pensiones</div>
        <p class="text-h3">{{ numberToStringWithCommas(pensionCostPerYear) }}€</p>
        <p class="text-h6">{{ numberToSpanishWords(pensionCostPerYear) }} euros</p>
      </q-card-section>
    </q-card>

    <q-card class="col-5">
      <q-card-section class="column items-center">
        <div class="text-h5">Gasto total al año en pensiones de jubilación</div>
        <p class="text-h3">{{ numberToStringWithCommas(retirementPensionCostPerYear) }}€</p>
        <p class="text-h6">{{ numberToSpanishWords(retirementPensionCostPerYear) }} euros</p>
      </q-card-section>
    </q-card>

    <q-card class="col-8 q-mt-lg">
      <q-card-section class="column items-center">
        <div class="text-h5">Calculador de días de pensiones</div>
        <div class="row items-center q-gutter-sm q-mt-md">
          <q-input outlined v-model="ammount" type="number" />
          <p>(En euros)</p>
        </div>
        <p class="text-h6 q-mt-md">
          Con {{ numberToStringWithCommas(ammount) }}€ se pueden financiar
          <strong>{{ daysHoursMinutes(ammount, pensionCostPerYear) }}</strong>
          de pensiones.
        </p>
        <p class="text-h6">
          Con {{ numberToStringWithCommas(ammount) }}€ se pueden financiar
          <strong>{{ daysHoursMinutes(ammount, retirementPensionCostPerYear) }}</strong>
          de pensiones de jubilación.
        </p>
        <p class="text-h6">
          Con {{ numberToStringWithCommas(ammount) }}€ se pueden financiar
          <strong>{{ numberToStringWithCommas(Math.floor(ammount / meanPension)) }}</strong>
          pensionistas por año. Un
          {{ ((Math.floor(ammount / meanPension) / pensionists) * 100).toFixed(2) }}% de todos los
          pensionistas.
        </p>
      </q-card-section>
    </q-card>

    <q-card class="col-2 q-mt-lg">
      <q-card-section class="column items-center">
        <p class="text-h6">Hay {{ numberToStringWithCommas(pensionists) }} pensionistas</p>
        <p class="text-h6">
          Un {{ ((pensionists / population) * 100).toFixed(2) }}% de la población española
        </p>
        <p class="text-h6">
          Pensión media:
          {{ numberToStringWithCommas(meanPension) }}€/año
        </p>
      </q-card-section>
    </q-card>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { data } from '../data/data.js'
import { dataPensionistas } from '../data/data-pensionistas.js'

onMounted(() => {
  pensionCostPerYear.value = calculateTotalCostPerYear()
  retirementPensionCostPerYear.value = calculateRetirementPensionCostPerYear()
  pensionists.value = calculateTotalRetiredPeople()
  meanPension.value = Math.floor(pensionCostPerYear.value / pensionists.value)
})

const pensionCostPerYear = ref(0)
const retirementPensionCostPerYear = ref(0)
const pensionists = ref(0)
const population = 49315949
const meanPension = ref(0)

const ammount = ref(0)

function numberToStringWithCommas(x) {
  return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.')
}

function calculateTotalCostPerYear() {
  const lastTwelveMonths = data.Respuesta.Datos.Metricas[0].Datos.slice(-12)
  const totalLastTwelveMonths =
    lastTwelveMonths.reduce((sum, month) => sum + month.Valor, 0) * 1000000
  return Math.floor(totalLastTwelveMonths)
}

function calculateRetirementPensionCostPerYear() {
  const lastTwelveMonths = data.Respuesta.Datos.Metricas[1].Datos.slice(-12)
  const totalLastTwelveMonths =
    lastTwelveMonths.reduce((sum, month) => sum + month.Valor, 0) * 1000000
  return Math.floor(totalLastTwelveMonths)
}

function calculateTotalRetiredPeople() {
  return (
    dataPensionistas.Respuesta.Datos.Metricas[0].Datos[
      dataPensionistas.Respuesta.Datos.Metricas[0].Datos.length - 1
    ].Valor * 1000
  )
}

function numberToSpanishWords(num) {
  const unidades = [
    'cero',
    'uno',
    'dos',
    'tres',
    'cuatro',
    'cinco',
    'seis',
    'siete',
    'ocho',
    'nueve',
  ]
  const especiales = [
    'diez',
    'once',
    'doce',
    'trece',
    'catorce',
    'quince',
    'dieciséis',
    'diecisiete',
    'dieciocho',
    'diecinueve',
  ]
  const decenas = [
    '',
    '',
    'veinte',
    'treinta',
    'cuarenta',
    'cincuenta',
    'sesenta',
    'setenta',
    'ochenta',
    'noventa',
  ]
  const centenas = [
    '',
    'ciento',
    'doscientos',
    'trescientos',
    'cuatrocientos',
    'quinientos',
    'seiscientos',
    'setecientos',
    'ochocientos',
    'novecientos',
  ]

  if (num < 10) return unidades[num]
  if (num < 20) return especiales[num - 10]
  if (num < 100) {
    if (num < 30) {
      if (num === 20) return decenas[2]
      return 'veinti' + unidades[num % 10]
    }
    let d = Math.floor(num / 10)
    let u = num % 10
    return decenas[d] + (u ? ' y ' + unidades[u] : '')
  }
  if (num < 1000) {
    if (num === 100) return 'cien'
    let c = Math.floor(num / 100)
    let r = num % 100
    return centenas[c] + (r ? ' ' + numberToSpanishWords(r) : '')
  }
  if (num < 1000000) {
    let miles = Math.floor(num / 1000)
    let resto = num % 1000
    let milesStr = miles === 1 ? 'mil' : numberToSpanishWords(miles) + ' mil'
    return milesStr + (resto ? ' ' + numberToSpanishWords(resto) : '')
  }
  if (num < 1000000000) {
    let millones = Math.floor(num / 1000000)
    let resto = num % 1000000
    let millonesStr = millones === 1 ? 'un millón' : numberToSpanishWords(millones) + ' millones'
    return millonesStr + (resto ? ' ' + numberToSpanishWords(resto) : '')
  }
  if (num < 1000000000000) {
    let milesDeMillones = Math.floor(num / 1000000000)
    let resto = num % 1000000000
    let milesDeMillonesStr =
      milesDeMillones === 1 ? 'mil ' : numberToSpanishWords(milesDeMillones) + ' mil '
    return milesDeMillonesStr + (resto ? ' ' + numberToSpanishWords(resto) : '')
  }
  return 'Número demasiado grande'
}

function daysHoursMinutes(ammount, pensionType) {
  const days = Math.floor(ammount / (pensionType / 365))
  const hours = Math.floor((ammount % (pensionType / 365)) / (pensionType / 365 / 24))
  const minutes = Math.floor((ammount % (pensionType / 365 / 24)) / (pensionType / 365 / 24 / 60))
  let result = ''
  if (days > 0) result += days + ' días'
  if (hours > 0) result += (result ? ', ' : '') + hours + ' horas'
  if (minutes > 0) result += (result ? ' y ' : '') + minutes + ' minutos'
  return result ? result : 'menos de un minuto'
}
</script>

<style scoped>
p {
  text-align: center;
}
</style>
