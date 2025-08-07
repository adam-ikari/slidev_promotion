<script setup lang="ts">
import { onMounted, onUnmounted, ref, watch } from 'vue'

const container = ref<HTMLDivElement | null>(null)
let scene: any
let camera: any
let renderer: any
let cube: any
let animationId: number

// 动态导入 three.js 以避免服务端渲染问题
let THREE: any

// 初始化Three.js场景的函数
const initScene = async () => {
  if (!container.value) return

  // 动态导入 three.js
  if (!THREE) {
    const threeModule = await import('three')
    THREE = threeModule
  }

  // 如果已经存在场景元素，先清理
  if (renderer) {
    renderer.dispose()
    if (container.value && renderer.domElement) {
      container.value.removeChild(renderer.domElement)
    }
  }
  
  // 清理动画循环
  if (animationId) {
    cancelAnimationFrame(animationId)
  }

  // 创建场景
  scene = new THREE.Scene()
  scene.background = null // 设置背景为透明
  
  // 创建相机
  camera = new THREE.PerspectiveCamera(75, container.value.clientWidth / container.value.clientHeight, 0.1, 1000)
  
  // 创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true }) // 启用透明背景
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  renderer.setClearColor(0x000000, 0) // 设置 clearColor 为透明
  container.value.appendChild(renderer.domElement)
  
  // 创建立方体
  const geometry = new THREE.BoxGeometry(2, 2, 2)
  const material = new THREE.MeshPhongMaterial({ 
    color: 0x00aaff,
    shininess: 100,
    transparent: true,
    opacity: 0.9
  })
  cube = new THREE.Mesh(geometry, material)
  scene.add(cube)
  
  // 添加灯光
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5)
  scene.add(ambientLight)
  
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(5, 5, 5)
  scene.add(directionalLight)
  
  // 设置相机位置
  camera.position.z = 5
  
  // 动画循环
  const animate = () => {
    animationId = requestAnimationFrame(animate)
    
    // 旋转立方体
    if (cube) {
      cube.rotation.x += 0.01
      cube.rotation.y += 0.01
    }
    
    renderer.render(scene, camera)
  }
  
  animate()
}

// 处理窗口大小调整
const handleResize = () => {
  if (!container.value || !camera || !renderer) return
  camera.aspect = container.value.clientWidth / container.value.clientHeight
  camera.updateProjectionMatrix()
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
}

onMounted(async () => {
  await initScene()
  
  window.addEventListener('resize', handleResize)
  setTimeout(handleResize, 0)
})

// 使用ResizeObserver监听容器大小变化，以便在幻灯片切换时重新初始化
const resizeObserver = new ResizeObserver(() => {
  if (container.value) {
    // 容器大小变化时重新初始化场景
    initScene()
  } else {
    handleResize()
  }
})

// 在容器可用时开始观察大小变化
watch(container, (newVal) => {
  if (newVal) {
    resizeObserver.observe(newVal)
  }
}, { immediate: true })

onUnmounted(() => {
  // 清理事件监听器
  window.removeEventListener('resize', handleResize)
  
  // 停止观察
  if (container.value) {
    resizeObserver.unobserve(container.value)
  }
  
  // 清理动画循环
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  
  // 清理渲染器
  if (renderer) {
    renderer.dispose()
  }
})
</script>

<template>
  <div class="three-demo-container" ref="container" style="width: 100%; height: 400px;"></div>
</template>

<style scoped>
.three-demo-container {
  overflow: hidden;
  background: transparent; /* 设置容器背景为透明 */
}
</style>