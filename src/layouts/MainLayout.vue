<template>
  <q-layout view="lHh Lpr lFf">
    <q-header class="bg-grey-2 q-mb-lg" elevated>
      <div class="row justify-center">
        <h1 class="text-black text-h4">Días de Pensiones</h1>
      </div>
      <div class="row justify-center">
        <q-toolbar class="row justify-center q-gutter-lg q-mb-sm">
          <q-btn
            v-for="button in buttons"
            :key="button.label"
            :label="button.label"
            :to="button.path"
            flat
            class="text-black col-4 col-md-1"
            :class="{ 'bg-white': $route.path === button.path }"
          />
        </q-toolbar>
      </div>
    </q-header>

    <q-page-container>
      <router-view class="q-pa-lg" />
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { onMounted } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const buttons = [
  { label: 'Inicio', path: '/' },
  { label: 'Comparador', path: '/comparador' },
]

onMounted(() => {
  console.log('Current route:', route.path)
})
</script>
