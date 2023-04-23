<template>

  <p style="margin: 15px; text-align: center; font-size: large">
    <strong>控制图基本信息</strong>
  </p>

  <a-descriptions bordered column="2">
    <a-descriptions-item label="控制图类型">{{ graphTypeText }}</a-descriptions-item>

    <a-descriptions-item label="批次总数" v-if="graphInfo.graphType === '二阶嵌套'">
      {{ graphInfo.batchNum }}
    </a-descriptions-item>

    <a-descriptions-item label="子组总数">{{ graphInfo.subgroupTotal }}</a-descriptions-item>

    <!-- 综合控制图只做了”嵌套-回归“控制图作为demo -->
    <a-descriptions-item label="子组容量" v-if="graphInfo.graphType === 'X-R' || graphInfo.graphType === 'X-S' || graphInfo.graphType === '中位数' || graphInfo.graphType === 'nP' || graphInfo.graphType === 'C' || graphInfo.grapohType === '回归' || graphInfo.grapohType === 'T-K' || graphInfo.graphType === '一阶嵌套' || graphInfo.graphType === '综合' || graphInfo.graphType === '二阶嵌套' || graphInfo.graphType === '多变量T^2'">
      {{ graphInfo.subgroupCapacity }}
    </a-descriptions-item>

    <a-descriptions-item label="变量个数" v-if="graphInfo.graphType === '单值多变量T^2' || graphInfo.graphType === '多变量T^2'">
      {{ graphInfo.varNum }}
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

  <!-- 回归、T-K控制图 -->
  <div v-for="(row, index1) in dataArrayRegressionTK" :key="index1" v-if="graphInfo.graphType ==='回归' || graphInfo.graphType === 'T-K'">
    <a-divider>子组编号{{ index1+1 }}</a-divider>
    <a-form>
      <a-form-item label="目标值" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入目标值，如：50.00" v-model:value="dataArrayRegressionTKStandard[index1]"></a-input>
      </a-form-item>
      <a-form-item label="精度" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input placeholder="请填入精度，如：0.10%" v-model:value="dataArrayRegressionTKPrecision[index1]"></a-input>
      </a-form-item>
      <a-form-item label="品种编号" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入品种编号，如：2" v-model:value="dataArrayRegressionTKSort[index1]"></a-input>
      </a-form-item>

      <br/>

      <a-form-item v-for="(item, index2) in row" :label="`样品编号${String(index2+1)}`" :key="index2" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArrayRegressionTK[index1][index2]"></a-input>
      </a-form-item>
    </a-form>
  </div>

  <!-- 一阶嵌套控制图数据 -->
  <div v-for="(row, index1) in dataArrayFirstOrderNested" :key="index1" v-if="graphInfo.graphType === '一阶嵌套'">
    <a-divider>子组编号{{ index1+1 }}</a-divider>
    <a-form>
      <a-form-item v-for="(item, index2) in row" :label="`样品编号${String(index2+1)}`" :key="index2" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArrayFirstOrderNested[index1][index2]"></a-input>
      </a-form-item>
    </a-form>
  </div>

  <!-- 单值多变量T^2控制图数据 -->
  <div v-for="(row, index1) in dataArrayT2Single" :key="index1" v-if="graphInfo.graphType === '单值多变量T^2'">
    <a-divider>子组编号{{ index1+1 }}</a-divider>
    <a-form>
      <a-form-item v-for="(item, index2) in row" :label="`变量编号${String(index2+1)}`" :key="index2" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArrayT2Single[index1][index2]"></a-input>
      </a-form-item>
    </a-form>
  </div>

  <!-- 综合控制图只做了”嵌套-回归“控制图作为demo -->
  <div v-for="(row, index1) in dataArrayIntegratedDemo" :key="index1" v-if="graphInfo.graphType ==='综合'">
    <a-divider>子组编号{{ index1+1 }}</a-divider>
    <a-form>
      <a-form-item label="目标值" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入目标值，如：50.00" v-model:value="dataArrayIntegratedDemoStandard[index1]"></a-input>
      </a-form-item>
      <!--
      <a-form-item label="精度" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input placeholder="请填入精度，如：0.10%" v-model:value="dataArrayRegressionTKPrecision[index1]"></a-input>
      </a-form-item>

      <a-form-item label="品种编号" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入品种编号，如：2" v-model:value="dataArrayRegressionTKSort[index1]"></a-input>
      </a-form-item>
      -->
      <br/>

      <a-form-item v-for="(item, index2) in row" :label="`样品编号${String(index2+1)}`" :key="index2" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
        <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArrayIntegratedDemo[index1][index2]"></a-input>
      </a-form-item>
    </a-form>
  </div>

  <!-- 二阶嵌套控制图数据 -->
  <div v-for="(batch, index1) in dataArraySecondOrderNested" :key="index1" v-if="graphInfo.graphType === '二阶嵌套'">
    <a-divider>批次编号{{ index1+1 }}</a-divider>
    <div v-for="(row, index2) in batch" :key="index2">
      <a-divider>子组编号{{ index2+1 }}</a-divider>
      <a-form>
        <a-form-item v-for="(item, index3) in row" :label="`样本编号${String(index3+1)}`" :key="index3" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
          <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArraySecondOrderNested[index1][index2][index3]"></a-input>
        </a-form-item>
      </a-form>
    </div>
  </div>

  <!-- 多变量T^2控制图数据 -->
  <div v-for="(batch, index1) in dataArrayT2" :key="index1" v-if="graphInfo.graphType === '多变量T^2'">
    <a-divider>子组编号{{ index1+1 }}</a-divider>
    <div v-for="(row, index2) in batch" :key="index2">
      <a-divider>变量编号{{ index2+1 }}</a-divider>
      <a-form>
        <a-form-item v-for="(item, index3) in row" :label="`样本编号${String(index3+1)}`" :key="index3" :label-col="{ span: 3 }" :wrapper-col="{ span: 19 }">
          <a-input type="number" placeholder="请填入实际测量值，如：50.12" v-model:value="dataArrayT2[index1][index2][index3]"></a-input>
        </a-form-item>
      </a-form>
    </div>
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
    const dataArrayRegressionTK = ref(arr6); const dataArrayRegressionTKStandard = ref(arr7); const dataArrayRegressionTKPrecision = ref(arr8); const dataArrayRegressionTKSort = ref(arr9);

    let arr10 = []   // 用于记录一阶嵌套控制图数据
    for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
      let temp = []
      for (let j = 0; j < Number(graphInfo.value.subgroupCapacity); j++) temp.push(null);
      arr10.push(temp);
    }
    const dataArrayFirstOrderNested = ref(arr10)

    let arr11 = []   // 用于记录单值多变量T^2控制图数据
    for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
      let temp = []
      for (let j = 0; j < Number(graphInfo.value.varNum); j++) temp.push(null);
      arr11.push(temp);
    }
    const dataArrayT2Single = ref(arr11)


    // ---------------- 综合控制图只做了”嵌套-回归“控制图作为demo -----------------------------------------
    let arr12 = [];   let arr13 = [];  // 用于记录综合控制图数据 arr12：测量数据 arr13：目标值
    for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
      let temp = []
      for (let j = 0; j < Number(graphInfo.value.subgroupCapacity); j++) temp.push(null);
      arr12.push(temp);
      arr13.push(null);
    }
    const dataArrayIntegratedDemo = ref(arr12); const dataArrayIntegratedDemoStandard = ref(arr13);
    // -----------------------------------------------------------------------------------------------


    let arr14 = []   // 用于记录二阶嵌套控制图数据
    for (let i = 0; i < Number(graphInfo.value.batchNum); i++) {
      let temp1 = []
      for (let j = 0; j < Number(graphInfo.value.subgroupTotal); j++) {
        let temp2 = []
        for (let k = 0; k < Number(graphInfo.value.subgroupCapacity); k++) {
          temp2.push(null);
        }
        temp1.push(temp2)
      }
      arr14.push(temp1);
    }
    const dataArraySecondOrderNested= ref(arr14)

    let arr15 = []   // 用于记录多变量T^2控制图数据
    for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
      let temp1 = []
      for (let j = 0; j < Number(graphInfo.value.varNum); j++) {
        let temp2 = []
        for (let k = 0; k < Number(graphInfo.value.subgroupCapacity); k++) {
          temp2.push(null);
        }
        temp1.push(temp2)
      }
      arr15.push(temp1);
    }
    const dataArrayT2= ref(arr15)



    const graphTypeText = ref();

    for (let i = 0; i < graphTypeDictionary.graphDictionaryValue.length; i++) {
      if (graphTypeDictionary.graphDictionaryValue[i] === graphInfo.value.graphType) graphTypeText.value = graphTypeDictionary.graphDictionaryText[i]
    }

    const handleSubmit = () => {
      console.log(dataArrayT2.value)
      axios.post("/spc/draw", {
        graphType: graphInfo.value.graphType,
        batchNum: graphInfo.value.batchNum,
        subgroupTotal: graphInfo.value.subgroupTotal,
        subgroupCapacity: graphInfo.value.subgroupCapacity,
        usl: graphInfo.value.USL,
        lsl: graphInfo.value.LSL,
        quantile: graphInfo.value.quantile,
        varNum: graphInfo.value.varNum,
        sl: SL.value,
        dataArrayXRXSMedium: dataArrayXRXSMedium.value,
        dataArrayXMR: dataArrayXMR.value,
        dataArrayCnP: dataArrayCnP.value,
        dataArrayPUPTUTSubgroupsCapacity: dataArrayPUPTUTSubgroupsCapacity.value,
        dataArrayPUPTUTDefectsNum: dataArrayPUPTUTDefectsNum.value,

        dataArrayRegressionTK: dataArrayRegressionTK.value,
        dataArrayRegressionTKStandard: dataArrayRegressionTKStandard.value,
        dataArrayRegressionTKPrecision: dataArrayRegressionTKPrecision.value,
        dataArrayRegressionTKSort: dataArrayRegressionTKSort.value,

        dataArrayFirstOrderNested: dataArrayFirstOrderNested.value,
        dataArraySecondOrderNested: dataArraySecondOrderNested.value,
        dataArrayT2Single: dataArrayT2Single.value,
        dataArrayT2: dataArrayT2.value,

        // ---------------- 综合控制图只做了”嵌套-回归“控制图作为demo ---------------
        dataArrayIntegratedDemo: dataArrayIntegratedDemo.value,
        dataArrayIntegratedDemoStandard: dataArrayIntegratedDemoStandard.value,
        // ---------------------------------------------------------------------
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
          if (graphData.value.graphType === 'T-K')                                        router.push({name: 'GraphTK', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === '一阶嵌套') router.push({name: 'GraphFirstOrderNested', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === '二阶嵌套') router.push({name: 'GraphSecondOrderNested', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === '多变量T^2') router.push({name: 'GraphT2', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === '单值多变量T^2') router.push({name: 'GraphT2Single', params:{ graphData: JSON.stringify(graphData.value)} })


          // ---- 综合控制图只做了”嵌套-回归“控制图作为demo ---
          if (graphData.value.graphType === '综合') router.push({name: 'GraphIntegratedDemo', params:{ graphData: JSON.stringify(graphData.value)} })
        } else {
          message.error("返回计算及分析结果出错！");
        }
      })

    }

    const handleClear = () => {
      // 清空二阶嵌套控制图数据
      for (let i = 0; i < Number(graphInfo.value.batchNum); i++) {
        for (let j = 0; j < Number(graphInfo.value.subgroupTotal); j++) {
          for (let k = 0; k < Number(graphInfo.value.subgroupCapacity); k++) {
            (dataArraySecondOrderNested.value)[i][j][k] = null;
          }
        }
      }

      // 清空多变量T^2控制图数据
      for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
        for (let j = 0; j < Number(graphInfo.value.varNum); j++) {
          for (let k = 0; k < Number(graphInfo.value.subgroupCapacity); k++) {
            (dataArrayT2.value)[i][j][k] = null;
          }
        }
      }

      for (let i = 0; i < Number(graphInfo.value.subgroupTotal); i++) {
        (dataArrayXMR.value)[i] = null; (dataArrayCnP.value)[i] = null; (dataArrayPUPTUTSubgroupsCapacity.value)[i] = null; (dataArrayPUPTUTDefectsNum.value)[i] = null;
        (dataArrayRegressionTKStandard.value)[i] = null; (dataArrayRegressionTKPrecision.value)[i] = null; (dataArrayRegressionTKSort.value)[i] = null;

        // ---- 综合控制图只做了”嵌套-回归“控制图作为demo ---
        (dataArrayIntegratedDemoStandard.value)[i] = null;
        for (let j = 0; j < Number(graphInfo.value.subgroupCapacity); j++) {
          (dataArrayXRXSMedium.value)[i][j] = null; (dataArrayRegressionTK.value)[i][j] = null; (dataArrayFirstOrderNested.value)[i][j] = null;

          // ---- 综合控制图只做了”嵌套-回归“控制图作为demo ---
          (dataArrayIntegratedDemo.value)[i][j] = null;
        }

        for (let j = 0 ; j < Number(graphInfo.value.varNum); j++) {
          (dataArrayT2Single.value)[i][j] = null;
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

      dataArrayRegressionTK,
      dataArrayRegressionTKStandard,
      dataArrayRegressionTKPrecision,
      dataArrayRegressionTKSort,

      dataArrayFirstOrderNested,
      dataArraySecondOrderNested,

      dataArrayT2Single,
      dataArrayT2,

      // ---- 综合控制图只做了”嵌套-回归“控制图作为demo ---
      dataArrayIntegratedDemo,
      dataArrayIntegratedDemoStandard,
    }
  }
}
</script>
<style scoped>

</style>
