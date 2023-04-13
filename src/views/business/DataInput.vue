<template>

  <p style="margin: 15px; text-align: center; font-size: large">
    <strong>控制图基本信息</strong>
  </p>

  <a-descriptions bordered column="2">
    <a-descriptions-item label="控制图类型">{{ graphTypeText }}</a-descriptions-item>
    <a-descriptions-item label="子组总数">{{ graphInfo.subgroupTotal }}</a-descriptions-item>

    <a-descriptions-item label="子组容量" v-if="graphInfo.graphType === 'X-R' || graphInfo.graphType === 'X-S' || graphInfo.graphType === '中位数' || graphInfo.graphType === 'nP' || graphInfo.graphType === 'C' || graphInfo.grapohType === '回归'">
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

    <a-descriptions-item label="分位数" v-if="(graphInfo.graphType === 'X-R'|| graphInfo.graphType === 'X-S' || graphInfo.graphType === '中位数' || graphInfo.graphType === 'X-MR' || graphInfo.graphType === 'P' || graphInfo.graphType === 'nP' || graphInfo.graphType === 'C' || graphInfo.graphType === 'U')">
      {{ graphInfo.quantile }}
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




  <!-- 根据控制图类型决定加载哪种形式的输入 -->
  <!-- X-R图、X-S图、中位数图 -->
  <div v-for="(row, index1) in dataArrayXRXSMedium" :key="index1" v-if="graphInfo.graphType === 'X-R' || graphInfo.graphType === 'X-S' || graphInfo.graphType ==='中位数'">
    <a-divider>子组编号{{ index1+1 }}</a-divider>
    <a-form>
      <a-form-item v-for="(item, index2) in row" :label="`样品编号${String(index2+1)}`" :key="index2" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArrayXRXSMedium[index1][index2]"></a-input>
      </a-form-item>
    </a-form>
  </div>

  <!-- X-MR图 -->
  <div v-if="graphInfo.graphType === 'X-MR'">
    <a-divider/>
    <a-form>
      <a-form-item v-for="(item, index) in dataArrayXMR" :label="`样本测量值${String(index+1)}`" :key="index" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArrayXMR[index]"></a-input>
      </a-form-item>
    </a-form>
  </div>

  <!-- C图、nP图 -->
  <div v-if="graphInfo.graphType === 'C' || graphInfo.graphType === 'nP'">
    <a-divider/>
    <a-form>
      <a-form-item v-for="(item, index) in dataArrayXMR" :label="`子组编号${String(index+1)}次品/缺陷数`" :key="index" :label-col="{ span: 4 }" :wrapper-col="{ span: 18 }">
        <a-input type="number" placeholder="请填入实际测量值，如：2" v-model:value="dataArrayCnP[index]"></a-input>
      </a-form-item>
    </a-form>
  </div>

  <!-- P图、U图、P_T图、U_T图 -->
  <div v-if="graphInfo.graphType === 'P' || graphInfo.graphType === 'U' || graphInfo.graphType === 'P_T' || graphInfo.graphType === 'U_T'">
    <a-divider/>
    <a-form>
      <a-form-item v-for="(item, index) in dataArrayPUPTUTSubgroupsCapacity" :label="`子组编号${String(index+1)}`" :key="index" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入该子组所含样本容量，如：50" v-model:value="dataArrayPUPTUTSubgroupsCapacity[index]"></a-input>
        <a-input type="number" placeholder="请填入该子组所含次品/缺陷数量，如：3" v-model:value="dataArrayPUPTUTDefectsNum[index]"></a-input>
      </a-form-item>
    </a-form>
  </div>

  <!-- 回归控制图 -->
  <div v-for="(row, index1) in dataArrayRegression" :key="index1" v-if="graphInfo.graphType ==='回归'">
    <a-divider>子组编号{{ index1+1 }}</a-divider>
    <a-form>
      <a-form-item label="目标值" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入目标值，如：50.00" v-model:value="dataArrayRegressionStandard[index1]"></a-input>
      </a-form-item>
      <a-form-item label="精度" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input placeholder="请填入精度，如：0.10%" v-model:value="dataArrayRegressionPrecision[index1]"></a-input>
      </a-form-item>
      <a-form-item label="品种编号" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入品种编号，如：2" v-model:value="dataArrayRegressionSort[index1]"></a-input>
      </a-form-item>

      <br/>

      <a-form-item v-for="(item, index2) in row" :label="`样品编号${String(index2+1)}`" :key="index2" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArrayRegression[index1][index2]"></a-input>
      </a-form-item>
    </a-form>
  </div>
</template>

<script>
import { ref} from "vue";
import {useRoute, useRouter} from "vue-router";
import { graphTypeDictionary } from './graphTypeDictionary'
import axios from "axios";
import {message} from "ant-design-vue";

