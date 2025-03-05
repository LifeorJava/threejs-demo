<template>
    <div>
      <!-- 标题 -->
      <h2>选择图表类型:</h2>
      <!-- 按钮用于切换图表类型 -->
      <button @click="setChartType('bar')">柱状图</button>
      <button @click="setChartType('line')">曲线图</button>
      <!-- 图表容器 -->
      <div ref="chart" :class="className" :style="{ height: height, width: width }"></div>
    </div>
  </template>
  
  <script>
  import * as echarts from 'echarts';
  import 'echarts-gl'; // 引入 ECharts GL 扩展
  
  export default {
    props: {
      className: {
        type: String,
        default: 'chart'
      },
      width: {
        type: String,
        default: '100%'
      },
      height: {
        type: String,
        default: '600px'
      },
      dataList: {
        type: Array,
        default: () => [
          [0, 0, 5],
          [0, 1, 1],
          [0, 2, 0],
          [0, 3, 0],
          [0, 4, 0],
          [0, 5, 0],
          [0, 6, 0],
          [1, 0, 7],
          [1, 1, 0],
          [1, 2, 0],
          [1, 3, 0],
          [1, 4, 0],
          [1, 5, 0],
          [1, 6, 0],
          [2, 0, 5],
          [2, 1, 1],
          [2, 2, 0],
          [2, 3, 0],
          [2, 4, 0],
          [2, 5, 0],
          [2, 6, 0],
          [3, 0, 7],
          [3, 1, 0],
          [3, 2, 0],
          [3, 3, 0],
          [3, 4, 0],
          [3, 5, 0],
          [3, 6, 0],
          [4, 0, 5],
          [4, 1, 1],
          [4, 2, 0],
          [4, 3, 0],
          [4, 4, 0],
          [4, 5, 0],
          [4, 6, 0]
        ]
      }
    },
    data() {
      return {
        chart: null, // 存储图表实例
        chartType: 'bar' // 默认图表类型为柱状图
      };
    },
    mounted() {
      this.$nextTick(() => {
        this.initChart(); // 初始化图表
      });
    },
    beforeDestroy() {
      if (!this.chart) {
        return;
      }
      this.chart.dispose(); // 销毁图表实例
      this.chart = null;
    },
    watch: {
      dataList() {
        this.initChart(); // 数据变化时重新初始化图表
      },
      chartType() {
        this.initChart(); // 图表类型变化时重新初始化图表
      }
    },
    methods: {
      setChartType(type) {
        this.chartType = type; // 设置图表类型
      },
      initChart() {
        let data = this.dataList;
        // 生成 X 轴类别（时间）
        let dataX = [];
        for (let i = 0; i <= 6; i++) {
          dataX.push(i.toString());
        }
        // 生成 Y 轴类别（类别）
        let dataY = ['生产', '收购', '运输', '销售', '消费'];
        // 准备 3D 系列数据
        var vdata = [];
        for (var k = 0; k < data.length; k++) {
          var xIndex = data[k][1]; // 时间索引
          var yIndex = data[k][0]; // 类别索引
          var value = data[k][2]; // 值
          if (xIndex >= 0 && xIndex < dataX.length && yIndex >= 0 && yIndex < dataY.length) {
            vdata.push([xIndex, yIndex, value]); // 添加到数据数组
          }
        }
        this.chart = echarts.init(this.$refs.chart); // 初始化图表实例
        const option = {
          xAxis3D: {
            type: 'category', // 类别类型
            name: '时间', // 轴名称
            data: dataX, // 数据
            axisLabel: {
              show: true, // 显示标签
              interval: 0 // 显示所有标签
            }
          },
          yAxis3D: {
            type: 'category', // 类别类型
            name: '类别', // 轴名称
            data: dataY, // 数据
            axisLabel: {
              show: true, // 显示标签
              interval: 0 // 显示所有标签
            }
          },
          zAxis3D: {
            type: 'value', // 数值类型
            name: '值' // 轴名称
          },
          grid3D: {
            boxWidth: 150, // 较小的宽度
            boxHeight: 60, // 较小的高度
            boxDepth: 100, // 较小的深度
            viewControl: {
              alpha: 15, // 角度
              beta: 15, // 角度
              distance: 250 // 视距
            }
          },
          tooltip: {
            trigger: 'item', // 触发方式
            formatter: function (params) {
              if (params.componentSubType === 'bar3D') {
                return `时间: ${dataX[params.value[0]]}, 类别: ${dataY[params.value[1]]}, 值: ${params.value[2]}`; // 柱状图提示信息
              } else if (params.componentSubType === 'lines3D') {
                return `时间: ${dataX[Math.floor(params.value[0])]}, 类别: ${dataY[Math.floor(params.value[1])]}`; // 曲线索引提示信息
              }
              return '';
            }
          }
        };
  
        if (this.chartType === 'bar') {
          option.series = [
            {
              type: 'bar3D', // 3D 柱状图类型
              shading: 'color', // 颜色阴影
              data: vdata, // 数据
              itemStyle: {
                opacity: 0.8 // 透明度
              },
              label: {
                show: true, // 显示标签
                formatter: function (params) {
                  return params.value[2]; // 显示值
                },
                textStyle: {
                  color: '#fff', // 文字颜色
                  fontSize: 12 // 字体大小
                }
              }
            }
          ];
        } else if (this.chartType === 'line') {
          // 将 vdata 转换为线格式
          let lineData = [];
          for (let i = 0; i < dataY.length; i++) {
            let categoryLines = [];
            for (let j = 0; j < dataX.length; j++) {
              let found = false;
              for (let k = 0; k < vdata.length; k++) {
                if (vdata[k][0] == j && vdata[k][1] == i) {
                  categoryLines.push(vdata[k]);
                  found = true;
                  break;
                }
              }
              if (!found) {
                categoryLines.push([j, i, 0]); // 添加零值如果不存在数据点
              }
            }
            lineData.push(categoryLines);
          }
  
          // 展平 lineData 数组
          let flatLineData = lineData.flat();
  
          option.series = [
            {
              type: 'lines3D', // 3D 曲线类型
              shading: 'color', // 颜色阴影
              data: flatLineData.map(point => ({
                coords: [[point[0], point[1]], [point[0], point[1]]], // 确保每条线有两个点
                value: point[2]
              })),
              lineStyle: {
                width: 2 // 线宽
              },
              effect: {
                show: true, // 显示特效
                period: 4, // 特效周期
                trailLength: 0.1, // 特效尾迹长度
                symbolSize: 3 // 符号大小
              }
            }
          ];
        }
  
        this.chart.setOption(option); // 设置图表选项
      }
    }
  };
  </script>
  
  <style scoped>
  /* 样式可以根据需要调整 */
  .chart {
    margin-top: 20px; /* 上边距 */
  }
  button {
    margin-right: 10px; /* 右边距 */
    padding: 10px 20px; /* 内边距 */
    font-size: 16px; /* 字体大小 */
    cursor: pointer; /* 鼠标指针样式 */
  }
  </style>
  