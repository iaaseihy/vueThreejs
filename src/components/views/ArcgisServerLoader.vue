<template>
  <!-- <canvas id="heatmap"></canvas> -->
  <div id="server"></div>
</template>

<script>
// import ArcGisMapServerImageryProviderExt from ''
import ArcGisMapServerImageryProviderExt from '../commonJS/ArcGisMapServerImageryProviderExt.min.js'
import 'cesium/Build/Cesium/Widgets/widgets.css'
import 'cesium/Build/Cesium/Widgets/bucket.css'
import {
  Viewer,
  Cartesian3,
  Color,
  SkyBox,
  ScreenSpaceEventHandler,
  UrlTemplateImageryProvider,
  WebMercatorTilingScheme,
  CesiumTerrainProvider,
  createOsmBuildings,
  WebMapTileServiceImageryProvider
} from 'cesium'
import { LOCAL_IMG_URL, LOCAL_TERRAIN_URL, GAODE_IMG_URL } from '../commonJS/config'
import positiveX from '../../assets/img/SkyBox/00h+00.jpg'
import negativeX from '../../assets/img/SkyBox/12h+00.jpg'
import positiveY from '../../assets/img/SkyBox/06h+00.jpg'
import negativeY from '../../assets/img/SkyBox/18h+00.jpg'
import positiveZ from '../../assets/img/SkyBox/06h+90.jpg'
import negativeZ from '../../assets/img/SkyBox/06h-90.jpg'
var viewer
export default {
  name: 'ArcGisServerLoader',
  data() {
    return {
      centeropt: {
        x: -2779232.6714578,
        y: 4696589.160926553,
        z: 3292956.1794273183,
        heading: 0.2767835731761714,
        pitch: -0.4650234919552627,
        roll: 0.0007965708034545216
      },
      item: {
        url: '',
        key: ''
      }
    }
  },
  mounted() {
    this.initViewer()
  },
  methods: {
    initViewer() {
      // 天地图影像
      // 本地影像
      var imageryProvider = new UrlTemplateImageryProvider({
        url: LOCAL_IMG_URL,
        // url: GAODE_IMG_URL,
        tilingScheme: new WebMercatorTilingScheme(),
        fileExtension: 'png',
        minimumLevel: 0,
        maximumLevel: 19
      })
      var terrainProvider = new CesiumTerrainProvider({
        url: LOCAL_TERRAIN_URL,
        requestWaterMask: true // 请求水波纹效果
      })
      viewer = new Viewer('server', {
        terrainExaggeration: 0.95,
        imageryProvider: imageryProvider,
        // terrainProvider: terrainProvider,
        baseLayerPicker: false,
        fullscreenButton: false,
        geocoder: false,
        homeButton: false,
        navigationHelpButton: false,
        sceneModePicker: false,
        timeline: false,
        animation: false,
        selectionIndicator: false,
        infoBox: false
      })

      // 用于渲染星空的SkyBox对象
      viewer.scene.skyBox = new SkyBox({
        sources: {
          positiveX: positiveX,
          negativeX: negativeX,
          positiveY: positiveY,
          negativeY: negativeY,
          positiveZ: positiveZ,
          negativeZ: negativeZ
        }
      })
      viewer._cesiumWidget._creditContainer.style.display = 'none'
      viewer.scene.globe.showGroundAtmosphere = false
      // viewer.scene.debugShowFramesPerSecond = true // 显示帧率
      // viewer.scene.globe.baseColor = Color.BLACK
      viewer.scene.globe.baseColor = new Color(1, 1, 1, 0) // 修改地球颜色
      // viewer.scene.primitives.add(createOsmBuildings())
      // viewer.scene.camera.flyTo({
      //   destination: Cartesian3.fromDegrees(-74.019, 40.6912, 750)
      // })

      viewer.camera.setView({
        // Cesium的坐标是以地心为原点，一向指向南美洲，一向指向亚洲，一向指向北极州
        // fromDegrees()方法，将经纬度和高程转换为世界坐标
        destination: Cartesian3.fromDegrees(99, 36.4, 18000008),
        orientation: {
          // 指向
          // heading:Cesium.Math.toRadians(90,0),
          // 视角
          // pitch:Cesium.Math.toRadians(-90),
          // roll:0.0
        }
      })
      viewer.scene.globe.depthTestAgainstTerrain = true
      const handler = new ScreenSpaceEventHandler(viewer.canvas)
      this.$store.state.cesiumDrawHandler = handler
      window.cesiumViewer = viewer
    },
    addSZ2000(item) {
      const { storedData } = this.state
      return new Promise((resolve, reject) => {
        var provider = new ArcGisMapServerImageryProviderExt({
          url: item.url
        })
        const imageryLayer = viewer.imageryLayers.addImageryProvider(provider)

        this.setState(
          {
            storedData: {
              ...storedData,
              [item.key]: imageryLayer
            }
          },
          () => {
            resolve()
          }
        )
      })
    }
  }
}
</script>

<style scoped>
#server {
  width: 100%;
  height: 100%;
}
</style>