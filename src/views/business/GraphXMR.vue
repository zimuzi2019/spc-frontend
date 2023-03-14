<template>
  <a-divider>数据基本信息</a-divider>

  <a-descriptions bordered style="margin: 5px">
    <a-descriptions-item label="总批次数">{{graphData.subgroupTotal}}</a-descriptions-item>
    <a-descriptions-item label="平均值">{{graphData.avgX}}</a-descriptions-item>
    <a-descriptions-item label="最大值">{{graphData.maxX}}</a-descriptions-item>
    <a-descriptions-item label="最小值">{{graphData.minX}}</a-descriptions-item>
    <a-descriptions-item label="中位数">{{graphData.midX}}</a-descriptions-item>
    <a-descriptions-item label="标准差">{{graphData.stdX}}</a-descriptions-item>
    <a-descriptions-item label="偏度">{{graphData.skewnessX}}</a-descriptions-item>
    <a-descriptions-item label="峰度">{{graphData.kurtosisX}}</a-descriptions-item>
    <a-descriptions-item label="规范上限 USL">{{graphData.usl}}</a-descriptions-item>
    <a-descriptions-item label="中心限 SL">{{graphData.sl}}</a-descriptions-item>
    <a-descriptions-item label="规范下限 LSL">{{graphData.lsl}}</a-descriptions-item>
    <a-descriptions-item label="预估不良率 PPM">{{graphData.ppm}}</a-descriptions-item>
    <a-descriptions-item label="上限值 UCL (X)">{{graphData.uclX}}</a-descriptions-item>
    <a-descriptions-item label="中心限 CL (X)">{{graphData.clX}}</a-descriptions-item>
    <a-descriptions-item label="下限值 LCL (X)">{{graphData.lclX}}</a-descriptions-item>
    <a-descriptions-item label="上限值 UCL (MR)">{{graphData.uclMR}}</a-descriptions-item>
    <a-descriptions-item label="中心限 CL (MR)">{{graphData.uclMR}}</a-descriptions-item>
    <a-descriptions-item label="下限值 LCL (MR)">{{graphData.lclMR}}</a-descriptions-item>
    <a-descriptions-item label="标准偏差">{{graphData.sigma}}</a-descriptions-item>
    <a-descriptions-item label="Pp">{{graphData.pp}}</a-descriptions-item>
    <a-descriptions-item label="Ppk">{{graphData.ppk}}</a-descriptions-item>
    <a-descriptions-item label="Ca">{{graphData.ca}}</a-descriptions-item>
    <a-descriptions-item label="Cp">{{graphData.cp}}</a-descriptions-item>
    <a-descriptions-item label="CPU">{{graphData.cpu}}</a-descriptions-item>
    <a-descriptions-item label="CPL">{{graphData.cpl}}</a-descriptions-item>
    <a-descriptions-item label="Cpk">{{graphData.cpk}}</a-descriptions-item>
    <a-descriptions-item label="Cpk Grade">{{graphData.cpkGrade}}</a-descriptions-item>
    <a-descriptions-item label="数据分布直方图">
      <div class="chart-container">
        <div class="chart" id="barChart"></div>
      </div>
    </a-descriptions-item>
  </a-descriptions>

  <a-divider>控制图分析</a-divider>

  <a-descriptions bordered style="margin: 5px">
    <a-descriptions-item label="X控制图">
      <div class="chart-container">
        <div class="chart" id="spcChartX"></div>
      </div>
    </a-descriptions-item>
  </a-descriptions>

  <a-descriptions  bordered style="margin: 5px">
    <a-descriptions-item label="C区点占比">{{graphData.pointsCRadioX}}</a-descriptions-item>

    <a-descriptions-item label="超出控制限点占比" v-if="graphData.specialPointsX.length !== 0">{{graphData.pointsSpecialRadioX}}</a-descriptions-item>

    <a-descriptions-item label="超出控制限点编号" v-if="graphData.specialPointsX.length !== 0">{{graphData.specialPointsX.join(' ')}}</a-descriptions-item>

    <a-descriptions-item label="多点连续递增" v-if="graphData.ascendChainXList.length !== 0">{{chainListDisplay(graphData.ascendChainXList)}}</a-descriptions-item>

    <a-descriptions-item label="多点连续递减" v-if="graphData.descendChainXList.length !== 0">{{chainListDisplay(graphData.descendChainXList)}}</a-descriptions-item>

    <a-descriptions-item label="多点连续落在中心线上侧" v-if="graphData.upperChainXList.length !== 0">{{chainListDisplay(graphData.upperChainXList)}}</a-descriptions-item>

    <a-descriptions-item label="多点连续落在中心线下侧" v-if="graphData.lowerChainXList.length !== 0">{{chainListDisplay(graphData.lowerChainXList)}}</a-descriptions-item>
  </a-descriptions><br/><br/>

  <a-descriptions bordered style="margin: 5px">
    <a-descriptions-item label="MR控制图">
      <div class="chart-container">
        <div class="chart" id="spcChartMR"></div>
      </div>
    </a-descriptions-item>
  </a-descriptions>

  <a-descriptions  bordered style="margin: 5px">
    <a-descriptions-item label="C区点占比">{{graphData.pointsCRadioMR}}</a-descriptions-item>

    <a-descriptions-item label="超出控制限点占比" v-if="graphData.specialPointsMR.length !== 0">{{graphData.pointsSpecialRadioMR}}</a-descriptions-item>

    <a-descriptions-item label="超出控制限点编号" v-if="graphData.specialPointsMR.length !== 0">{{graphData.specialPointsMR.join(' ')}}</a-descriptions-item>

    <a-descriptions-item label="多点连续递增" v-if="graphData.ascendChainMRList.length !== 0">{{chainListDisplay(graphData.ascendChainMRList)}}</a-descriptions-item>

    <a-descriptions-item label="多点连续递减" v-if="graphData.descendChainMRList.length !== 0">{{chainListDisplay(graphData.descendChainMRList)}}</a-descriptions-item>

    <a-descriptions-item label="多点连续落在中心线上侧" v-if="graphData.upperChainMRList.length !== 0">{{chainListDisplay(graphData.upperChainMRList)}}</a-descriptions-item>

    <a-descriptions-item label="多点连续落在中心线下侧" v-if="graphData.lowerChainMRList.length !== 0">{{chainListDisplay(graphData.lowerChainMRList)}}</a-descriptions-item>
  </a-descriptions>

