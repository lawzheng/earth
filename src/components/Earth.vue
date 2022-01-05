<template>
  <div class="map" ref="mapDom"></div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { onMounted, ref } from "vue";
import envConifg from "../config/earth.config";
import worldGeo from "../assets/geojson/world.geo";
import { countryLine } from '../utils/countryLine';

export default {
  name: "Earth",
  setup() {
    const mapDom = ref(null);
    let renderer = null;
    let camera = null;
    let scene = null;
    let orbitControls = null;
    const textureLoader = new THREE.TextureLoader();
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

    const initEarth = () => {
      const R = envConifg.r;
      worldGeo.features.forEach((country) => {
        if (country.geometry.type === "Polygon") {
          country.geometry.coordinates = [country.geometry.coordinates];
        }
        var line = countryLine(R, country.geometry.coordinates);
        scene.add(line);
      });
    }

    const initBg = () => {
      // 加载星空纹理
      const texture = textureLoader.load(envConifg.bg);
      // 生成球体
      const sphereGeometry = new THREE.SphereGeometry(1000, 50, 50);
      // 调转球体正反
      sphereGeometry.scale(-1, 1, 1);
      // 赋予纹理贴图
      const sphereMaterial = new THREE.MeshBasicMaterial({ map: texture });
      // 生成几何球体
      const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
      // 放入场景内
      scene.add(sphere);
    }

    const initInside = () => {
      const sphereGeometry = new THREE.SphereGeometry(envConifg.r - 1, 50, 50);
      const sphereMaterial = new THREE.MeshBasicMaterial({
        color: 0x000000,
        opacity: 0.9,
        transparent: true,
      });
      const InsideSphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
      scene.add(InsideSphere);
    }

    const initSprite = () => {
      const texture = textureLoader.load(envConifg.yun);
      const spriteMaterial = new THREE.SpriteMaterial({
        map: texture,
        transparent: true,
        opacity: 0.7,
      });
      const sprite = new THREE.Sprite(spriteMaterial);
      sprite.scale.set(envConifg.r * Math.PI, envConifg.r * Math.PI, 1);
      scene.add(sprite);
    }


    const glRender = () => {
      renderer.render(scene, camera);
      requestAnimationFrame(glRender);
    };

    onMounted(() => {
      initTHREE();
      initCamera();
      scene = new THREE.Scene();
      const ambientLight = new THREE.AmbientLight(0xffffff);
      scene.add(ambientLight);
      initOrbitControls();
      initEarth();
      initBg();
      initInside();
      // initSprite();
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
