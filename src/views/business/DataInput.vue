<template>
</template>

<script>
import {computed, onMounted} from "vue";
import {useRoute} from "vue-router";

export default {
  name: "DataInput",

  setup() {
    const route = useRoute()

    const graphInfo = computed(() => route.params.graphInfo )

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