export default {
  name: "DataInput",

  setup() {
    const route = useRoute()
    const router = useRouter();

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
    let arr4 = []; let arr5 = [];  // 用于记录P、U、P_T、U_T图数据  arr4：子组容量 arr5：次品缺陷数量
    for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
      arr2.push(null); arr3.push(null); arr4.push(null); arr5.push(null);
    }
    const dataArrayXMR = ref(arr2); const dataArrayCnP = ref(arr3); const dataArrayPUPTUTSubgroupsCapacity = ref(arr4); const dataArrayPUPTUTDefectsNum = ref(arr5);


    let arr6 = [];   let arr7 = [];  let arr8 = []; let arr9 = [];// 用于记录回归控制图数据 arr6：测量数据 arr7：目标值  arr8：精度 arr9：品种编号
    for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
      let temp = []
      for (let j = 0; j < Number(graphInfo.value.subgroupCapacity); j++) temp.push(null);
      arr6.push(temp);

      arr7.push(null); arr8.push(null); arr9.push(null);
    }
    const dataArrayRegression = ref(arr6); const dataArrayRegressionStandard = ref(arr7); const dataArrayRegressionPrecision = ref(arr8); const dataArrayRegressionSort = ref(arr9);



    const graphTypeText = ref();

    for (let i = 0; i < graphTypeDictionary.graphDictionaryValue.length; i++) {
      if (graphTypeDictionary.graphDictionaryValue[i] === graphInfo.value.graphType) graphTypeText.value = graphTypeDictionary.graphDictionaryText[i]
    }



    const handleSubmit = () => {
      console.log(dataArrayRegressionPrecision.value)
      axios.post("/spc/draw", {
        graphType: graphInfo.value.graphType,
        subgroupTotal: graphInfo.value.subgroupTotal,
        subgroupCapacity: graphInfo.value.subgroupCapacity,
        usl: graphInfo.value.USL,
        lsl: graphInfo.value.LSL,
        quantile: graphInfo.value.quantile,
        sl: SL.value,
        dataArrayXRXSMedium: dataArrayXRXSMedium.value,
        dataArrayXMR: dataArrayXMR.value,
        dataArrayCnP: dataArrayCnP.value,
        dataArrayPUPTUTSubgroupsCapacity: dataArrayPUPTUTSubgroupsCapacity.value,
        dataArrayPUPTUTDefectsNum: dataArrayPUPTUTDefectsNum.value,

        dataArrayRegression: dataArrayRegression.value,
        dataArrayRegressionStandard: dataArrayRegressionStandard.value,
        dataArrayRegressionPrecision: dataArrayRegressionPrecision.value,
        dataArrayRegressionSort: dataArrayRegressionSort.value,
      }).then((response) => {
        const data = response.data;
        if (data.success) {
          const graphData = ref(data.result);

          if (graphData.value.graphType === 'X-R')                                        router.push({name: 'GraphXR', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === 'X-S')                                        router.push({name: 'GraphXS', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === '中位数')                                      router.push({name: 'GraphMedium', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === 'X-MR')                                       router.push({name: 'GraphXMR', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === 'P' || graphData.value.graphType === 'U')     router.push({name: 'GraphPU', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === 'P_T' || graphData.value.graphType === 'U_T') router.push({name: 'GraphPTUT', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === 'C' || graphData.value.graphType === 'nP')    router.push({name: 'GraphCnP', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === '回归')                                        router.push({name: 'GraphRegression', params:{ graphData: JSON.stringify(graphData.value)} })
        } else {
          message.error("返回计算及分析结果出错！");
        }
      })

    }

    const handleClear = () => {
      for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
        (dataArrayXMR.value)[i] = null; (dataArrayCnP.value)[i] = null; (dataArrayPUPTUTSubgroupsCapacity.value)[i] = null; (dataArrayPUPTUTDefectsNum.value)[i] = null;
        (dataArrayRegressionStandard.value)[i] = null; (dataArrayRegressionPrecision.value)[i] = null; (dataArrayRegressionSort.value)[i] = null;

        for (let j = 0; j < Number(graphInfo.value.subgroupCapacity); j++) {
          (dataArrayXRXSMedium.value)[i][j] = null; (dataArrayRegression.value)[i][j] = null;
        }
      }
    }

    return {
      graphInfo,
      graphTypeText,
      SL,

      handleSubmit,
      handleClear,

      dataArrayXRXSMedium,
      dataArrayXMR,
      dataArrayCnP,
      dataArrayPUPTUTSubgroupsCapacity,
      dataArrayPUPTUTDefectsNum,

      dataArrayRegression,
      dataArrayRegressionStandard,
      dataArrayRegressionPrecision,
      dataArrayRegressionSort,
    }
  }
}
</script>
<style scoped>

</style>
