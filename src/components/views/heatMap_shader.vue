<template>
  <!-- <canvas id="heatmap"></canvas> -->
  <div id="info"></div>
</template>

<script>
import Stats from '../threeJS/stats.module'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js'
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js'
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js'
// import { OrbitControls } from './commonJS/OrbitControls'
import { FontLoader } from 'three/examples/jsm/loaders/FontLoader'
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry'
const segments = 30
const w = 256
const h = 256
let renderer, scene, camera, ctx, canvas, controls
export default {
  name: 'heatmapShader',
  data() {
    return {
      container: null
    }
  },
  mounted() {
    this.init()
    this.animate()
  },
  methods: {
    // 随机给出温度值 储存在2维数组
    getTemperature() {
      let temperatureArray = new Array()
      for (let i = 0; i < segments; i++) {
        temperatureArray[i] = parseInt(Math.random() * 25 + 10)
      }
      return temperatureArray
    },
    // 绘制辐射圆
    drawCircular(context, opts) {
      let { x, y, radius, weight } = opts
      radius = parseInt(radius * weight)

      // 创建圆设置填充色
      let rGradient = context.createRadialGradient(x, y, 0, x, y, radius)
      rGradient.addColorStop(0, 'rgba(255, 0, 0, 1)')
      rGradient.addColorStop(1, 'rgba(0, 255, 0, 0)')
      context.fillStyle = rGradient

      // 设置globalAlpha
      context.globalAlpha = weight
      context.beginPath()
      context.arc(x, y, radius, 0, 2 * Math.PI)
      context.closePath()

      context.fill()
    },
    getPaletteMap() {
      // 颜色条的颜色分布
      let colorStops = {
        1.0: '#f00',
        0.8: '#e2fa00',
        0.6: '#33f900',
        0.3: '#0349df',
        0.0: '#fff'
      }

      // 颜色条的大小
      let width = 256,
        height = 10

      // 创建canvas
      let paletteCanvas = document.createElement('canvas')
      //   let paletteCanvas = document.getElementById('heatmap')
      paletteCanvas.width = width
      paletteCanvas.height = height
      paletteCanvas.style.position = 'absolute'
      paletteCanvas.style.top = '0'
      paletteCanvas.style.right = '0'
      //   let ctx = paletteCanvas.getContext('2d')
      ctx = paletteCanvas.getContext('2d')
      // 创建线性渐变色
      let linearGradient = ctx.createLinearGradient(0, 0, width, 0)
      for (const key in colorStops) {
        linearGradient.addColorStop(key, colorStops[key])
      }

      // 绘制渐变色条
      ctx.fillStyle = linearGradient
      ctx.fillRect(0, 0, width, height)

      document.body.appendChild(paletteCanvas)

      let paletteTexture = new THREE.Texture(paletteCanvas)
      paletteTexture.minFilter = THREE.NearestFilter
      paletteTexture.needsUpdate = true

      return paletteTexture
    },
    // 获取透明度阶梯图
    getAlphaScaleMap(width, height) {
      // canvas = document.getElementById('heatmap')
      canvas = document.createElement('canvas')
      canvas.width = width
      canvas.height = height

      //   let context = canvas.getcontext('2d')
      let context = canvas.getContext('2d')

      // 随机生成温度
      let tenperature = this.getTemperature()

      // 绘制透明度阶梯图
      for (let i = 0; i < segments; i++) {
        // 计算出当前温度占标准温度的权值
        let weight = tenperature[i] / 35

        this.drawCircular(context, {
          x: Math.random() * w,
          y: Math.random() * h,
          radius: 80,
          weight: weight
        })
      }

      let tex = new THREE.Texture(canvas)
      tex.minFilter = THREE.NearestFilter
      tex.needsUpdate = true
      return tex
    },
    init() {
      var that = this
      that.container = document.getElementById('info')
      renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true })
      renderer.setPixelRatio(window.devicePixelRatio)
      renderer.setSize(that.container.innerWidth, that.container.innerHeight)
      // renderer.setSize(window.innerWidth, window.innerHeight)

      scene = new THREE.Scene()

      camera = new THREE.PerspectiveCamera(40, that.container.innerWidth / that.container.innerHeight, 1, 10000)
      camera.position.set(0, 0, 3000)
      that.container.appendChild(renderer.domElement)
      scene.add(new THREE.AmbientLight(0xeef0ff))
      // 添加光源
      var spotLight = new THREE.SpotLight(0xcccccc)
      spotLight.position.set(-100, 300, 10)
      scene.add(spotLight)

      var heatMapGeo = new THREE.PlaneBufferGeometry(1500, 1500)

      var heatMapMaterial = new THREE.ShaderMaterial({
        transparent: true,
        // vertexShader: document.getElementById('vertexshader').textContent,
        // fragmentShader: document.getElementById('fragmentshader').textContent,
        vertexShader: `varying vec2 vUv;
    void main() {
      vUv = uv;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }`,
        fragmentShader: `#ifdef GL_ES
    precision highp float;
    #endif
    varying vec2 vUv;
    uniform sampler2D alphaScaleMap;
    uniform sampler2D paletteMap;
    void main() {
      vec4 alphaColor = texture2D(alphaScaleMap, vUv);
      vec4 color = texture2D(paletteMap, vec2(alphaColor.a, 0.0));
      gl_FragColor = vec4(color.r, color.g, color.b, alphaColor.a);
    }`,
        uniforms: {
          alphaScaleMap: {
            type: 't',
            value: this.getAlphaScaleMap(w, h)
          },
          paletteMap: {
            type: 't',
            value: this.getPaletteMap()
          }
        }
      })

      var heatMapPlane = new THREE.Mesh(heatMapGeo, heatMapMaterial)
      scene.add(heatMapPlane)

      controls = new OrbitControls(camera, renderer.domElement)
      controls.enableDamping = true
      controls.dampingFactor = 0.25
      controls.enableZoom = true
      controls.autoRotate = false
      controls.minDistance = 1
      controls.maxDistance = 10000
      controls.enablePan = true
      controls.update()
      window.addEventListener('resize', this.onWindowResize, false)
    },
    onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()

      renderer.setSize(window.innerWidth, window.innerHeight)
    },
    animate() {
      requestAnimationFrame(this.animate)
      controls.update() // only required if controls.enableDamping = true, or if controls.autoRotate = true
      this.render()
    },
    render() {
      renderer.render(scene, camera)
    }
  }
}
</script>

<style scoped>
#info {
  width: 100%;
  height: 100%;
}
</style>