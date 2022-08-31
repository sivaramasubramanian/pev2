<script lang="ts" setup>
import VLink from "../components/VLink.vue"

interface Props {
  title?: string
}

const props = defineProps<Props>()
</script>

<template>
  <div class="d-flex flex-column vh-100">
    <div class="navbar container">
      <div v-if="props?.title" class="text-center ml-auto">
        Plan: {{ props?.title }}
      </div>
      <v-link class="btn btn-link ml-auto" to="/">Home</v-link>
      <v-link class="btn btn-link" to="/about">About</v-link>
      <a class="btn btn-link" href="https://github.com/sivaramasubramanian/pev2"
        >Github</a
      >
    </div>
    <slot></slot>
  </div>
</template>
