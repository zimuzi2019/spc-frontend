<template>

  <p style="margin: 15px; text-align: center; font-size: large">
    <strong>控制图基本信息</strong>
  </p>

  <a-descriptions bordered column="2">
    <a-descriptions-item label="控制图类型">{{ graphTypeText }}</a-descriptions-item>
    <a-descriptions-item label="子组总数">{{ graphInfo.subgroupTotal }}</a-descriptions-item>

    <a-descriptions-item label="子组容量" v-if="graphInfo.graphType === 'X-R' || graphInfo.graphType === 'X-S' || graphInfo.graphType === '中位数' || graphInfo.graphType === 'nP' || graphInfo.graphType === 'C'">
      {{ graphInfo.subgroupCapacity }}
    </a-descriptions-item>



    <a-descriptions-item label="规范上限（USL）" v-if="graphInfo.graphType === 'X-R' || graphInfo.graphType === 'X-S' || graphInfo.graphType === '中位数' || graphInfo.graphType === 'X-MR'">
      {{ graphInfo.USL }}
    </a-descriptions-item>
    <a-descriptions-item label="中心值（SL）" v-if="graphInfo.graphType === 'X-R' || graphInfo.graphType === 'X-S' || graphInfo.graphType === '中位数' || graphInfo.graphType === 'X-MR'">
      {{ SL }}
    </a-descriptions-item>
    <a-descriptions-item label="规范下限（LSL）" v-if="graphInfo.graphType === 'X-R' || graphInfo.graphType === 'X-S' || graphInfo.graphType === '中位数' || graphInfo.graphType === 'X-MR'">
      {{ graphInfo.LSL }}
    </a-descriptions-item>
  </a-descriptions>


  <a-divider>请在下方输入框中绘制控制图所需数据</a-divider>


  <a-form layout="inline" style="margin: 30px">
    <a-form-item>
      <a-button type="primary" @click="handleClear">重置</a-button>
    </a-form-item>
    <a-form-item>
      <a-button type="primary" @click="handleSubmit">提交</a-button>
    </a-form-item>
  </a-form>




  <!-- 根据控制图类型决定加载哪种形式的输入格式 -->
  <div v-for="(row, index1) in dataArrayXRXSMedium" :key="index1" v-if="graphInfo.graphType === 'X-R' || graphInfo.graphType === 'X-S' || graphInfo.graphType ==='中位数'">
    <a-divider>子组编号{{ index1+1 }}</a-divider>
    <a-form layout="horizontal">
      <a-form-item v-for="(item, index2) in row" :label="`样品编号${String(index2+1)}`" :key="index2" :label-col="{ span: 2 }">
        <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArrayXRXSMedium[index1][index2]"></a-input>
      </a-form-item>
    </a-form>
  </div>


  <div v-if="graphInfo.graphType === 'X-MR'">
    <a-divider/>
    <a-form layout="horizontal">
      <a-form-item v-for="(item, index) in dataArrayXMR" :label="`样本测量值${String(index+1)}`" :key="index" :label-col="{ span: 2 }">
        <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArrayXMR[index]"></a-input>
      </a-form-item>
    </a-form>
  </div>

  <div v-if="graphInfo.graphType === 'C' || graphInfo.graphType === 'nP'">
    <a-divider/>
    <a-form layout="horizontal">
      <a-form-item v-for="(item, index) in dataArrayXMR" :label="`子组编号${String(index+1)}次品/缺陷数`" :key="index" :label-col="{ span: 3 }">
        <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArrayCnP[index]"></a-input>
      </a-form-item>
    </a-form>
  </div>

  <div v-if="graphInfo.graphType === 'P' || graphInfo.graphType === 'U'">
    <a-divider/>
    <a-form layout="horizontal">
      <a-form-item v-for="(item, index) in dataArrayPU1" :label="`子组编号${String(index+1)}`" :key="index" :label-col="{ span: 2 }">
        <a-input type="number" placeholder="请填入该子组所含样本容量，如：50" v-model:value="dataArrayPU1[index]"></a-input>
        <a-input type="number" placeholder="请填入该子组所含次品/缺陷数量，如：3" v-model:value="dataArrayPU2[index]"></a-input>
      </a-form-item>
    </a-form>
  </div>




