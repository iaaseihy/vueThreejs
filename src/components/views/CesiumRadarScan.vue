<template>
  <div>
    <div id="cesiumContainer" class="fullSize"></div>
    <div id="creditContainer" style="display: none"></div>
    <div id="menu">
      <p>
        <button @click="setvisible('position')">定位</button>
        <button @click="setvisible('add')">添加</button>
        <button @click="setvisible('del')">删除</button>
      </p>
    </div>
  </div>
</template>

<script>
import * as Cesium from 'cesium'
import '../commonJS/1-2ImageryProvider-WebExtend'
import { LOCAL_IMG_URL } from '../commonJS/config'
import 'cesium/Build/Cesium/Widgets/widgets.css'
// import * as Cesium from '../../cesium/Cesium1.48/Cesium/1-2ImageryProvider-WebExtend.js'
// import * as Cesium from '../../cesium/Cesium1.48/Cesium/Widgets/widgets.css'
var viewer, scene // scene,viewer 用全局变量，不放属性data里面，避免由于vue的双向绑定机制劫持数据导致数据量大的时候页面卡顿
export default {
  name: 'CesiumRadarScan',
  data() {
    return {
      // viewer: null,
      lon: 120.34448164198324, // -74.01296152309055;
      lat: 31.16295978144941, // 40.70524201566827
      // scene: null,
      camera: null,
      renderer: null,
      trackballControls: null,
      clock: null,
      textureLoader: null,
      loadedFont: null,
      //盆栽
      PottedPlant: null,
      PottedPlant1: null
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      var that = this
      //   var GoogleMap = ImageryProviderWebExtendTool.createGoogleMapsByUrl(Cesium, { url: 'http://mt1.google.cn/vt/lyrs=s&hl=zh-CN&x={x}&y={y}&z={z}' })
      var imageryProvider = new Cesium.UrlTemplateImageryProvider({
        url: LOCAL_IMG_URL,
        tilingScheme: new Cesium.WebMercatorTilingScheme(),
        fileExtension: 'png',
        minimumLevel: 0,
        maximumLevel: 19
      })

      // // 加载arcgis数据——ArcGisMapServerImageryProvider
      // var imageryProviderArcGIS = new Cesium.ArcGisMapServerImageryProvider({
      //   url: 'http://atlasmaps.esri.com/arcgis/rest/services/Esri/USA_Population_Density/MapServer',
      //   enablePickFeatures: false
      // })
      viewer = new Cesium.Viewer('cesiumContainer', {
        imageryProvider: imageryProvider,
        creditContainer: 'creditContainer',
        selectionIndicator: true,
        animation: false,
        baseLayerPicker: false,
        geocoder: false,
        timeline: false,
        sceneModePicker: true,
        navigationHelpButton: false,
        infoBox: true,
        fullscreenButton: true,
        isAdd: false
      })
      viewer.camera.setView({
        destination: Cesium.Cartesian3.fromDegrees(that.lon, that.lat, 18000008)
      })
      //   var lat = 40.70524201566827 // 42.006;
      //   var lon = -74.01296152309055 //128.055;
      viewer.scene.globe.depthTestAgainstTerrain = true
      viewer.scene.debugShowFramesPerSecond = true // 显示帧率
      // 取消双击事件
      viewer.cesiumWidget.screenSpaceEventHandler.removeInputAction(Cesium.ScreenSpaceEventType.LEFT_DOUBLE_CLICK)
      // 监听地图移动完成事件
      // viewer.camera.moveEnd.addEventListener(this.onMoveendMap)
      viewer.camera.moveEnd.addEventListener(this.getCurrentExtent)
    },
    // 获取当前相机视角内的图幅范围
    getCurrentExtent() {
      // 范围对象
      var extent = {}

      // 得到当前三维场景
      var scene = viewer.scene

      // 得到当前三维场景的椭球体
      var ellipsoid = scene.globe.ellipsoid
      var canvas = scene.canvas

      // canvas左上角
      var car3_lt = viewer.camera.pickEllipsoid(new Cesium.Cartesian2(0, 0), ellipsoid)

      // canvas右下角
      var car3_rb = viewer.camera.pickEllipsoid(new Cesium.Cartesian2(canvas.width, canvas.height), ellipsoid)

      // 当canvas左上角和右下角全部在椭球体上
      if (car3_lt && car3_rb) {
        var carto_lt = ellipsoid.cartesianToCartographic(car3_lt)
        var carto_rb = ellipsoid.cartesianToCartographic(car3_rb)
        extent.xmin = Cesium.Math.toDegrees(carto_lt.longitude)
        extent.ymax = Cesium.Math.toDegrees(carto_lt.latitude)
        extent.xmax = Cesium.Math.toDegrees(carto_rb.longitude)
        extent.ymin = Cesium.Math.toDegrees(carto_rb.latitude)
      }

      // 当canvas左上角不在但右下角在椭球体上
      else if (!car3_lt && car3_rb) {
        var car3_lt2 = null
        var yIndex = 0
        do {
          // 这里每次10像素递加，一是10像素相差不大，二是为了提高程序运行效率
          yIndex <= canvas.height ? (yIndex += 10) : canvas.height
          car3_lt2 = viewer.camera.pickEllipsoid(new Cesium.Cartesian2(0, yIndex), ellipsoid)
        } while (!car3_lt2)
        var carto_lt2 = ellipsoid.cartesianToCartographic(car3_lt2)
        var carto_rb2 = ellipsoid.cartesianToCartographic(car3_rb)
        extent.xmin = Cesium.Math.toDegrees(carto_lt2.longitude)
        extent.ymax = Cesium.Math.toDegrees(carto_lt2.latitude)
        extent.xmax = Cesium.Math.toDegrees(carto_rb2.longitude)
        extent.ymin = Cesium.Math.toDegrees(carto_rb2.latitude)
      }

      // 获取高度
      extent.height = Math.ceil(viewer.camera.positionCartographic.height)
      // 获取当前缩放的地图层级
      let level = 0
      if (viewer.scene.globe._surface._tilesToRender.length) {
        level = viewer.scene.globe._surface._tilesToRender[0].level
        console.log('当前地图层级=======', level)
      }
      console.log('地图变化监听事件', extent, (extent.xmin + extent.xmax) / 2, (extent.ymax + extent.ymin) / 2)
    },
    /*
          添加雷达扫描线 地形遮挡开启   lon:-74.01296152309055 lat:40.70524201566827 height:129.14366696393927
          viewer
          cartographicCenter 扫描中心
          radius  半径 米
          scanColor 扫描颜色
          duration 持续时间 毫秒
        */
    AddRadarScanPostStage(viewer, cartographicCenter, radius, scanColor, duration) {
      var ScanSegmentShader =
        'uniform sampler2D colorTexture;\n' +
        'uniform sampler2D depthTexture;\n' +
        'varying vec2 v_textureCoordinates;\n' +
        'uniform vec4 u_scanCenterEC;\n' +
        'uniform vec3 u_scanPlaneNormalEC;\n' +
        'uniform vec3 u_scanLineNormalEC;\n' +
        'uniform float u_radius;\n' +
        'uniform vec4 u_scanColor;\n' +
        'vec4 toEye(in vec2 uv, in float depth)\n' +
        ' {\n' +
        ' vec2 xy = vec2((uv.x * 2.0 - 1.0),(uv.y * 2.0 - 1.0));\n' +
        ' vec4 posInCamera =czm_inverseProjection * vec4(xy, depth, 1.0);\n' +
        ' posInCamera =posInCamera / posInCamera.w;\n' +
        ' return posInCamera;\n' +
        ' }\n' +
        'bool isPointOnLineRight(in vec3 ptOnLine, in vec3 lineNormal, in vec3 testPt)\n' +
        '{\n' +
        'vec3 v01 = testPt - ptOnLine;\n' +
        'normalize(v01);\n' +
        'vec3 temp = cross(v01, lineNormal);\n' +
        'float d = dot(temp, u_scanPlaneNormalEC);\n' +
        'return d > 0.5;\n' +
        '}\n' +
        'vec3 pointProjectOnPlane(in vec3 planeNormal, in vec3 planeOrigin, in vec3 point)\n' +
        '{\n' +
        'vec3 v01 = point -planeOrigin;\n' +
        'float d = dot(planeNormal, v01) ;\n' +
        'return (point - planeNormal * d);\n' +
        '}\n' +
        'float distancePointToLine(in vec3 ptOnLine, in vec3 lineNormal, in vec3 testPt)\n' +
        '{\n' +
        'vec3 tempPt = pointProjectOnPlane(lineNormal, ptOnLine, testPt);\n' +
        'return length(tempPt - ptOnLine);\n' +
        '}\n' +
        'float getDepth(in vec4 depth)\n' +
        '{\n' +
        'float z_window = czm_unpackDepth(depth);\n' +
        'z_window = czm_reverseLogDepth(z_window);\n' +
        'float n_range = czm_depthRange.near;\n' +
        'float f_range = czm_depthRange.far;\n' +
        'return (2.0 * z_window - n_range - f_range) / (f_range - n_range);\n' +
        '}\n' +
        'void main()\n' +
        '{\n' +
        'gl_FragColor = texture2D(colorTexture, v_textureCoordinates);\n' +
        'float depth = getDepth( texture2D(depthTexture, v_textureCoordinates));\n' +
        'vec4 viewPos = toEye(v_textureCoordinates, depth);\n' +
        'vec3 prjOnPlane = pointProjectOnPlane(u_scanPlaneNormalEC.xyz, u_scanCenterEC.xyz, viewPos.xyz);\n' +
        'float dis = length(prjOnPlane.xyz - u_scanCenterEC.xyz);\n' +
        'float twou_radius = u_radius * 2.0;\n' +
        'if(dis < u_radius)\n' +
        '{\n' +
        'float f0 = 1.0 -abs(u_radius - dis) / u_radius;\n' +
        'f0 = pow(f0, 64.0);\n' +
        'vec3 lineEndPt = vec3(u_scanCenterEC.xyz) + u_scanLineNormalEC * u_radius;\n' +
        'float f = 0.0;\n' +
        'if(isPointOnLineRight(u_scanCenterEC.xyz, u_scanLineNormalEC.xyz, prjOnPlane.xyz))\n' +
        '{\n' +
        'float dis1= length(prjOnPlane.xyz - lineEndPt);\n' +
        'f = abs(twou_radius -dis1) / twou_radius;\n' +
        'f = pow(f, 3.0);\n' +
        '}\n' +
        'gl_FragColor = mix(gl_FragColor, u_scanColor, f + f0);\n' +
        '}\n' +
        '}\n'

      var _Cartesian3Center = Cesium.Cartographic.toCartesian(cartographicCenter)
      var _Cartesian4Center = new Cesium.Cartesian4(_Cartesian3Center.x, _Cartesian3Center.y, _Cartesian3Center.z, 1)

      var _CartographicCenter1 = new Cesium.Cartographic(cartographicCenter.longitude, cartographicCenter.latitude, cartographicCenter.height + 500)
      var _Cartesian3Center1 = Cesium.Cartographic.toCartesian(_CartographicCenter1)
      var _Cartesian4Center1 = new Cesium.Cartesian4(_Cartesian3Center1.x, _Cartesian3Center1.y, _Cartesian3Center1.z, 1)

      var _CartographicCenter2 = new Cesium.Cartographic(cartographicCenter.longitude + Cesium.Math.toRadians(0.001), cartographicCenter.latitude, cartographicCenter.height)
      var _Cartesian3Center2 = Cesium.Cartographic.toCartesian(_CartographicCenter2)
      var _Cartesian4Center2 = new Cesium.Cartesian4(_Cartesian3Center2.x, _Cartesian3Center2.y, _Cartesian3Center2.z, 1)
      var _RotateQ = new Cesium.Quaternion()
      var _RotateM = new Cesium.Matrix3()

      var _time = new Date().getTime()

      var _scratchCartesian4Center = new Cesium.Cartesian4()
      var _scratchCartesian4Center1 = new Cesium.Cartesian4()
      var _scratchCartesian4Center2 = new Cesium.Cartesian4()
      var _scratchCartesian3Normal = new Cesium.Cartesian3()
      var _scratchCartesian3Normal1 = new Cesium.Cartesian3()

      var ScanPostStage = new Cesium.PostProcessStage({
        fragmentShader: ScanSegmentShader,
        uniforms: {
          u_scanCenterEC: function () {
            return Cesium.Matrix4.multiplyByVector(viewer.camera._viewMatrix, _Cartesian4Center, _scratchCartesian4Center)
          },
          u_scanPlaneNormalEC: function () {
            var temp = Cesium.Matrix4.multiplyByVector(viewer.camera._viewMatrix, _Cartesian4Center, _scratchCartesian4Center)
            var temp1 = Cesium.Matrix4.multiplyByVector(viewer.camera._viewMatrix, _Cartesian4Center1, _scratchCartesian4Center1)
            _scratchCartesian3Normal.x = temp1.x - temp.x
            _scratchCartesian3Normal.y = temp1.y - temp.y
            _scratchCartesian3Normal.z = temp1.z - temp.z

            Cesium.Cartesian3.normalize(_scratchCartesian3Normal, _scratchCartesian3Normal)
            return _scratchCartesian3Normal
          },
          u_radius: radius,
          u_scanLineNormalEC: function () {
            var temp = Cesium.Matrix4.multiplyByVector(viewer.camera._viewMatrix, _Cartesian4Center, _scratchCartesian4Center)
            var temp1 = Cesium.Matrix4.multiplyByVector(viewer.camera._viewMatrix, _Cartesian4Center1, _scratchCartesian4Center1)
            var temp2 = Cesium.Matrix4.multiplyByVector(viewer.camera._viewMatrix, _Cartesian4Center2, _scratchCartesian4Center2)

            _scratchCartesian3Normal.x = temp1.x - temp.x
            _scratchCartesian3Normal.y = temp1.y - temp.y
            _scratchCartesian3Normal.z = temp1.z - temp.z

            Cesium.Cartesian3.normalize(_scratchCartesian3Normal, _scratchCartesian3Normal)

            _scratchCartesian3Normal1.x = temp2.x - temp.x
            _scratchCartesian3Normal1.y = temp2.y - temp.y
            _scratchCartesian3Normal1.z = temp2.z - temp.z

            var tempTime = ((new Date().getTime() - _time) % duration) / duration
            Cesium.Quaternion.fromAxisAngle(_scratchCartesian3Normal, tempTime * Cesium.Math.PI * 2, _RotateQ)
            Cesium.Matrix3.fromQuaternion(_RotateQ, _RotateM)
            Cesium.Matrix3.multiplyByVector(_RotateM, _scratchCartesian3Normal1, _scratchCartesian3Normal1)
            Cesium.Cartesian3.normalize(_scratchCartesian3Normal1, _scratchCartesian3Normal1)
            return _scratchCartesian3Normal1
          },
          u_scanColor: scanColor
        }
      })

      viewer.scene.postProcessStages.add(ScanPostStage)
    },
    // var isAdd = false;
    setvisible(value) {
      var that = this
      switch (value) {
        case 'position':
          viewer.camera.setView({
            destination: Cesium.Cartesian3.fromDegrees(that.lon, that.lat, 10000)
          })
          break
        case 'add':
          if (!that.isAdd) {
            viewer.entities.add({
              position: Cesium.Cartesian3.fromDegrees(that.lon, that.lat),
              point: {
                pixelSize: 10,
                color: Cesium.Color.RED
              }
            })
            var CartographicCenter = new Cesium.Cartographic(Cesium.Math.toRadians(that.lon), Cesium.Math.toRadians(that.lat), 0)
            var scanColor = new Cesium.Color(1.0, 0.0, 0.0, 1)
            this.AddRadarScanPostStage(viewer, CartographicCenter, 1000, scanColor, 4000)
            that.isAdd = true
          }
          break
        case 'del':
          if (that.isAdd) {
            viewer.scene.postProcessStages.removeAll()
            that.isAdd = false
          }
          break
      }
    }
  }
}
</script>

<style scoped>
#cesiumContainer {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
}
#menu {
  position: absolute;
  top: 80px;
  left: 10px;
  z-index: 999;
}
</style>
