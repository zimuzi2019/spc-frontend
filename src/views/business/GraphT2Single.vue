<template>
  <div id="content">
    <a-divider>数据基本信息</a-divider>

    <a-descriptions bordered style="margin: 5px">
      <a-descriptions-item label="总组数">{{graphData.subgroupTotal}}</a-descriptions-item>
      <a-descriptions-item label="变量个数">{{graphData.varNum}}</a-descriptions-item>
      <a-descriptions-item label="上限值 UCL (分析用)">{{graphData.uclT2Analysis}}</a-descriptions-item>
      <a-descriptions-item label="下限值 LCL (分析用)">{{graphData.lclT2Analysis}}</a-descriptions-item>
      <a-descriptions-item label="上限值 UCL (控制用)">{{graphData.uclT2Control}}</a-descriptions-item>
      <a-descriptions-item label="下限值 LCL (控制用)">{{graphData.lclT2Control}}</a-descriptions-item>
    </a-descriptions>

    <a-divider>控制图分析</a-divider>

    <a-descriptions bordered style="margin: 5px">
      <a-descriptions-item label="分析用T^2控制图">
        <div class="chart-container">
          <div class="chart" id="spcChartT2Analysis"></div>
        </div>
      </a-descriptions-item>
    </a-descriptions>

    <a-descriptions  bordered style="margin: 5px">
      <a-descriptions-item label="超出控制限点占比" v-if="graphData.specialPointsT2Analysis.length !== 0">{{graphData.pointsSpecialRadioT2Analysis}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点编号" v-if="graphData.specialPointsT2Analysis.length !== 0">{{graphData.specialPointsT2Analysis.join(' ')}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递增" v-if="graphData.ascendChainT2AnalysisList.length !== 0">{{chainListDisplay(graphData.ascendChainT2AnalysisList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递减" v-if="graphData.descendChainT2AnalysisList.length !== 0">{{chainListDisplay(graphData.descendChainT2AnalysisList)}}</a-descriptions-item>
    </a-descriptions><br/><br/>

    <a-descriptions bordered style="margin: 5px">
      <a-descriptions-item label="控制用T^2控制图">
        <div class="chart-container">
          <div class="chart" id="spcChartT2Control"></div>
        </div>
      </a-descriptions-item>
    </a-descriptions>

    <a-descriptions  bordered style="margin: 5px">
      <a-descriptions-item label="超出控制限点占比" v-if="graphData.specialPointsT2Control.length !== 0">{{graphData.pointsSpecialRadioT2Control}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点编号" v-if="graphData.specialPointsT2Control.length !== 0">{{graphData.specialPointsT2Control.join(' ')}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递增" v-if="graphData.ascendChainT2ControlList.length !== 0">{{chainListDisplay(graphData.ascendChainT2ControlList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递减" v-if="graphData.descendChainT2ControlList.length !== 0">{{chainListDisplay(graphData.descendChainT2ControlList)}}</a-descriptions-item>
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
  name: "GraphT2Single",

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

      const spcChartT2Analysis = echarts.init(document.getElementById("spcChartT2Analysis"))
      spcChartT2Analysis.setOption({
        legend: {
          data: ['T^2'],
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
          name: '统计量T^2',
          nameLocation: 'center',
          nameGap: 40,
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },

          // 为了显示UCL和LCL保留
          max: graphData.value.graduationT2 - (graphData.value.graduationT2 % 1) + 1,
          min: 0,
          interval: 1
        },

        series: [
          {
            name: 'T^2',
            data: graphData.value.dataArrayT2,
            type: 'line',
            itemStyle: {
              color: (params) => {
                if (params.value > graphData.value.uclT2Analysis) return '#ff6347'
                else if (params.value < graphData.value.lclT2Analysis) return '#ff00ff'
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
                  yAxis: graphData.value.uclT2Analysis,
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
                  name: 'LCL',
                  yAxis: graphData.value.lclT2Analysis,
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


      const spcChartT2Control = echarts.init(document.getElementById("spcChartT2Control"))
      spcChartT2Control.setOption({
        legend: {
          data: ['T^2'],
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
          name: '统计量T^2',
          nameLocation: 'center',
          nameGap: 40,
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },

          max: graphData.value.graduationT2 - (graphData.value.graduationT2 % 1) + 1,
          min: 0,
          interval: 1
        },

        series: [
          {
            name: 'T^2',
            data: graphData.value.dataArrayT2,
            type: 'line',
            itemStyle: {
              color: (params) => {
                if (params.value > graphData.value.uclT2Control) return '#ff6347'
                else if (params.value < graphData.value.lclT2Control) return '#ff00ff'
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
                  yAxis: graphData.value.uclT2Control,
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
                  name: 'LCL',
                  yAxis: graphData.value.lclT2Control,
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
