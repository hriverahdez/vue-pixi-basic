<script setup lang="ts">
import type { GraphicsInst } from 'vue3-pixi'
import { Application } from 'vue3-pixi'
import * as node from './node.json'
import Node from './Node.vue'

function getRand(min = 10, max = 500) {
  return Math.random() * (max - min) + min
}
</script>

<template>
  <Application
    ref="pixiApp"
    :background-color="0xffffff"
    :width="1000"
    :height="1000"
    :resolution="1"
  >
    <Container name="contentBounds">
      <Graphics
        :x="0"
        :y="0"
        @render="
          (graphics: GraphicsInst) => {
            graphics.clear()
            graphics.beginFill(0x000000)
            graphics.drawRect(0, 0, 10, 10)
            graphics.endFill()
          }
        "
      />

      <template v-for="n in 20" :key="n">
        <Node :node="node" :position="{ x: getRand(), y: getRand() }" :name="`node${n}`" />
      </template>
    </Container>
  </Application>
</template>
