<template>
  <div id="canvas"></div>
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
// var scene, mesh, textureLine // scene,mesh 两个用全局变量，不放属性里面就可以了，具体原因不知?????????
let camera, controls, scene, renderer, createModle, clock, composer, container, stats
let radius = 10.0
export default {
  name: 'heatMap',
  data() {
    return {}
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      container = document.getElementById('canvas')
      // 渲染器
      renderer = new THREE.WebGLRenderer({ antialias: true })
      renderer.setPixelRatio(window.devicePixelRatio)
      renderer.setSize(window.innerWidth, window.innerHeight)
      renderer.shadowMap.enabled = true
      //   container = document.body.appendChild(renderer.domElement)
      container.appendChild(renderer.domElement)
      // 相机
      camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 1, 200000)
      camera.position.set(500, 500, 2000)
      // 相机控制
      controls = new OrbitControls(camera, container)
      controls.target.set(500, 500, 0)
      // 场景
      scene = new THREE.Scene()
      for (let i = 0; i < 1000; i++) {
        this.createModle(1000 * Math.random(), 1000 * Math.random(), 0)
      }
      composer = new EffectComposer(renderer)
      composer.addPass(new RenderPass(scene, camera))
      var textureLoader = new THREE.TextureLoader()
      var shaderPass = new ShaderPass({
        uniforms: {
          tDiffuse: { type: 't', value: null },
          grayTexture: { type: 't', value: null }
        },
        vertexShader: ` varying vec2 vUv;
void main() {
vUv = uv;
vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
gl_Position = projectionMatrix * mvPosition;
}`,
        fragmentShader: `uniform sampler2D tDiffuse;
uniform sampler2D grayTexture;
varying vec2 vUv;
void main() {
vec4 texel = texture2D( tDiffuse, vUv );
if(texel.a > 1.0)
texel.a = 1.0;
vec4 heatColor = texture2D( grayTexture, vec2(texel.a, texel.a));
gl_FragColor = vec4(heatColor.rgb, 1.0);
}`
      })
      // shaderPass.uniforms.grayTexture.value = textureLoader.load('./colorline.png');
      shaderPass.uniforms.grayTexture.value = textureLoader.load('static/texture/colorline.png')
      shaderPass.renderToScreen = true
      composer.addPass(shaderPass)
      // var effectCopy = new THREE.ShaderPass(THREE.CopyShader);
      // effectCopy.renderToScreen = true;
      // composer.addPass( effectCopy );
      // 窗口大小改变
      window.addEventListener('resize', this.onWindowResize, false)
      // 帧率统计信息
      stats = new Stats()
      container.appendChild(stats.dom)
      clock = new THREE.Clock()
      // 开启渲染
      this.animate()
    },
    // 窗口大小改变
    onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(window.innerWidth, window.innerHeight)
      composer.setSize(window.innerWidth, window.innerHeight)
    },
    // 渲染回调
    animate() {
      controls.update()
      let time = clock.getDelta()
      requestAnimationFrame(this.animate)
      // renderer.render(scene, camera);
      composer.render()
      stats.update()
    },
    createModle(x, y, z) {
      // 创建几何buffer
      let geometry = new THREE.BufferGeometry()
      let vertices = []
      vertices.push(-10, -10, 0)
      vertices.push(10, -10, 0)
      vertices.push(-10, 10, 0)
      vertices.push(10, -10, 0)
      vertices.push(10, 10, 0)
      vertices.push(-10, 10, 0)
      geometry.addAttribute('position', new THREE.Float32BufferAttribute(vertices, 3))
      // 创建shader
      let shaderMaterail = new THREE.ShaderMaterial({
        uniforms: {
          radius: { value: radius }
        },
        vertexShader: ` varying vec2 pos;
void main() {
pos = position.xy;
vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
gl_Position = projectionMatrix * mvPosition;
}`,
        fragmentShader: `uniform float radius;
varying vec2 pos;
void main() {
float flag = sqrt(pow(pos.x, 2.0) + pow(pos.y, 2.0));
if(flag > radius)
discard;
else{
float f = (radius - flag) / radius;
gl_FragColor = vec4(f, f, f, f);
}
}`,
        // side : THREE.DoubleSide
        transparent: true,
        opacity: 0.5,
        depthTest: false
      })
      let mesh = new THREE.Mesh(geometry, shaderMaterail)
      mesh.position.set(x, y, 0)
      scene.add(mesh)
    }
  }
}
</script>

<style scoped>
#canvas {
  width: 100%;
  height: 100%;
  background-color: #00ff00;
}
</style>