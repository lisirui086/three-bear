<script setup>
// 引入组合式API
import { onMounted, ref, onUnmounted } from 'vue'

// 引入threejs
import * as THREE from 'three'

// 引入GLTF库
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'

// 引入控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

// 获取画布节点
const canvas = ref(null)

// 组件挂在完毕
onMounted(()=>{
  // 初始化场景
  const scene = new THREE.Scene()

  // 初始化相机
  const camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )

  // 初始化相机位置
  camera.position.set(1.5, 1, 1.5)

  // 摄像机视锥体的长宽比
  camera.aspect = window.innerWidth / window.innerHeight

  // 更新摄像机投影矩阵。在任何参数被改变以后必须被调用。
  camera.updateProjectionMatrix()

  // 加载背景纹理
  const textureLoader = new THREE.TextureLoader()
  const bgTexture = textureLoader.load('/public/assets/imgs/050.jpg')

  // 将背景纹理映射到全景
  bgTexture.mapping = THREE.EquirectangularRefractionMapping

  // 将背景纹理设置成场景的背景和环境纹理
  scene.background = bgTexture
  scene.environment = bgTexture

  // 添加平行光
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.8)
  scene.add(ambientLight)

  // 加载小熊模型
  const gltfLoader = new GLTFLoader()
  gltfLoader.load('/assets/model/bear.gltf', gltf => {
    const model = gltf.scene.children[0]

    // 设置材质
    model.material = new THREE.MeshPhongMaterial({
      color: 0xffffff,
      envMap: bgTexture,
      // 环境贴图对表面的影响程度
      reflectivity: 0.8,
      // 空气的折射率（IOR）（约为1）除以材质的折射率
      refractionRatio: 0.8
    })
    model.scale.set(1,1,1)
    scene.add(model)
  })

  // 初始化渲染器
  const renderer = new THREE.WebGLRenderer({
    antialias: true,
    preserveDrawingBuffer: true
  })

  // 渲染器大小
  renderer.setSize(window.innerWidth, window.innerHeight)

  // 根据屏幕宽高改变，重新渲染相机，场景宽高大小
  const handleResize = () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  }

  // 将侦听屏幕宽高改变函数添加到window
  window.addEventListener('resize', handleResize)

  // 将渲染器追加到画布上
  canvas.value.appendChild(renderer.domElement)

  // 创建控制器
  const controls = new OrbitControls(camera, canvas.value)

  // 开启控制器阻尼
  controls.enableDamping = true

  // 渲染函数
  const renderFn = () => {
    // 更新控制器
    controls.update()
    // 更新渲染器
    renderer.render(scene, camera)
    // 循环渲染
    requestAnimationFrame(renderFn)
  }

  renderFn()
})

// 组件卸载完毕
onUnmounted(()=>{
  // 卸载侦听屏幕宽高变化，重新渲染相机，场景
  window.removeEventListener('resize', handleResize)
})

</script>

<template>
  <div class='canvas' ref='canvas'></div>
</template>

<style scoped>
.canvas {
  width: 100vw;
  height: 100vh;
}
</style>
