<template>

</template>

<script>
import {useRoute} from "vue-router";
import {onMounted, ref} from "vue";
import * as echarts  from "echarts"

export default {
  name: "GraphXRXSMedium",

  setup() {
    const route = useRoute()
    const graphData = ref(JSON.parse(route.params.graphData))

    onMounted(() => {
      console.log(graphData.value);
    })

    return {
      graphData,
    }
  }
}
</script>

<style scoped>

</style>
