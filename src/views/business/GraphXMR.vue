<template>

</template>

<script>
import {useRoute} from "vue-router";
import {onMounted, ref} from "vue";

export default {
  name: "GraphXMR",

  setup() {
    const route = useRoute()
    const graphData = ref(JSON.parse(route.params.graphData))

    onMounted(() => {
      console.log(graphData.value)
    })
  }
}
</script>

<style scoped>

</style>