</template>

<script>
import {useRoute} from "vue-router";
import {onMounted, ref} from "vue";
import * as echarts from "echarts";

export default {
  name: "GraphXMR",

  setup() {
    const route = useRoute()
    const graphData = ref(JSON.parse(route.params.graphData))

    const chainListDisplay = function(chainList){
      return chainList.map(function(item) {
        return item.join('-')
      }).join(' ')
    }

    onMounted(() => {
      console.log(graphData.value)

      let count = {'<μ-3.25σ': 0, 'μ-3σ': 0, 'μ-2.5σ': 0, 'μ-2σ': 0, 'μ-1.5σ': 0, 'μ-σ': 0, 'μ-0.5σ': 0, 'μ': 0, 'μ+0.5σ': 0, 'μ+σ': 0, 'μ+1.5σ': 0, 'μ+2σ': 0, 'μ+2.5σ': 0, 'μ+3σ': 0, '>μ+3.25σ': 0}
      let data = graphData.value.dataArrayX
      let avgX = graphData.value.avgX
      let stdX = graphData.value.stdX

      data.forEach(
        function(item) {
          if (item < avgX - 3.25*stdX) count['<μ-3.25σ']++
          else if (item >= avgX - 3.25*stdX && item < avgX - 2.75*stdX) count['μ-3σ']++
          else if (item >= avgX - 2.75*stdX && item < avgX - 2.25*stdX) count['μ-2.5σ']++
          else if (item >= avgX - 2.25*stdX && item < avgX - 1.75*stdX) count['μ-2σ']++
          else if (item >= avgX - 1.75*stdX && item < avgX - 1.25*stdX) count['μ-1.5σ']++
          else if (item >= avgX - 1.25*stdX && item < avgX - 0.75*stdX) count['μ-σ']++
          else if (item >= avgX - 0.75*stdX && item < avgX - 0.25*stdX) count['μ-0.5σ']++
          else if (item >= avgX - 0.25*stdX && item < avgX + 0.25*stdX) count['μ']++
          else if (item >= avgX + 0.25*stdX && item < avgX + 0.75*stdX) count['μ+0.5σ']++
          else if (item >= avgX + 0.75*stdX && item < avgX + 1.25*stdX) count['μ+σ']++
          else if (item >= avgX + 1.25*stdX && item < avgX + 1.75*stdX) count['μ+1.5σ']++
          else if (item >= avgX + 1.75*stdX && item < avgX + 2.25*stdX) count['μ+2σ']++
          else if (item >= avgX + 2.25*stdX && item < avgX + 2.75*stdX) count['μ+2.5σ']++
          else if (item >= avgX + 2.75*stdX && item < avgX + 3.25*stdX) count['μ+3σ']++
          else count['>μ+3.25σ']++
        }
      )

      let keys = ['<μ-3.25σ', 'μ-3σ', 'μ-2.5σ', 'μ-2σ', 'μ-1.5σ', 'μ-σ', 'μ-0.5σ', 'μ', 'μ+0.5σ', 'μ+σ', 'μ+1.5σ', 'μ+2σ', 'μ+2.5σ', 'μ+3σ', '>μ+3.25σ']
      let values = []
      keys.forEach(
        function(key) {
          values.push(count[key])
        }
      )

      let n = graphData.value.subgroupTotal
      values = values.map(
        function (item) {
          return Math.floor(item / n * 100) / 100;
        }
      )

      // function normalDistributionCal(x, avgX, stdX) {
      //    return (
      //      ( 1 / (Math.sqrt(2 * Math.PI) * stdX) ) * Math.exp( -1 * (x - avgX) * (x - avgX) / (2 * stdX * stdX))
      //    )
      //  }

      // let samplePoints = [
      //    -3*stdX, -2.5*stdX, -2*stdX, -1.5*stdX, -stdX, -0.5*stdX, 0, 0.5*stdX, stdX, 1.5*stdX, 2*stdX, 2.5*stdX, 3*stdX
      // ]

      // let normalDistributionData = samplePoints.map(
      //   function(item) {
      //     return normalDistributionCal(item, avgX, stdX)
      //   }
      // )
      // normalDistributionData.unshift(null)
      // normalDistributionData.push(null)

      // console.log(normalDistributionData)

      let normalDistributionData = [0.0006, 0.0024, 0.0092, 0.0279, 0.0655, 0.1210, 0.1747, 0.0987*2, 0.1747, 0.1210, 0.0655, 0.0279, 0.0092, 0.0024, 0.0006]

      const barChart = echarts.init(document.getElementById('barChart'))
      barChart.setOption({
        legend: {
          data: [ '区间概率', '正态分布参考值' ],
          orient: 'horizontal',
          left: 'center',
        },

        tooltip: {
          trigger: 'axis',
          axisPointer: { type: 'shadow' }
        },

        xAxis: {
          type: 'category',
          name: '区间',
          nameLocation: 'center',
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },
          nameGap: 50,
          data: keys,
          axisTick: {
            alignWithLabel: true
          },
          axisLabel: {
            rotate: 45,
          }
        },

        yAxis: {
          type: 'value',
          name: '概率',
          nameLocation: 'center',
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },
          nameGap:40,
        },

        series: [
          {
            name: '区间概率',
            type: 'bar',
            barWidth: '100%',
            data: values,
            label: {
              show: true,
              position: 'inside',
              formatter: '{c}',
            },
            itemStyle: {
              color: 'rgba(0, 119, 204, 0.6)'
            }
          },
          {
            name: '正态分布参考值',
            type: 'line',
            smooth: true,
            data: normalDistributionData,
          }]
      })



      let markLineData = [
        {
          name: 'UCL',
          yAxis: graphData.value.uclX,
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
          yAxis: graphData.value.clX,
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
        }
      ]
      // X-MR图的X图LSL可能小于0
      if (graphData.value.lclX >= 0){
        markLineData.push({
          name: 'LCL',
          yAxis: graphData.value.lclX,
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
        })
      }

      const spcChartX = echarts.init(document.getElementById("spcChartX"))
      spcChartX.setOption({
        legend: {
          data: ['样本测量值'],
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
          name: '样本测量值',
          nameLocation: 'center',
          nameGap: 40,
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },
          min: 0,
          max: graphData.value.graduationX - (graphData.value.graduationX % 50) + 50,
          interval: 50
        },

        series: [
          {
            name: '样本测量值',
            data: graphData.value.dataArrayX,
            type: 'line',
            itemStyle: {
              color: (params) => {
                if (params.value > graphData.value.uclX) return '#ff6347'
                else if (params.value < graphData.value.lclX) return '#ff00ff'
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
              data: markLineData
            }
          }
        ]
      })

      let dataMR = graphData.value.dataArrayMR
      dataMR.unshift(null)
      const spcChartMR = echarts.init(document.getElementById("spcChartMR"))
      spcChartMR.setOption({
        legend: {
          data: ['极差值'],
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
          name: '样本测量值',
          nameLocation: 'center',
          nameGap: 40,
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },
          min: 0,
          max: graphData.value.graduationMR - (graphData.value.graduationMR % 50) + 50,
          interval: 50
        },

        series: [
          {
            name: '极差值',
            data: dataMR,
            type: 'line',
            itemStyle: {
              color: (params) => {
                if (params.value > graphData.value.uclMR) return '#ff6347'
                else if (params.value < graphData.value.lclMR) return '#ff00ff'
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
                  yAxis: graphData.value.uclMR,
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
                  yAxis: graphData.value.clMR,
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
                  yAxis: graphData.value.lclMR,
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
      chainListDisplay
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
