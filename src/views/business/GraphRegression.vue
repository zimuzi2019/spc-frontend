<template>
  <div id="content">
    <a-divider>数据基本信息</a-divider>

    <a-descriptions bordered style="margin: 5px">
      <a-descriptions-item label="子组容量">{{graphData.subgroupCapacity}}</a-descriptions-item>
      <a-descriptions-item label="总组数">{{graphData.subgroupTotal}}</a-descriptions-item>
      <a-descriptions-item label="总样本数">{{graphData.samplesNum}}</a-descriptions-item>
      <a-descriptions-item label="精度种数">{{graphData.precisionNum}}</a-descriptions-item>
      <a-descriptions-item label="品种数">{{graphData.sortNum}}</a-descriptions-item>
      <a-descriptions-item label="上限值 UCL (X)">{{graphData.uclXBar}}</a-descriptions-item>
      <a-descriptions-item label="中心限 CL (X)">{{graphData.clXBar}}</a-descriptions-item>
      <a-descriptions-item label="下限值 LCL (X)">{{graphData.lclXBar}}</a-descriptions-item>
      <a-descriptions-item label="上限值 UCL (S)">{{graphData.uclS}}</a-descriptions-item>
      <a-descriptions-item label="中心限 CL (S)">{{graphData.clS}}</a-descriptions-item>
      <a-descriptions-item label="下限值 LCL (S)">{{graphData.lclS}}</a-descriptions-item>
    </a-descriptions>

    <a-divider>控制图分析</a-divider>

    <a-descriptions bordered style="margin: 5px">
      <a-descriptions-item label="X控制图">
        <div class="chart-container">
          <div class="chart" id="spcChartXBar"></div>
        </div>
      </a-descriptions-item>
    </a-descriptions>

    <a-descriptions  bordered style="margin: 5px">
      <a-descriptions-item label="C区点占比">{{graphData.pointsCRadioXBar}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点占比" v-if="graphData.specialPointsXBar.length !== 0">{{graphData.pointsSpecialRadioXBar}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点编号" v-if="graphData.specialPointsXBar.length !== 0">{{graphData.specialPointsXBar.join(' ')}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递增" v-if="graphData.ascendChainXBarList.length !== 0">{{chainListDisplay(graphData.ascendChainXBarList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递减" v-if="graphData.descendChainXBarList.length !== 0">{{chainListDisplay(graphData.descendChainXBarList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续落在中心线上侧" v-if="graphData.upperChainXBarList.length !== 0">{{chainListDisplay(graphData.upperChainXBarList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续落在中心线下侧" v-if="graphData.lowerChainXBarList.length !== 0">{{chainListDisplay(graphData.lowerChainXBarList)}}</a-descriptions-item>
    </a-descriptions><br/><br/>

    <a-descriptions bordered style="margin: 5px">
      <a-descriptions-item label="S控制图">
        <div class="chart-container">
          <div class="chart" id="spcChartS"></div>
        </div>
      </a-descriptions-item>
    </a-descriptions>

    <a-descriptions  bordered style="margin: 5px">
      <a-descriptions-item label="C区点占比">{{graphData.pointsCRadioS}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点占比" v-if="graphData.specialPointsS.length !== 0">{{graphData.pointsSpecialRadioS}}</a-descriptions-item>

      <a-descriptions-item label="超出控制限点编号" v-if="graphData.specialPointsS.length !== 0">{{graphData.specialPointsS.join(' ')}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递增" v-if="graphData.ascendChainSList.length !== 0">{{chainListDisplay(graphData.ascendChainSList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续递减" v-if="graphData.descendChainSList.length !== 0">{{chainListDisplay(graphData.descendChainSList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续落在中心线上侧" v-if="graphData.upperChainSList.length !== 0">{{chainListDisplay(graphData.upperChainSList)}}</a-descriptions-item>

      <a-descriptions-item label="多点连续落在中心线下侧" v-if="graphData.lowerChainSList.length !== 0">{{chainListDisplay(graphData.lowerChainSList)}}</a-descriptions-item>
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
  name: "GraphRegression",

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

      const spcChartXBar = echarts.init(document.getElementById("spcChartXBar"))
      spcChartXBar.setOption({
        legend: {
          data: ['均值'],
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
          name: '均值',
          nameLocation: 'center',
          nameGap: 40,
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },

          // 为了显示UCL和LCL保留
          max: graphData.value.graduationXBar - (graphData.value.graduationXBar % 1) + 1,
          min: -(graphData.value.graduationXBar - (graphData.value.graduationXBar % 1) + 1),
          interval: 1
        },

        series: [
          {
            name: '均值',
            data: graphData.value.dataArrayXBar,
            type: 'line',
            itemStyle: {
              color: (params) => {
                if (params.value > graphData.value.uclXBar) return '#ff6347'
                else if (params.value < graphData.value.lclXBar) return '#ff00ff'
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
                  yAxis: graphData.value.uclXBar,
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
                  yAxis: graphData.value.clXBar,
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
                  yAxis: graphData.value.lclXBar,
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


      const spcChartS = echarts.init(document.getElementById("spcChartS"))
      spcChartS.setOption({
        legend: {
          data: ['标准差'],
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
          name: '标准差',
          nameLocation: 'center',
          nameGap: 40,
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },
          max: graphData.value.graduationS - (graphData.value.graduationS % 1) + 1,
          min: 0,
          interval: 1
        },

        series: [
          {
            name: '标准差',
            data: graphData.value.dataArrayS,
            type: 'line',
            itemStyle: {
              color: (params) => {
                if (params.value > graphData.value.uclS) return '#ff6347'
                else if (params.value < graphData.value.lclS) return '#ff00ff'
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
                  yAxis: graphData.value.uclS,
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
                  yAxis: graphData.value.clS,
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
                  yAxis: graphData.value.lclS,
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
