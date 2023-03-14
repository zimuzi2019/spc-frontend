<template>
  <div id="content">
    <a-divider>数据基本信息</a-divider>

    <a-descriptions bordered style="margin: 5px">
      <a-descriptions-item label="子组数">{{graphData.subgroupTotal}}</a-descriptions-item>
      <a-descriptions-item label="平均子组容量">{{graphData.avgSubgroupCapacity}}</a-descriptions-item>
      <a-descriptions-item label="最大子组容量">{{graphData.subgroupCapacityMax}}</a-descriptions-item>
      <a-descriptions-item label="最小子组容量">{{graphData.subgroupCapacityMin}}</a-descriptions-item>
      <a-descriptions-item label="总样本数">{{graphData.samplesNum}}</a-descriptions-item>
      <a-descriptions-item label="总不良品数">{{graphData.defectsNum}}</a-descriptions-item>
      <a-descriptions-item label="平均不良率">{{graphData.avgDefectsNum}}</a-descriptions-item>
      <a-descriptions-item label="上限值 UCL">{{graphData.ucl}}</a-descriptions-item>
      <a-descriptions-item label="下限值 LCL">{{graphData.lcl}}</a-descriptions-item>
    </a-descriptions>

    <br/>

    <a-divider>控制图分析</a-divider>

    <a-descriptions bordered style="margin: 5px">
      <a-descriptions-item :label="graphData.graphType + '控制图'">
        <div class="chart-container">
          <div class="chart" id="spcChart"></div>
        </div>
      </a-descriptions-item>
    </a-descriptions>

    <a-descriptions  bordered style="margin: 5px">
      <a-descriptions-item label="C区点占比">{{graphData.pointsCRadio}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点占比" v-if="graphData.specialPoints.length !== 0">{{graphData.pointsSpecialRadio}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点编号" v-if="graphData.specialPoints.length !== 0">{{graphData.specialPoints.join(' ')}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递增" v-if="graphData.ascendChainList.length !== 0">{{chainListDisplay(graphData.ascendChainList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递减" v-if="graphData.descendChainList.length !== 0">{{chainListDisplay(graphData.descendChainList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续落在中心线上侧" v-if="graphData.upperChainList.length !== 0">{{chainListDisplay(graphData.upperChainList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续落在中心线下侧" v-if="graphData.lowerChainList.length !== 0">{{chainListDisplay(graphData.lowerChainList)}}</a-descriptions-item>
    </a-descriptions>
  </div>

  <a-divider/>
  <p style="margin-left: 80%">
    <a-form layout="inline">
      <a-form-item>
        <a-button type="primary" @click="savePDF">导出PDF</a-button>
      </a-form-item>
      <a-form-item>
        <a-button type="primary" @click="savePNG">导出PNG</a-button>
      </a-form-item>
    </a-form>
  </p>
</template>

<script>
import {useRoute} from "vue-router";
import {onMounted, ref} from "vue";
import * as echarts from "echarts";
import jsPDF from 'jspdf'
import html2canvas from 'html2canvas'

export default {
  name: "GraphPU",

  setup() {
    const route = useRoute()
    const graphData = ref(JSON.parse(route.params.graphData))

    const chainListDisplay = function(chainList){
      return chainList.map(function(item) {
        return item.join('-')
      }).join(' ')
    }

    const savePNG = function() {
      const pdf = new jsPDF();

      html2canvas(document.querySelector("#content")).then(canvas => {
        const link = document.createElement('a');
        link.download = 'report.png';
        link.href = canvas.toDataURL();
        link.click();
      });
    }

    const savePDF = function() {
      const pdf = new jsPDF();

      html2canvas(document.querySelector("#content")).then(canvas => {
        const imgData = canvas.toDataURL("image/png");
        pdf.addImage(imgData, 'PNG', 0, 0, 210, 297);
        pdf.save("report.pdf");
      });
    }

    onMounted(() => {
      console.log(graphData.value)

      const spcChart = echarts.init(document.getElementById('spcChart'))
      spcChart.setOption({
        legend: {
          data: [ '标准化单位不良品率' ],
          orient: 'horizontal',
          left: 'center',
        },

        tooltip: {
          trigger: 'axis',
        },

        xAxis: {
          type: 'category',
          name: '子组编号',
          nameLocation: 'center',
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },
          nameGap: 40,
          data: Array.from({length: graphData.value.subgroupTotal}, (_, i) => i + 1),
        },

        yAxis: {
          type: 'value',
          min: -graphData.value.graduation,
          max: graphData.value.graduation,
          interval: 2,
          name: '标准化单位不良品率',
          nameLocation: 'center',
          nameGap: 40,
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },
        },

        series: [
          {
            name: '上限值UCL',
            data: [],
            type: 'line',
            itemStyle: {
              color: '#ff6347'
            },
            markLine: {
              symbol: 'none',
              precision: 3,
              data: [
                {
                  name: 'UCL',
                  yAxis: graphData.value.ucl,
                  lineStyle: {
                    color: '#ff6347',
                    width: 2,
                  },
                  label: {
                    color: '#ff6347',
                    fontWeight: 'bold',
                    fontSize: '10',
                    formatter: '{b} {c}'
                  }
                },
              ]
            }
          },
          {
            name: '下限值LCL',
            data: [],
            type: 'line',
            itemStyle: {
              color: '#ff00ff'
            },
            markLine: {
              symbol: 'none',
              precision: 3,
              data: [
                {
                  name: 'LCL',
                  yAxis: graphData.value.lcl,
                  lineStyle: {
                    color: '#ff00ff',
                    width: 2,
                  },
                  label: {
                    color: '#ff00ff',
                    fontWeight: 'bold',
                    fontSize: '10',
                    formatter: '{b} {c}'
                  }
                },
              ]
            }
          },
          {
            name: '中心限CL',
            data: [],
            type: 'line',
            itemStyle: {
              color: '#0000ff'
            },
            markLine: {
              symbol: 'none',
              precision: 3,
              data: [
                {
                  name: 'CL',
                  yAxis: graphData.value.cl,
                  lineStyle: {
                    color: '#0000ff',
                    width: 2,
                  },
                  label: {
                    color: '#0000ff',
                    fontWeight: 'bold',
                    fontSize: '10',
                    formatter: '{b}  {c}'
                  }
                },
              ]
            }
          },
          {
            name: '标准化单位不良品率',
            data: graphData.value.dataArray,
            type: 'line',
            itemStyle: {
              color: (params) => {
                if (params.value > graphData.value.ucl) return '#ff6347'
                else if (params.value < graphData.value.lcl) return '#ff00ff'
                else return '#ADD8E6'
              }
            },
            lineStyle: {
              color: '#ADD8E6',
              width: 1,
            },
          }
        ],
      })
    })

    return {
      graphData,
      chainListDisplay,
      savePDF,
      savePNG
    }
  }
}
</script>

<style scoped>
.chart-container {
  display: flex;
  justify-content: center;
}

.chart {
  width: 1000px;
  height: 500px;
}
</style>
