<template>
  <div class="canvas-container" ref="screenDom"></div>
</template>

<script setup>
import * as THREE from "three";
import { ref, onMounted, onUnmounted } from "vue";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"; // 轨道控制器
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader"; // rgbe加载器 加载hdr图片
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader"; // 加载glb模型加载器
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader"; // 使用了压缩的格式，需要解压缩的文件
import { Reflector } from "three/examples/jsm/objects/Reflector";
let screenDom = ref(null);
onMounted(() => {
  // 创建场景
  let scene = new THREE.Scene();
  // 创建相机
  let camera = new THREE.PerspectiveCamera(
    75, // 视角
    screenDom.value.clientWidth / screenDom.value.clientHeight, // 当前屏幕的宽高比
    0.1, // 能看到的最近距离
    1000 // 最远距离
  );
  camera.position.set(0, 1.5, 6);  // 上调1.5, 远离屏幕6
  // 创建渲染器
  let renderer = new THREE.WebGLRenderer({ antialias: true }); // antialias 抗锯齿效果 更加清晰，没有边缘锯齿感
  renderer.setSize(screenDom.value.clientWidth, screenDom.value.clientHeight); // 设置渲染器的大小
  screenDom.value.appendChild(renderer.domElement); // 画布添加进来
  // 创建辅助坐标轴
  // let axes = new THREE.AxesHelper(5);
  // scene.add(axes);
  // 添加控制器
  let control = new OrbitControls(camera, renderer.domElement);

  // 创建rgbe加载器
  let hdrLoader = new RGBELoader();
  hdrLoader.load("./assets/sky12.hdr", (texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping; // 矩形图，天空是包围住的，映射到球的里边，球的映射，在四周都能看到
    scene.background = texture; // 设置为背景
    scene.environment = texture; // 月亮会发光，反光，设置环境
  });

  // 添加机器人
  // 设置解压缩的加载器
  let dracoLoader = new DRACOLoader(); // 用来解压缩
  dracoLoader.setDecoderPath("./draco/gltf/"); // 专门加压缩文件的配置
  dracoLoader.setDecoderConfig({ type: "js" }); // 设置解压缩配置类型
  let gltfLoader = new GLTFLoader(); // 设置加载器loader
  gltfLoader.setDRACOLoader(dracoLoader); // 加压缩后，在解析gltf、glb文件
  gltfLoader.load("./assets/robot.glb", (gltf) => { // 解析后，加入场景
    scene.add(gltf.scene); // 添加进去，但是发黑
  });
  // 添加直线光
  let light1 = new THREE.DirectionalLight(0xffffff, 0.3);
  light1.position.set(0, 10, 10); // 正面光
  let light2 = new THREE.DirectionalLight(0xffffff, 0.3);
  light1.position.set(0, 10, -10); // 背面光
  let light3 = new THREE.DirectionalLight(0xffffff, 0.8);
  light1.position.set(10, 10, 10); // 斜向下光
  scene.add(light1, light2, light3);

  // 添加底部光阵
  let video = document.createElement("video");
  video.src = "./assets/zp2.mp4";
  video.loop = true; // 循环播放
  video.muted = true; // 静音
  video.play(); // 播放
  let videoTexture = new THREE.VideoTexture(video); // 创建视频纹理
  const videoGeoPlane = new THREE.PlaneBufferGeometry(8, 4.5); // 视频的平面的形状大小，宽屏比例
  const videoMaterial = new THREE.MeshBasicMaterial({ // 材质
    map: videoTexture, // 映射材料样式
    transparent: true, // 透明
    side: THREE.DoubleSide, // 两面都可以看到
    alphaMap: videoTexture, // 透明纹理
  });
  const videoMesh = new THREE.Mesh(videoGeoPlane, videoMaterial); // mesh网格化
  videoMesh.position.set(0, 0.2, 0); // 设置位置
  videoMesh.rotation.set(-Math.PI / 2, 0, 0); // 沿着x轴旋转 -90度
  scene.add(videoMesh); // 加入场景

  // 添加镜面反射
  let reflectorGeometry = new THREE.PlaneBufferGeometry(100, 100);
  let reflectorPlane = new Reflector(reflectorGeometry, {
    textureWidth: window.innerWidth,
    textureHeight: window.innerHeight,
    color: 0x332222,
  });
  reflectorPlane.rotation.x = -Math.PI / 2;
  scene.add(reflectorPlane);

  function render() {
    requestAnimationFrame(render);
    renderer.render(scene, camera);
  }
  render();

  // 监听画面变化，更新渲染画面
  window.addEventListener("resize", () => {
    //   console.log("画面变化了");
    // 更新摄像头
    camera.aspect = window.innerWidth / window.innerHeight;
    //   更新摄像机的投影矩阵
    camera.updateProjectionMatrix();

    //   更新渲染器
    renderer.setSize(window.innerWidth, window.innerHeight);
    //   设置渲染器的像素比
    renderer.setPixelRatio(window.devicePixelRatio);
  });
});
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
.canvas-container {
  width: 100vw;
  height: 100vh;
}
</style>
