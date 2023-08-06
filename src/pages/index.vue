<script lang="ts">
import { computed, onMounted, ref } from 'vue'

export default {
  setup() {
    const size = ref(150)
    const borderThickness = ref(10)
    const ellipseShadow = computed(() => {
      return {
        transition: 'all 1.5s ease-out',
        boxShadow: '0 95px 25px rgba(0, 0, 0, 0.1)',
        height: '25%',
        bottom: '-125%',
      }
    })

    onMounted(() => {
      navigator.mediaDevices.getUserMedia({ audio: true }).then((stream) => {
        const audioContext = new AudioContext()
        const analyser = audioContext.createAnalyser()
        const source = audioContext.createMediaStreamSource(stream)
        source.connect(analyser)

        const dataArray = new Uint8Array(analyser.frequencyBinCount)

        const update = () => {
          analyser.getByteTimeDomainData(dataArray)

          const rms = Math.sqrt(dataArray.reduce((sum, value) => sum + ((value - 128) * (value - 128)), 0) / dataArray.length)
          size.value = 150 + rms * 6

          requestAnimationFrame(update)
        }

        update()
      })
    })

    return {
      size,
      borderThickness,
      ellipseShadow,
    }
  },
}
</script>

<template>
  <div class="h-screen flex items-center justify-center bg-orange-600">
    <div :style="{ width: `${size}px`, height: `${size}px`, borderWidth: `${borderThickness}px` }" class="relative border-white rounded-full transition-all duration-300 ease-out">
      <div :style="ellipseShadow" class="scaleY(-1) absolute bottom-0 left-0 h-1/2 w-full transform rounded-b-full" />
    </div>
  </div>
</template>
