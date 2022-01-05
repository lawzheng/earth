<template>
  <div class="map" ref="mapDom"></div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { onMounted, ref } from "vue";
import envConifg from "../config/earth.config";

export default {
  name: "EarthByPicture",
  setup() {
    const mapDom = ref(null);
    let renderer = null;
    let camera = null;
    let scene = null;
    let orbitControls = null;
    const initTHREE = () => {
      renderer = new THREE.WebGLRenderer({
        antialias: true,
      });
      renderer.setSize(mapDom.value.clientWidth, mapDom.value.clientHeight);
      renderer.setClearColor(0xffffff);
      mapDom.value.appendChild(renderer.domElement);
    };

    const initCamera = () => {
      camera = new THREE.PerspectiveCamera(
        45,
        mapDom.value.clientWidth / mapDom.value.clientHeight,
        1,
        2000
      );
      camera.position.z = 300;
      camera.up.set(0, 1, 0);
      camera.lookAt(0, 0, 0);
    };

    const initOrbitControls = () => {
      const os = new OrbitControls(camera, renderer.domElement);
      os.target = new THREE.Vector3(0, 0, 0); //控制焦点
      os.autoRotate = false; //将自动旋转关闭
      os.enablePan = false; // 不禁止鼠标平移, 可以用键盘来平移
      os.maxDistance = 1000; // 最大外移动
      os.minDistance = 100; // 向内最小外移动
      orbitControls = os;
    };

    const initBg = () => {
      // 把背景图加载过来当做纹理。
      const texture = new THREE.TextureLoader().load(envConifg.earthBg);
      // 这个绘制球体
      const geometry = new THREE.SphereGeometry(envConifg.r, 50, 50);
      // 放入纹理
      const material = new THREE.MeshLambertMaterial({
        map: texture,
      });
      const mesh = new THREE.Mesh(geometry, material);
      scene.add(mesh);
    };

    const glRender = () => {
      renderer.render(scene, camera);
      requestAnimationFrame(glRender);
    };

    onMounted(() => {
      initTHREE();
      initCamera();
      scene = new THREE.Scene();
      scene.add(new THREE.AxisHelper(4));
      const ambientLight = new THREE.AmbientLight(0xffffff);
      scene.add(ambientLight);
      initOrbitControls();
      initBg();
      glRender();
    });
    return {
      mapDom,
    };
  },
};
</script>

<style scoped>
.map {
  width: 100%;
  height: 100%;
}
</style>
