<template>
  <div id="info">
    three.js - animation - groups
    <!-- <canvas id="heatmap"></canvas> -->
  </div>
</template>

<script>
import Stats from '../threeJS/stats.module'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
// import { OrbitControls } from './commonJS/OrbitControls'
import { FontLoader } from 'three/examples/jsm/loaders/FontLoader'
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry'
import { GUI } from 'dat.gui'
var scene, mesh, textureLine, ctx, canvas // scene,mesh 两个用全局变量，不放属性里面就可以了，具体原因不知?????????
const segments = 30
const w = 256
const h = 256
export default {
  name: 'AnimationGroup',
  data() {
    return {
      clock: null,
      container: null,
      // scene: null,
      camera: null,
      renderer: null,
      mixer: null,
      stats: null,
      ambientLight: null,
      spotLight: null,
      controls: ' '
    }
  },
  mounted() {
    this.init()
    this.animate()
  },
  methods: {
    init() {
      var that = this
      that.container = document.getElementById('info')
      scene = new THREE.Scene()

      //

      // 创建渲染器
      that.renderer = new THREE.WebGLRenderer({
        alpha: true, // canvas中是否包含透明度
        antialias: true // 是否执行抗锯齿
      })

      // 配置相机
      that.camera = new THREE.PerspectiveCamera(40, that.container.innerWidth / that.container.innerHeight, 1, 1000)
      that.camera.position.set(50, 50, 100)

      // that.camera.lookAt(scene.position)
      that.camera.lookAt(new THREE.Vector3(0, 0, 0))

      // 配置渲染器
      that.renderer.setPixelRatio(window.devicePixelRatio) // 为了兼容高清屏幕
      that.renderer.outputEncoding = THREE.sRGBEncoding // 转换输出的颜色为sRGB颜色空间，使其输出颜色与建模软件上一致
      that.renderer.antialias = true
      that.renderer.autoClear = true
      that.renderer.setClearColor(0x050505)
      that.renderer.setSize(that.container.innerWidth, that.container.innerHeight)
      that.container.appendChild(that.renderer.domElement)

      // 初始化控制器
      that.controls = new OrbitControls(that.camera, that.renderer.domElement)
      // this.controls.target.set(0, 0, 0);//------------------
      // this.controls.maxPolarAngle = Math.PI / 3;

      // 如果使用animate方法时，将此函数删除
      // controls.addEventListener( 'change', render );
      // 使动画循环使用时阻尼或自转 意思是否有惯性
      that.controls.enableDamping = true
      // 动态阻尼系数 就是鼠标拖拽旋转灵敏度
      that.controls.dampingFactor = 0.25
      // 是否可以缩放
      that.controls.enableZoom = true
      // 是否自动旋转
      that.controls.autoRotate = false
      // 设置相机距离原点的最远距离
      that.controls.minDistance = 1
      // 设置相机距离原点的最远距离
      that.controls.maxDistance = 10000
      // 是否开启右键拖拽
      that.controls.enablePan = true
      that.controls.update()

      // 创建灯光
      that.ambientLight = new THREE.AmbientLight({ color: 0x404040 })
      scene.add(that.ambientLight)

      // 添加光源
      that.spotLight = new THREE.SpotLight(0xcccccc)
      that.spotLight.position.set(-100, 300, 10)
      scene.add(that.spotLight)

      //
      const textLoad = new FontLoader().load('static/font/Source Han Sans CN Normal_Regular.json', function (font) {
        var txtGeo = new TextGeometry('THREE.JS字体！', {
          font: font,
          size: 0.8, // Float。字体大小，默认值为100
          height: 0.1, // Float。挤出文本的厚度。默认值为50
          curveSegments: 12, // Integer。（表示文本的）曲线上点的数量。默认值为12
          bevelEnabled: true, // Boolean。是否开启斜角，默认为false
          bevelThickness: 0.1, // Float。文本上斜角的深度，默认值为20
          bevelSize: 0.05, // Float。斜角与原始文本轮廓之间的延伸距离。默认值为8
          bevelSegments: 3 // Integer。斜角的分段数。默认值为3
        })
        // var txtMater = new THREE.MeshBasicMaterial({ color: 0x0000ff })
        // var txtMesh = new THREE.Mesh(txtGeo, txtMater)
        var txtMater = new THREE.MeshNormalMaterial({
          flatShading: THREE.FlatShading,
          transparent: true, // transparent开启且为true时才可以调节透明度opacity
          opacity: 0.9
        })
        var txtMesh = new THREE.Mesh(txtGeo, txtMater)
        // txtMesh.position.set(-2, 2.3, -0.4)
        txtMesh.position.set(-2, 10.3, -0.4)
        scene.add(txtMesh)
      })

      // all objects of this animation group share a common animation state
      const animationGroup = new THREE.AnimationObjectGroup()
      // 创建网格;
      const geometry = new THREE.BoxGeometry(5, 5, 5)
      // 给几何体添加纹理,加载贴图图片
      var texture = new THREE.TextureLoader().load('static/texture/waterNormals.jpg', () => {
        this.renderer.render(scene, this.camera)
      })
      const material = new THREE.MeshBasicMaterial({ map: texture, transparent: true })

      const meshCollection = []
      for (let i = 0; i < 5; i++) {
        for (let j = 0; j < 5; j++) {
          var mesh = new THREE.Mesh(geometry, material)

          mesh.position.x = 32 - 16 * i
          mesh.position.y = 0
          mesh.position.z = 32 - 16 * j
          scene.add(mesh)
          animationGroup.add(mesh)
          meshCollection.push(mesh)
        }
      }
      console.log(meshCollection, 'meshCollection')
      const gui = new GUI()
      for (let item = 0; item < meshCollection.length; item++) {
        this.makeXYZGUI(gui, meshCollection[item].position, 'position' + (parseInt(item) + 1))
      }
      // create some keyframe tracks
      // 创建一个x坐标为1,y坐标为0的向量,z坐标为0的向量,no arguments; will be initialised to (0, 0, 0)
      const xAxis = new THREE.Vector3(1, 0, 0)
      // 四元数对象Quaternion使用x、y、z和w四个分量表示,四元数的方法.setFromAxisAngle(axis, angle)通过旋转轴axis和旋转角度angle设置四元数数据，也就是x、y、z和w四个分量,绕单位向量Vector3(x,y,z)表示的轴旋转θ度.Implementation of a quaternion. This is used for rotating things without encountering the dreaded gimbal lock issue, amongst other advantages.|setFromAxisAngle ( axis : Vector3, angle : Float ) : Quaternion,Sets this quaternion from rotation specified by axis and angle.Adapted from the method here.Axis is assumed to be normalized, angle is in radians
      const qInitial = new THREE.Quaternion().setFromAxisAngle(xAxis, 0)
      const qFinal = new THREE.Quaternion().setFromAxisAngle(xAxis, Math.PI)
      // 四元数类型的关键帧轨道,QuaternionKeyframeTrack( name : String - (必须) 关键帧轨道(KeyframeTrack)的标识符, times : Array- (必须) 关键帧的时间数组, values : Array- 与时间数组中的时间点对应的值数组 )
      const quaternionKF = new THREE.QuaternionKeyframeTrack(
        '.quaternion',
        [0, 1, 2],
        [qInitial.x, qInitial.y, qInitial.z, qInitial.w, qFinal.x, qFinal.y, qFinal.z, qFinal.w, qInitial.x, qInitial.y, qInitial.z, qInitial.w]
      )
      // 反应颜色变化的关键帧轨道,ColorKeyframeTrack( name : String - (必须) 关键帧轨道(KeyframeTrack)的标识符, times : Array- (必须) 关键帧的时间数组, values : Array- 与时间数组中的时间点对应的值数组,  interpolation: - 使用的插值类型。 取值参考 Animation Constants. 默认值为 InterpolateLinear.)
      const colorKF = new THREE.ColorKeyframeTrack('.material.color', [0, 1, 2], [1, 0, 0, 0, 1, 0, 0, 0, 1], THREE.InterpolateDiscrete) // [1, 0, 0, 0, 1, 0, 0, 0, 1]颜色值变化过程：红-绿-蓝| THREE.InterpolateDiscrete //离散插值(THREE.InterpolateLinear//线性插值, THREE.InterpolateSmooth//柔和插值)
      // 数字类型的关键帧轨道,NumberKeyframeTrack( name : String- (必须) 关键帧轨道(KeyframeTrack)的标识符, times : Array- (必须) 关键帧的时间数组, values : Array- 与时间数组中的时间点对应的值数组 )
      const opacityKF = new THREE.NumberKeyframeTrack('.material.opacity', [0, 1, 2], [1, 0, 1])

      // 动画剪辑（AnimationClip）是一个可重用的关键帧轨道集，它代表动画,create clip多个帧动画创建一个 剪辑对象 命名为 default,duration为3,AnimationClip( name : String- 此剪辑的名称, duration : Number- 持续时间 (单位秒). 如果传入负数, 持续时间将会从传入的数组中计算得到, tracks : Array- 一个由关键帧轨道（KeyframeTracks）组成的数组 )
      const clip = new THREE.AnimationClip('default', 3, [quaternionKF, colorKF, opacityKF])

      /**
       * 播放编辑好的关键帧数据
       */
      // apply the animation group to the mixer as the root object播放编辑好的动画对象,动画混合器是用于场景中特定对象的动画的播放器。当场景中的多个对象独立动画时，每个对象都可以使用同一个动画混合器
      that.mixer = new THREE.AnimationMixer(animationGroup) // animationGroup作为混合器的参数，可以播放animationGroup中所有子对象的帧动画
      // clipAction (clip : AnimationClip, optionalRoot : Object3D) : AnimationAction,剪辑clip作为参数，通过混合器clipAction方法返回一个操作对象AnimationAction
      const clipAction = that.mixer.clipAction(clip)

      // 可以调节播放速度 默认值为 1
      clipAction.timeScale = 1

      // 开始播放
      clipAction.play()

      //

      that.stats = new Stats()
      that.container.appendChild(that.stats.dom)

      //

      that.clock = new THREE.Clock()

      /**
       * 添加道路流光特效
       */
      textureLine = new THREE.TextureLoader().load('static/texture/line.png')
      textureLine.wrapS = textureLine.wrapT = THREE.RepeatWrapping // // 贴图的重复方式，每个都重复
      textureLine.repeat.set(1, 1) // 纹理在整个表面上重复多少次，在每个方向U和V上。
      textureLine.needsUpdate = true

      const materialine = new THREE.MeshBasicMaterial({
        map: textureLine,
        side: THREE.BackSide,
        transparent: true
      })

      // 创建顶点数组
      const points = [new THREE.Vector3(0, 0, 0), new THREE.Vector3(10, 0, 0), new THREE.Vector3(10, 0, 10), new THREE.Vector3(0, 0, 10)]
      // let points = [new THREE.Vector3(0, 0, 0), new THREE.Vector3(0, 0, 10), new THREE.Vector3(0, 0, 20), new THREE.Vector3(0, 0, 40)]
      // CatmullRomCurve3创建一条平滑的三维样条曲线
      const curve = new THREE.CatmullRomCurve3(points) // 曲线路径

      // 创建管道
      const tubeGeometry = new THREE.TubeGeometry(curve, 80, 0.1)

      const meshLine = new THREE.Mesh(tubeGeometry, materialine)

      scene.add(meshLine)

      //   // 添加热力图
      //   var heatMapGeo = new THREE.PlaneBufferGeometry(1500, 1500)

      //   var heatMapMaterial = new THREE.ShaderMaterial({
      //     transparent: true,
      //     // vertexShader: document.getElementById('vertexshader').textContent,
      //     // fragmentShader: document.getElementById('fragmentshader').textContent,
      //     vertexShader: `varying vec2 vUv;
      // void main() {
      //   vUv = uv;
      //   gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      // }`,
      //     fragmentShader: `#ifdef GL_ES
      // precision highp float;
      // #endif
      // varying vec2 vUv;
      // uniform sampler2D alphaScaleMap;
      // uniform sampler2D paletteMap;
      // void main() {
      //   vec4 alphaColor = texture2D(alphaScaleMap, vUv);
      //   vec4 color = texture2D(paletteMap, vec2(alphaColor.a, 0.0));
      //   gl_FragColor = vec4(color.r, color.g, color.b, alphaColor.a);
      // }`,
      //     uniforms: {
      //       alphaScaleMap: {
      //         type: 't',
      //         value: this.getAlphaScaleMap(w, h)
      //       },
      //       paletteMap: {
      //         type: 't',
      //         value: this.getPaletteMap()
      //       }
      //     }
      //   })

      //   var heatMapPlane = new THREE.Mesh(heatMapGeo, heatMapMaterial)
      //   scene.add(heatMapPlane)

      //
      // 添加辅助线
      var axesHelper = new THREE.AxesHelper(100)
      scene.add(axesHelper)
      // var arrowHelper = new THREE.ArrowHelper( dir, origin, length, hex );
      // scene.add( arrowHelper );

      /* 监听事件,自适应窗口 */
      window.addEventListener('resize', that.onWindowResize, false)
    },

    /* 浏览器窗口变动触发 */
    onWindowResize() {
      var that = this
      // 重新设置相机宽高比例
      that.camera.aspect = window.innerWidth / window.innerHeight
      // 更新相机投影矩阵
      that.camera.updateProjectionMatrix()
      // 重新设置渲染器渲染范围
      that.renderer.setSize(window.innerWidth, window.innerHeight)
    },

    animate() {
      // 线条流动
      if (textureLine) textureLine.offset.x -= 0.01
      requestAnimationFrame(this.animate)
      this.render()
    },

    render() {
      var that = this
      // 数据更新
      const delta = that.clock.getDelta()
      if (that.mixer) {
        that.mixer.update(delta)
      }

      that.renderer.render(scene, that.camera)

      that.stats.update()
    },

    makeXYZGUI(gui, vector3, name) {
      const folder = gui.addFolder(name)
      folder.add(vector3, 'x', -1000, 1000, 0.01)
      folder.add(vector3, 'y', 0, 10, 0.1)
      folder.add(vector3, 'z', -1000, 1000, 0.01)
      folder.open()
    },

    // 随机给出温度值 储存在2维数组
    getTemperature() {
      const temperatureArray = new Array()
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
      const rGradient = context.createRadialGradient(x, y, 0, x, y, radius)
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
      const colorStops = {
        1.0: '#f00',
        0.8: '#e2fa00',
        0.6: '#33f900',
        0.3: '#0349df',
        0.0: '#fff'
      }

      // 颜色条的大小
      const width = 256,
        height = 10
      // 创建canvas
      var paletteCanvas = document.createElement('canvas')
      //   let paletteCanvas = document.getElementById('heatmap')
      paletteCanvas.width = width
      paletteCanvas.height = height
      paletteCanvas.style.position = 'absolute'
      paletteCanvas.style.top = '0'
      paletteCanvas.style.right = '0'
      //   let ctx = paletteCanvas.getContext('2d')
      ctx = paletteCanvas.getContext('2d')
      // 创建线性渐变色
      const linearGradient = ctx.createLinearGradient(0, 0, width, 0)
      for (const key in colorStops) {
        linearGradient.addColorStop(key, colorStops[key])
      }

      // 绘制渐变色条
      ctx.fillStyle = linearGradient
      ctx.fillRect(0, 0, width, height)

      document.body.appendChild(paletteCanvas)

      const paletteTexture = new THREE.Texture(paletteCanvas)
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
      const context = canvas.getContext('2d')

      // 随机生成温度
      const tenperature = this.getTemperature()

      // 绘制透明度阶梯图
      for (let i = 0; i < segments; i++) {
        // 计算出当前温度占标准温度的权值
        const weight = tenperature[i] / 35

        this.drawCircular(context, {
          x: Math.random() * w,
          y: Math.random() * h,
          radius: 80,
          weight: weight
        })
      }

      const tex = new THREE.Texture(canvas)
      tex.minFilter = THREE.NearestFilter
      tex.needsUpdate = true
      return tex
    }
  }
}
</script>

<style scoped>
#info {
  margin: 0 auto;
  height: 100%;
  width: 100%;
  overflow: hidden;
  /* border: 3px solid #2c3e50; */
}
</style>