<script setup lang="ts">
import PlanNode from "@/components/PlanNode.vue"

defineProps(["node", "plan", "viewOptions"])
</script>

<template>
  <foreignObject
    :x="(node.x || 0) - (node.xSize || 0) / 2"
    :y="node.y"
    width="10px"
    height="10px"
  >
    <plan-node
      :node="node.data"
      :plan="plan"
      :viewOptions="viewOptions"
      class="position-fixed"
    ></plan-node>
  </foreignObject>
</template>

<style>
foreignObject {
  overflow: visible;
}
</style>
