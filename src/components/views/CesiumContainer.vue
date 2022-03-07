<template>
  <div id="cesiumContainer" class="fullSize"></div>
</template>

<script>
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
export default {
  name: 'CesiumContainer',
  data() {
    return {}
  },
  mounted() {
    /* eslint no-new: */
    this.initViewer()
    //new Viewer('cesiumContainer')
  },
  methods: {
    initViewer() {
      // 天地图影像
      var imageryProvider1 = new WebMapTileServiceImageryProvider({
        url: 'http://t0.tianditu.com/img_w/wmts?service=wmts&request=GetTile&version=1.0.0&LAYER=img&tileMatrixSet=w&TileMatrix={TileMatrix}&TileRow={TileRow}&TileCol={TileCol}&style=default&format=tiles&tk=你的token',
        layer: 'tdtBasicLayer',
        style: 'default',
        format: 'image/jpeg'
      })
      // 注记
      var imageryProvider2 = new WebMapTileServiceImageryProvider({
        url: 'http://t0.tianditu.com/cia_w/wmts?service=wmts&request=GetTile&version=1.0.0&LAYER=cia&tileMatrixSet=w&TileMatrix={TileMatrix}&TileRow={TileRow}&TileCol={TileCol}&style=default.jpg&tk=你的token',
        layer: 'tdtAnnoLayer',
        style: 'default',
        format: 'image/jpeg'
      })
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
        requestWaterMask: true //请求水波纹效果
      })
      const viewer = new Viewer('cesiumContainer', {
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

      //用于渲染星空的SkyBox对象
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
      // viewer.scene.globe.baseColor = Color.BLACK
      viewer.scene.globe.baseColor = new Color(1, 1, 1, 0) //修改地球颜色
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
          //heading:Cesium.Math.toRadians(90,0),
          // 视角
          //pitch:Cesium.Math.toRadians(-90),
          //roll:0.0
        }
      })
      viewer.scene.globe.depthTestAgainstTerrain = true
      const handler = new ScreenSpaceEventHandler(viewer.canvas)
      this.$store.state.cesiumDrawHandler = handler
      window.cesiumViewer = viewer
    }
  }
}
</script>

<style scoped>
#cesiumContainer {
  width: 100%;
  height: 100%;
}
</style>
