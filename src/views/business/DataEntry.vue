<template>
  <div style="margin:30px; font-size: 15px; text-align: center; text-decoration: underline;">
    请选择直接上传填写完整的数据登入表或者在线录入数据以便后续绘制控制图
    <a-divider/>
  </div>

  <div class="dash-template">
    <div class="dash-box">

          <a-upload
            v-model:file-list="fileList"
            accept = ".xlsx"
            name = "file"
            :multiple = "false"
            @change = "handleChange"
            action = "/spc/read"
          >
            <a-button type="primary" size="large">
              <UploadOutlined/>
              上传数据登入表
            </a-button>
          </a-upload>

      <div class="dash-top"></div>
      <div class="dash-right"></div>
      <div class="dash-bottom"></div>
      <div class="dash-left"></div>
      <div class="dash-top-radius"></div>
      <div class="dash-right-radius"></div>
      <div class="dash-bottom-radius"></div>
      <div class="dash-left-radius"></div>
    </div>




    <div class="dash-box">
      <a-button type="primary" size="large" @click="showModal">在线录入数据</a-button>
      <a-modal
        v-model:visible="visible"
        title="收集表"
        @ok="handleOk"
      >

        <a-form
          v-model="graphInfo"
          :label-col="{ span: 7 }"
          :wrapper-col="{ span: 15 }"
        >
          <br/>   <!-- 调一下间距，不知道有没有更好的方法 -->
          <a-form-item label="控制图类型" name="graphType">
            <a-select v-model:value = "graphInfo.graphType" placeholder="请选择控制图类型">
              <a-select-option v-for="(item, index) in graphTypeDictionary.graphDictionaryText"
                               :key = "item"
                               :value="graphTypeDictionary.graphDictionaryValue[index]"
              >
                {{item}}
              </a-select-option>
            </a-select>
          </a-form-item>

          <a-form-item label="子组总数">
            <a-input v-model:value = "graphInfo.subgroupTotal" placeholder="请输入子组总数（如：100）" type="number"/>
          </a-form-item>

          <a-form-item label="子组容量" v-if="(graphInfo.graphType === 'X-R'|| graphInfo.graphType === 'X-S' || graphInfo.graphType === '中位数' || graphInfo.graphType === 'nP' || graphInfo.graphType === 'C' || graphInfo.graphType === '回归' || graphInfo.graphType === 'T-K' || graphInfo.graphType === '一阶嵌套')" type="number">
            <a-input v-model:value = "graphInfo.subgroupCapacity" placeholder="请输入子组容量（如：50）" type="number"/>
          </a-form-item>

          <a-form-item label="分位数" v-if="(graphInfo.graphType === 'X-R'|| graphInfo.graphType === 'X-S' || graphInfo.graphType === '中位数' || graphInfo.graphType === 'X-MR' || graphInfo.graphType === 'P' || graphInfo.graphType === 'nP' || graphInfo.graphType === 'C' || graphInfo.graphType === 'U')">
            <a-select v-model:value = "graphInfo.quantile" placeholder="请选择是否使用分位数计算控制限">
              <a-select-option value="使用">使用</a-select-option>
              <a-select-option value="不使用">不使用</a-select-option>
            </a-select>
          </a-form-item>

          <a-form-item label="规范上限值（USL）" v-if="(graphInfo.graphType === 'X-R'|| graphInfo.graphType === 'X-S' || graphInfo.graphType === '中位数' || graphInfo.graphType === 'X-MR')">
            <a-input v-model:value = "graphInfo.USL" placeholder="请输入规范上限值USL（如：200.0312）" type="number"/>
          </a-form-item>

          <a-form-item label="规范下限值（LSL）" v-if="(graphInfo.graphType === 'X-R'|| graphInfo.graphType === 'X-S' || graphInfo.graphType === '中位数' || graphInfo.graphType === 'X-MR')">
            <a-input v-model:value = "graphInfo.LSL" placeholder="请输入规范下限值LSL（如：100.0112）" type="number"/>
          </a-form-item>
        </a-form>
      </a-modal>


      <div class="dash-top"></div>
      <div class="dash-right"></div>
      <div class="dash-bottom"></div>
      <div class="dash-left"></div>
      <div class="dash-top-radius"></div>
      <div class="dash-right-radius"></div>
      <div class="dash-bottom-radius"></div>
      <div class="dash-left-radius"></div>
    </div>
  </div>
</template>

<script>
import {defineComponent, onMounted, ref} from 'vue';
import { UploadOutlined } from '@ant-design/icons-vue';
import { graphTypeDictionary } from './graphTypeDictionary'
import { message } from 'ant-design-vue';
import { useRouter } from 'vue-router'

