<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'

const container = ref<HTMLDivElement | null>(null)
let scene: any
let camera: any
let renderer: any
let cube: any
let animationId: number

// 动态导入 three.js 以避免服务端渲染问题
let THREE: any

onMounted(async () => {
  if (!container.value) return

  // 动态导入 three.js
  const threeModule = await import('three')
  THREE = threeModule

  // 创建场景
  scene = new THREE.Scene()
  
  // 创建相机
  camera = new THREE.PerspectiveCamera(75, container.value.clientWidth / container.value.clientHeight, 0.1, 1000)
  
  // 创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  renderer.setClearColor(0x1e1e20)
  container.value.appendChild(renderer.domElement)
  
  // 创建立方体
  const geometry = new THREE.BoxGeometry(2, 2, 2)
  const material = new THREE.MeshPhongMaterial({ 
    color: 0x00aaff,
    shininess: 100 
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
  
  // 窗口大小调整
  const handleResize = () => {
    if (!container.value || !camera || !renderer) return
    camera.aspect = container.value.clientWidth / container.value.clientHeight
    camera.updateProjectionMatrix()
    renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  }
  
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
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
  border: 1px solid #444;
  border-radius: 8px;
  overflow: hidden;
}
</style>