<template>
  <div>
    {{ msg }}
  </div>
</template>

<script>
import axios from 'axios';
import {onMounted, ref} from "vue";

export default {
  setup() {
    const msg = ref("");

    const hello = () => {
      axios.get("/spc/test/jeecgDemo/hello").then((res) => {
        console.log("success!!!");
        console.log(res);
        const data = res.data;
        if (data.success) {
          msg.value = data.result;
        }
      })
    }

    onMounted(() => {
      hello();
    });

    return {
      msg
    };
  }
}

</script>