</template>

<script>
import { onMounted, ref} from "vue";
import {useRoute} from "vue-router";
import { graphTypeDictionary } from './graphTypeDictionary'
import axios from "axios";

export default {
  name: "DataInput",

  setup() {
    const route = useRoute()
    const graphInfo = ref(JSON.parse(route.params.graphInfo))
    const SL = ref((Number(graphInfo.value.USL) + Number(graphInfo.value.LSL)) / 2)





    let arr1 = []   // 用于记录X-R、X-S、中位数图数据
    for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
      let temp = []
      for (let j = 0; j < Number(graphInfo.value.subgroupCapacity); j++) temp.push(null);
      arr1.push(temp);
    }
    const dataArrayXRXSMedium = ref(arr1)

    let arr2 = []  // 用于记录X-MR图数据
    let arr3 = []  // 用于记录C、nP图数据
    let arr4 = []; let arr5 = [];  // 用于记录P、U图数据  arr4：子组容量 arr5：次品缺陷数量
    for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
      arr2.push(null); arr3.push(null); arr4.push(null); arr5.push(null);
    }
    const dataArrayXMR = ref(arr2); const dataArrayCnP = ref(arr3); const dataArrayPU1 = ref(arr4); const dataArrayPU2 = ref(arr5);






    const graphTypeText = ref();

    for (let i = 0; i < graphTypeDictionary.graphDictionaryValue.length; i++) {
      if (graphTypeDictionary.graphDictionaryValue[i] === graphInfo.value.graphType) graphTypeText.value = graphTypeDictionary.graphDictionaryText[i]
    }



    const handleSubmit = () => {
      if (graphInfo.value.graphType === 'X-R' || graphInfo.value.graphType === 'X-S' || graphInfo.value.graphType === '中位数') console.log(dataArrayXRXSMedium.value)
      if (graphInfo.value.graphType === 'X-MR') console.log(dataArrayXMR.value)
      if (graphInfo.value.graphType === 'C' || graphInfo.value.graphType === 'nP') console.log(dataArrayCnP.value)
      if (graphInfo.value.graphType === 'P' || graphInfo.value.graphType === 'U') console.log(dataArrayPU1.value, dataArrayPU2.value)

      axios.post("/spc/draw", {
        graphType: graphInfo.value.graphType,
        subgroupTotal: graphInfo.value.subgroupTotal,
        subgroupCapacity: graphInfo.value.subgroupCapacity,
        usl: graphInfo.value.USL,
        lsl: graphInfo.value.LSL,
        dataArrayXRXSMedium: dataArrayXRXSMedium.value,
        dataArrayXMR: dataArrayXMR.value,
        dataArrayCnP: dataArrayCnP.value,
        dataArrayPU1: dataArrayPU1.value,
        dataArrayPU2: dataArrayPU2.value,
      }).then((res) => {
        console.log("接收数据成功");
      })

    }

    const handleClear = () => {
      for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
        (dataArrayXMR.value)[i] = null; (dataArrayCnP.value)[i] = null; (dataArrayPU1.value)[i] = null; (dataArrayPU2.value)[i] = null;
        for (let j = 0; j < Number(graphInfo.value.subgroupCapacity); j++) {
          (dataArrayXRXSMedium.value)[i][j] = null;
        }
      }
    }

    onMounted(() => {
      console.log(graphInfo.value)
    })

    return {
      graphInfo,
      graphTypeText,
      SL,

      handleSubmit,
      handleClear,

      dataArrayXRXSMedium,
      dataArrayXMR,
      dataArrayCnP,
      dataArrayPU1,
      dataArrayPU2,
    }
  }
}
</script>
<style scoped>

</style>
