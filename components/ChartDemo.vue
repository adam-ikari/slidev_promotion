<script setup lang="ts">
import { onMounted, onUnmounted, ref, watch } from 'vue'

const chartContainer = ref<HTMLDivElement | null>(null)
let chartInstance: any = null

// 动态导入 echarts 以避免服务端渲染问题
let echarts: any

// 添加一个函数来初始化或更新图表
const initChart = async () => {
  if (!chartContainer.value) return

  // 如果echarts未加载，则动态导入
  if (!echarts) {
    const echartsModule = await import('echarts')
    echarts = echartsModule
  }

  // 如果图表实例已存在，则先销毁
  if (chartInstance) {
    chartInstance.dispose()
  }

  // 初始化图表
  chartInstance = echarts.init(chartContainer.value)

  // 图表配置
  const option = {
    title: {
      text: 'Slidev 功能使用统计',
      subtext: '2025年数据',
      left: 'center',
      textStyle: {
        color: '#000000'
      },
      subtextStyle: {
        color: '#cccccc'
      }
    },
    tooltip: {
      trigger: 'item'
    },
    legend: {
      orient: 'vertical',
      left: 'left',
      textStyle: {
        color: '#000000'
      }
    },
    series: [
      {
        name: '使用比例',
        type: 'pie',
        radius: '50%',
        data: [
          { value: 40, name: '动画效果' },
          { value: 25, name: '代码高亮' },
          { value: 20, name: 'Vue组件' },
          { value: 10, name: '主题定制' },
          { value: 5, name: '导出功能' }
        ],
        emphasis: {
          itemStyle: {
            shadowBlur: 10,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.5)'
          }
        },
        label: {
          color: '#000000'
        }
      }
    ],
    backgroundColor: 'transparent'
  }

  // 设置图表选项
  chartInstance.setOption(option)
}

onMounted(async () => {
  await initChart()

  // 响应窗口大小变化
  const resizeHandler = () => {
    if (chartInstance) {
      chartInstance.resize()
    }
  }

  window.addEventListener('resize', resizeHandler)
  setTimeout(resizeHandler, 0)

  // 清理事件监听器
  onUnmounted(() => {
    window.removeEventListener('resize', resizeHandler)
    if (chartInstance) {
      chartInstance.dispose()
    }
  })
})

// 监听容器大小变化，以便在幻灯片切换时重新渲染图表
const resizeObserver = new ResizeObserver(() => {
  if (chartInstance) {
    chartInstance.resize()
  } else {
    // 如果图表实例不存在，尝试重新初始化
    initChart()
  }
})

// 在组件挂载后开始观察容器大小变化
watch(chartContainer, (newVal) => {
  if (newVal) {
    resizeObserver.observe(newVal)
  }
}, { immediate: true })

// 组件卸载时停止观察
onUnmounted(() => {
  if (chartContainer.value) {
    resizeObserver.unobserve(chartContainer.value)
  }
})
</script>

<template>
  <div class="chart-container" ref="chartContainer" style="width: 400pz; height: 400px;"></div>
</template>

<style scoped>
.chart-container {
  overflow: hidden;
  background: transparent;
}
</style>