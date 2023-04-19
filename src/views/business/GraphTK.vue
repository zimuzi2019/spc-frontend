<template>
  <div id="content">
    <a-divider>数据基本信息</a-divider>

    <a-descriptions bordered style="margin: 5px">
      <a-descriptions-item label="子组容量">{{graphData.subgroupCapacity}}</a-descriptions-item>
      <a-descriptions-item label="总组数">{{graphData.subgroupTotal}}</a-descriptions-item>
      <a-descriptions-item label="总样本数">{{graphData.samplesNum}}</a-descriptions-item>
      <a-descriptions-item label="品种数">{{graphData.sortNum}}</a-descriptions-item>
      <a-descriptions-item label="上限值 UCL (T)">{{graphData.uclT}}</a-descriptions-item>
      <a-descriptions-item label="中心限 CL (T)">{{graphData.clT}}</a-descriptions-item>
      <a-descriptions-item label="下限值 LCL (T)">{{graphData.lclT}}</a-descriptions-item>
      <a-descriptions-item label="上限值 UCL (K)">{{graphData.uclK}}</a-descriptions-item>
      <a-descriptions-item label="中心限 CL (K)">{{graphData.clK}}</a-descriptions-item>
      <a-descriptions-item label="下限值 LCL (K)">{{graphData.lclK}}</a-descriptions-item>
    </a-descriptions>

    <a-divider>控制图分析</a-divider>

    <a-descriptions bordered style="margin: 5px">
      <a-descriptions-item label="T控制图">
        <div class="chart-container">
          <div class="chart" id="spcChartT"></div>
        </div>
      </a-descriptions-item>
    </a-descriptions>

    <a-descriptions  bordered style="margin: 5px">
      <a-descriptions-item label="C区点占比">{{graphData.pointsCRadioT}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点占比" v-if="graphData.specialPointsT.length !== 0">{{graphData.pointsSpecialRadioT}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点编号" v-if="graphData.specialPointsT.length !== 0">{{graphData.specialPointsT.join(' ')}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递增" v-if="graphData.ascendChainTList.length !== 0">{{chainListDisplay(graphData.ascendChainTList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递减" v-if="graphData.descendChainTList.length !== 0">{{chainListDisplay(graphData.descendChainTList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续落在中心线上侧" v-if="graphData.upperChainTList.length !== 0">{{chainListDisplay(graphData.upperChainTList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续落在中心线下侧" v-if="graphData.lowerChainTList.length !== 0">{{chainListDisplay(graphData.lowerChainTList)}}</a-descriptions-item>
    </a-descriptions><br/><br/>

    <a-descriptions bordered style="margin: 5px">
      <a-descriptions-item label="K控制图">
        <div class="chart-container">
          <div class="chart" id="spcChartK"></div>
        </div>
      </a-descriptions-item>
    </a-descriptions>

    <a-descriptions  bordered style="margin: 5px">
      <a-descriptions-item label="C区点占比">{{graphData.pointsCRadioK}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点占比" v-if="graphData.specialPointsK.length !== 0">{{graphData.pointsSpecialRadioK}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点编号" v-if="graphData.specialPointsK.length !== 0">{{graphData.specialPointsK.join(' ')}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递增" v-if="graphData.ascendChainKList.length !== 0">{{chainListDisplay(graphData.ascendChainKList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递减" v-if="graphData.descendChainKList.length !== 0">{{chainListDisplay(graphData.descendChainKList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续落在中心线上侧" v-if="graphData.upperChainKList.length !== 0">{{chainListDisplay(graphData.upperChainKList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续落在中心线下侧" v-if="graphData.lowerChainKList.length !== 0">{{chainListDisplay(graphData.lowerChainKList)}}</a-descriptions-item>
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
import * as echarts  from "echarts"
import jsPDF from "jspdf";
import html2canvas from "html2canvas";

export default {
  name: "GraphTK",

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
      console.log(graphData.value);

      let subgroupTotal = graphData.value.subgroupTotal
      let subgroupCapacity = graphData.value.subgroupCapacity
      let dataArray = graphData.value.dataArray

      const spcChartT = echarts.init(document.getElementById("spcChartT"))
      spcChartT.setOption({
        legend: {
          data: ['T'],
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
          name: '统计量T',
          nameLocation: 'center',
          nameGap: 40,
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },

          // 为了显示UCL和LCL保留
          max: graphData.value.graduationT - (graphData.value.graduationT % 1) + 1,
          min: -(graphData.value.graduationT - (graphData.value.graduationT % 1) + 1),
          interval: 1
        },

        series: [
          {
            name: 'T',
            data: graphData.value.dataArrayT,
            type: 'line',
            itemStyle: {
              color: (params) => {
                if (params.value > graphData.value.uclT) return '#ff6347'
                else if (params.value < graphData.value.lclT) return '#ff00ff'
                else return '#ADD8E6'
              }
            },
            lineStyle: {
              color: '#ADD8E6',
              width: 1,
            },
            markLine: {
              symbol: 'none',
              precision: 3,
              data: [
                {
                  name: 'UCL',
                  yAxis: graphData.value.uclT,
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
                {
                  name: 'CL',
                  yAxis: graphData.value.clT,
                  lineStyle: {
                    color: '#0000ff',
                    width: 2,
                  },
                  label: {
                    color: '#0000ff',
                    fontWeight: 'bold',
                    fontSize: '10',
                    formatter: '{b} {c}'
                  }
                },
                {
                  name: 'LCL',
                  yAxis: graphData.value.lclT,
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
          }
        ]
      })


      const spcChartK = echarts.init(document.getElementById("spcChartK"))
      spcChartK.setOption({
        legend: {
          data: ['K'],
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
          name: '统计量K',
          nameLocation: 'center',
          nameGap: 40,
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },
          max: graphData.value.graduationK - (graphData.value.graduationK % 1) + 1,
          min: -(graphData.value.graduationK - (graphData.value.graduationK % 1) + 1),
          interval: 1
        },

        series: [
          {
            name: 'K',
            data: graphData.value.dataArrayK,
            type: 'line',
            itemStyle: {
              color: (params) => {
                if (params.value > graphData.value.uclK) return '#ff6347'
                else if (params.value < graphData.value.lclK) return '#ff00ff'
                else return '#ADD8E6'
              }
            },
            lineStyle: {
              color: '#ADD8E6',
              width: 1,
            },
            markLine: {
              symbol: 'none',
              precision: 3,
              data: [
                {
                  name: 'UCL',
                  yAxis: graphData.value.uclK,
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
                {
                  name: 'CL',
                  yAxis: graphData.value.clK,
                  lineStyle: {
                    color: '#0000ff',
                    width: 2,
                  },
                  label: {
                    color: '#0000ff',
                    fontWeight: 'bold',
                    fontSize: '10',
                    formatter: '{b} {c}'
                  }
                },
                {
                  name: 'LCL',
                  yAxis: graphData.value.lclK,
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
          }
        ]
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
