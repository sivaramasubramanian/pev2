<script lang="ts" setup>
import { ref } from "vue"
import { directive as vTippy } from "vue-tippy"
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome"
import sqlFormatter from "@sqltools/formatter"

interface Props {
  content: string
}
const props = defineProps<Props>()
const copied = ref<boolean>(false)

function formatQuery() {
  let formattedString = sqlFormatter.format(props.content, {
    language: "sql",
    reservedWordCase: "upper",
  })
  copied.value = true
  window.setTimeout(() => {
    copied.value = false
  }, 2000)

  return formattedString
}
</script>

<template>
  <div class="copy position-absolute" style="top: 0; right: 50px">
    <button
      name="formatQueryButton"
      id="formatQueryButton"
      class="btn btn-outline-secondary bg-light btn-sm m-2"
      :class="copied ? 'd-none' : 'd-block'"
      @click="$emit('formatted', formatQuery())"
    >
      <font-awesome-icon fixed-width icon="align-left" />
    </button>
    <button
      class="btn btn-outline-secondary bg-light btn-sm m-2"
      :class="copied ? 'd-block' : 'd-none'"
      v-tippy="{ placement: 'bottom', arrow: true, content: 'formatted' }"
    >
      <font-awesome-icon fixed-width icon="check" class="text-success" />
    </button>
  </div>
</template>
