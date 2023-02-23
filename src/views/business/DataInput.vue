<template>
  <h1>{{graphInfo.graphType}}</h1>
  <h1>{{graphInfo.subgroupTotal}}</h1>
</template>

<script>
import {computed, onMounted, ref} from "vue";
import {useRoute} from "vue-router";

export default {
  name: "DataInput",

  setup() {
    const route = useRoute()
    const graphInfo = ref(JSON.parse(route.params.graphInfo))

    onMounted(() => {
      console.log(graphInfo.value)
    })

    return {
      graphInfo
    }
  }
}
</script>
<style scoped>

</style>