export default defineComponent({
  name: "DataEntry",
  components: {UploadOutlined},

  setup() {
    const router = useRouter()

    const visible = ref(false)

    const graphInfo = ref({})


    const showModal = () => {
      graphInfo.value ={};
      visible.value = true;
    }

    const handleOk = () => {
      visible.value = false;
      console.log(graphInfo.value)

      // 表单填写内容校验
      // 这里暂时就不校验分位数栏有无填写了
      let isLegal = true

      if( !graphInfo.value.graphType || !graphInfo.value.subgroupTotal ) {
        message.error("请填写控制图类型和子组总数！")
        isLegal = false
      } else {
        if (!Number.isInteger( Number(graphInfo.value.subgroupTotal) ) ) {
          message.error("子组总数请填写整数！")
          isLegal = false
        }

        if ( graphInfo.value.graphType === "X-R" || graphInfo.value.graphType === "X-S" || graphInfo.value.graphType === "中位数" || graphInfo.value.graphType === "nP" || graphInfo.value.graphType === "C" || graphInfo.value.graphType === '回归' || graphInfo.value.graphType === 'T-K' || graphInfo.value.graphType === '一阶嵌套') {
          if (!graphInfo.value.subgroupCapacity) {
            message.error("请填写子组容量！")
            isLegal = false

          } else if ( !Number.isInteger( Number(graphInfo.value.subgroupCapacity) ) ) {
            message.error("子组容量请填写整数！")
            isLegal = false
          }
        }

        if ( (graphInfo.value.graphType === "X-R" || graphInfo.value.graphType === "X-S" || graphInfo.value.graphType === "中位数" || graphInfo.value.graphType === "X-MR") && (!graphInfo.value.USL || !graphInfo.value.LSL) ) {
            message.error("请填写规范上限（USL）和规范下限（LSL）！")
            isLegal = false
        }
      }
      //

      // 跳转
      if (isLegal) router.push({name: 'DataInput', params:{ graphInfo: JSON.stringify(graphInfo.value)} })
    }


    // 文件上传
    const handleChange = info => {
      if (info.file.status !== 'uploading') {
        console.log(info.file, info.fileList);
      }

      if (info.file.status === 'done') {
        message.success(`数据登入表${info.file.name}上传成功`);

        if (info.file.response.success) {
          const graphData = ref(info.file.response.result);

          if (graphData.value.graphType === 'X-R')                                         router.push({name: 'GraphXR', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === 'X-S')                                         router.push({name: 'GraphXS', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === '中位数')                                       router.push({name: 'GraphMedium', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === 'X-MR')                                        router.push({name: 'GraphXMR', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === 'P' || graphData.value.graphType === 'U')      router.push({name: 'GraphPU', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === 'P_T' || graphData.value.graphType === 'U_T')  router.push({name: 'GraphPTUT', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === 'C' || graphData.value.graphType === 'nP') router.push({name: 'GraphCnP', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === '回归')  router.push({name: 'GraphRegression', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === 'T-K') router.push({name: 'GraphTK', params:{ graphData: JSON.stringify(graphData.value)} })
          if (graphData.value.graphType === '一阶嵌套') router.push({name: 'GraphFirstOrderNested', params:{ graphData: JSON.stringify(graphData.value)} })
        } else {
          message.error("返回计算及分析结果出错！");
        }
      } else if (info.file.status === 'error') {
        message.error("上传失败，请尝试重新上传");
      }
    };

    const fileList = ref([])

    onMounted(() => {
      console.log(graphTypeDictionary)
    })

    return {
      showModal,
      handleOk,
      visible,
      graphInfo,
      graphTypeDictionary,

      handleChange,
      fileList,
    }
  }
})
</script>

<style scoped>
.dash-template {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.dash-box {
  position: relative;
  width: 50%;
  height: 450px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.dash-left {
  width: 2px;
  position: absolute;
  left: 30px;
  top: 40px;
  bottom: 40px;
  background-image: linear-gradient(to top, #2e6bee 0%, #2e6bee 50%, transparent 50%);
  background-size: 1px 8px;
}

.dash-right {
  width: 2px;
  position: absolute;
  right: 30px;
  top: 40px;
  bottom: 40px;
  background-image: linear-gradient(to bottom, #2e6bee 0%, #2e6bee 50%, transparent 50%);
  background-size: 1px 8px;
}

.dash-top {
  height: 2px;
  position: absolute;
  left: 40px;
  right: 40px;
  top: 30px;
  background-image: linear-gradient(to right, #2e6bee 0%, #2e6bee 50%, transparent 50%);
  background-size: 8px 1px;
}

.dash-bottom {
  height: 2px;
  position: absolute;
  left: 40px;
  right: 40px;
  bottom: 30px;
  background-image: linear-gradient(to left, #2e6bee 0%, #2e6bee 50%, transparent 50%);
  background-size: 8px 1px
}

.dash-top-radius {
  position: absolute;
  left: 30px;
  top: 30px;
  width: 7px;
  height: 7px;
  border-top:2px solid #2e6bee;
  border-left: 2px solid #2e6bee;
  border-top-left-radius: 100%;
}

.dash-right-radius {
  position: absolute;
  right: 30px;
  top: 30px;
  width: 7px;
  height: 7px;
  border-top:2px solid #2e6bee;
  border-right: 2px solid #2e6bee;
  border-top-right-radius: 100%;
}

.dash-bottom-radius {
  position: absolute;
  bottom: 30px;
  right: 30px;
  width: 7px;
  height: 7px;
  border-bottom:2px solid #2e6bee;
  border-right: 2px solid #2e6bee;
  border-bottom-right-radius: 100%;
}

.dash-left-radius {
  position: absolute;
  left: 30px;
  bottom: 30px;
  width: 7px;
  height: 7px;
  border-bottom:2px solid #2e6bee;
  border-left: 2px solid #2e6bee;
  border-bottom-left-radius: 100%;
}
</style>
