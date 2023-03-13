<template>
  <a-divider>数据基本信息</a-divider>

  <a-descriptions bordered style="margin: 5px">
    <a-descriptions-item label="子组数">{{graphData.subgroupTotal}}</a-descriptions-item>
    <a-descriptions-item label="子组容量">{{graphData.subgroupCapacity}}</a-descriptions-item>
    <a-descriptions-item label="总样本数">{{graphData.sampleNum}}</a-descriptions-item>
    <a-descriptions-item label="总不良品数">{{graphData.defectsNum}}</a-descriptions-item>
    <a-descriptions-item label="平均不良率">{{graphData.avgDefectNum}}</a-descriptions-item>
    <a-descriptions-item label="上限值UCL">{{graphData.ucl}}</a-descriptions-item>
    <a-descriptions-item label="下限值UCL">{{graphData.lcl}}</a-descriptions-item>
  </a-descriptions>
  <a-descriptions bordered style="margin: 5px">
    <a-descriptions-item label="计数值数据直方图">
      <div class="chart-container">
        <div class="chart" id="barChart"></div>
      </div>
    </a-descriptions-item>
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
</template>

<script>
import {useRoute} from "vue-router";
import {onMounted, ref} from "vue";
import * as echarts from "echarts"
import {message} from "ant-design-vue";

export default {
  name: "GraphCnP",

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

      // 统计频数
      let count = {}
      let data = graphData.value.dataArray
      data.forEach(
        function(item) {
          let key = Math.floor(item)
          if (key in count) count[key]++
          else count[key] = 1
        }
      )

      // 提取数据
      let keys = Object.keys(count)
      let values = [];
      keys.forEach(
        function(key) {
          values.push(count[key])
        }
      )

      const barChart = echarts.init(document.getElementById('barChart'))
      barChart.setOption({
        legend: {
          data: [ '频数' ],
          orient: 'horizontal',
          left: 'center',
        },

        tooltip: {
          trigger: 'axis',
        },

        xAxis: {
          type: 'category',
          name: '不良品数量/缺陷数量',
          nameLocation: 'center',
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },
          nameGap: 40,
          data: keys,
        },

        yAxis: {
          type: 'value',
          name: '频数',
          nameLocation: 'center',
          nameTextStyle: {
            color: '#000',
            fontWeight: 'bold',
            fontSize: 15
          },
          nameGap:40,
          // interval: 1,
        },

        series: [{
          name: '频数',
          type: 'bar',
          data: values,
          barWidth: '80%',
          itemStyle: {
            color: 'rgba(0, 119, 204, 0.6)'
          }
        }]
      })

      const spcChart = echarts.init(document.getElementById('spcChart'))
      spcChart.setOption({
        //title: {
        //  text: graphData.value.graphType + "控制图",
        //  left: 'center',
        //  nameTextStyle: {
        //    color: '#000',
        //    fontWeight: 'bold',
        //    fontSize: 20
        //  },
        //},

        //toolbox: {
        //  show: true,
        //  feature: {
        //    saveAsImage: {
        //      show: true,
        //      title: '保存为图片',
        //      type: 'png',
        //      pixelRatio: 1,
        //      lang: ['点击保存'] // 设置保存按钮的文本
        //    },
        //    magicType: {
        //      show: true,
        //      position: 'bottom'
        //    }
        //  }
        //},

        legend: {
          data: [ '不良品数量/缺陷数量' ],
          orient: 'horizontal',
          left: 'center',
          //top: '7%'
        },

        tooltip: {
          trigger: 'axis',
          //axisPointer: { type: 'cross' }
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
          // min: 0,
          // max: graphData.value.graduation - (graphData.value.graduation % 5) + 5,
          // interval: 5,
          name: '不良品数量/缺陷数量',
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
                    formatter: '{b} {c}'
                  }
                },
              ]
            }
          },
          {
            name: '不良品数量/缺陷数量',
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
        /*
        graphic:{
          type: 'group',
          right: 60,
          top: 5,
          children: [
            {
              type: 'text',
              left: 0,
              top: 10,
              z: 101,
              style: {
                fill: '#000',
                text: '子组数: ' + graphData.value.subgroupTotal,
              }
            },
            {
              type: 'text',
              left: 0,
              top: 30,
              z: 101,
              style: {
                fill: '#000',
                text: '子组容量: ' + graphData.value.subgroupCapacity,
              }
            },
            {
              type: 'text',
              left: 0,
              top: 50,
              z: 101,
              style: {
                fill: '#000',
                text: '总样本数: '+ graphData.value.sampleNum,
              }
            },
            {
              type: 'text',
              left: 0,
              top: 70,
              z: 101,
              style: {
                fill: '#000',
                text: '总缺陷数: '+ graphData.value.defectsNum,
              }
            },
            {
              type: 'text',
              left: 0,
              top: 90,
              z: 101,
              style: {
                fill: '#000',
                text: '平均缺陷数: '+ graphData.value.avgDefectNum,
              }
            },
            {
              type: 'text',
              left: 0,
              top: 110,
              z: 101,
              style: {
                fill: '#000',
                text: '上限值 UCL: '+ graphData.value.ucl,
              }
            },
            {
              type: 'text',
              left: 0,
              top: 130,
              z: 101,
              style: {
                fill: '#000',
                text: '下限值 LCL: '+ graphData.value.lcl,
              }
            },
          ]
        }
        */
      })
    })

    return {
      graphData,
      chainListDisplay,
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
