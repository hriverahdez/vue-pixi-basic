<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'

import * as PIXI from 'pixi.js'
import { type GraphicsInst } from 'vue3-pixi'

const icon64 =
  'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAADPSURBVHgBtVFBDoIwEByIiSfrA2xAH0A8C1E+4l+Es+EXHrkazsTg3fAAhfABevOEQFISlNImxEmazna3050uMBHa2jBO9e5J6rxXnvsigQoKqAW0oXOdk2eWtUsUizDjZGtZeKQpzkEAVFV3JkPPAqUUtyRp+d5xUBRFV2jQ1XgH3yCE9OL4+P6pcS/zvkDzehiGYGWJaxS1FhhjULLAP2xjmhiKRRaUx8gFXcdGnNw73ozRl11ekoUwp4kSB3un1BnGBDjGuI6J+J8FVXwArGpkXrM0Y1EAAAAASUVORK5CYII='

interface Props {
  node: any
  position: { x: number; y: number }
  name: string
  icon?: string | null
}

const props = withDefaults(defineProps<Props>(), {
  icon: icon64,
  type: null,
  link: null,
})

const emit = defineEmits<{
  contextmenu: [event: PIXI.FederatedPointerEvent]
}>()

const translatedPosition = computed(() => {
  return props.position
})

const backgroundColor = computed(() => {
  // In case of unknown type, use Hibiscus Dark
  return 'red'
})

const texture = ref<PIXI.Texture<PIXI.Resource>>()

const NODE_ICON_SIZE = 16
const scaleFactor = ref(0)

onMounted(() => {
  if (props.icon) {
    const imageLocal = new window.Image()

    imageLocal.src = props.icon
    imageLocal.onload = () => {
      texture.value = PIXI.Texture.from(props.icon!)
      scaleFactor.value =
        NODE_ICON_SIZE / Math.max(imageLocal.naturalWidth, imageLocal.naturalHeight)
    }
  }
})

const hoverSize = ref({ x: -10, y: -10, width: 40, height: 40 })

const renderHoverArea = (graphics: GraphicsInst) => {
  graphics.clear()

  graphics.drawRect(
    hoverSize.value.x,
    hoverSize.value.y,
    hoverSize.value.width,
    hoverSize.value.height,
  )
  graphics.endFill()
}

const nodeNamePosition = computed(() => {
  const { x, y } = translatedPosition.value
  return {
    x: x + hoverSize.value.x + hoverSize.value.width / 2,
    y: y - 32 / 2 - 4,
  }
})

const hitArea = computed(
  () =>
    new PIXI.Rectangle(
      hoverSize.value.x,
      hoverSize.value.y,
      hoverSize.value.width,
      hoverSize.value.height,
    ),
)
</script>

<template>
  <Container @rightclick="emit('contextmenu', $event)">
    <Graphics :hit-area="hitArea" :position="translatedPosition" @render="renderHoverArea" />

    <Text
      :position="nodeNamePosition"
      :resolution="3"
      :scale="0.5"
      :style="{
        fill: 'black',
        fontFamily: 'Roboto Condensed',
        fontSize: 32,
        fontWeight: 'bold',
      }"
      :anchor="{ x: 0.5, y: 0.5 }"
    >
      {{ name }}
    </Text>

    <Graphics
      :name="node.id"
      :position="translatedPosition"
      event-mode="none"
      @render="
        (graphics: GraphicsInst) => {
          graphics.clear()
          graphics.beginFill(backgroundColor)
          graphics.drawRoundedRect(0, 0, 32, 32, 4)
          graphics.endFill()
        }
      "
    />

    <Sprite
      v-if="texture"
      :texture="texture as any"
      :anchor="0.5"
      event-mode="none"
      :scale="scaleFactor"
      :x="translatedPosition.x + 32 / 2"
      :y="translatedPosition.y + 32 / 2"
    />
  </Container>
</template>
