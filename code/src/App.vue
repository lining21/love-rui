<template>
  <div
    class="scenes"
    style="
      position: fixed;
      left: 0;
      top: 0;
      z-index: 10;
      pointer-events: none;
      transition: all 1s;
    "
    :style="{
      transform: `translate3d(0, ${-index * 100}vh, 0)`,
    }"
  >
  <!-- 每一行数据是 100vh高度，通过动态的index，将每一屏切换上来 -->
    <div v-for="item in scenes" style="width: 100vw; height: 100vh">
      <h1 style="padding: 100px 50px; font-size: 20px; color: #fff">
        {{ item.text }}
      </h1>
      <h1 v-if="item.tip" style="text-align: left; color: #ffeb3b; margin: 0px 0px -200px 60px;">
        <!-- {{ tipText }} -->
      </h1>
    </div>
  </div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";
import { Water } from "three/examples/jsm/objects/Water2";
import gsap from "gsap";
import { ref } from "vue";

const isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
let timer = null;

// 初始化场景
const scene = new THREE.Scene();
// 初始化相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
if (isMobile) {
  camera.position.set(-3.23, 10.98, 30.06);
} else {
  camera.position.set(-3.23, 2.98, 4.06);
}
//  更新投影矩阵，设置了宽高比的就设置投影矩阵
camera.updateProjectionMatrix();
// 初始化渲染器
const renderer = new THREE.WebGLRenderer({
  // 设置抗锯齿
  antialias: true,
});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
// 设置色调映射
renderer.outputEncoding = THREE.sRGBEncoding; // 色调编码
renderer.toneMapping = THREE.ACESFilmicToneMapping; // 类似电影的调光效果
renderer.toneMappingExposure = 0.5; // 色调映射的曝光程度
renderer.shadowMap.enabled = true; // 渲染器允许阴影投射
renderer.physicallyCorrectLights = true; // 按照物理的光照效果，整体会暗下来 默认情况下，three.js的光强数值不真实。为了使得光强更趋于真实值
// 设置水面效果

// 初始化控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.target.set(-8, 2, 0); // 控制器 的 target，初始化第一个
controls.enableDamping = true; // 阻尼效果

// 初始化loader
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("./draco/");
const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(dracoLoader);

// 加载环境纹理 天空
let rgbeLoader = new RGBELoader();
rgbeLoader.load("./textures/sky.hdr", (texture) => {
  // 如何将图像应用到对象。默认为 UV贴图（THREE.UVMapping）类型，这里U,V 坐标用来应用映射，要求是单个纹理。
  texture.mapping = THREE.EquirectangularReflectionMapping; // 圆柱反射映射
  scene.background = texture;
  scene.environment = texture;
});

// 加载模型 小房子和山
gltfLoader.load("./model/scene.glb", (gltf) => {
  const model = gltf.scene;
  // 遍历所有元素
  model.traverse((child) => {
    if (child.name === "Plane") {
      child.visible = false;
    }
    // 如果是物体的话
    if (child.isMesh) {
      // 设置物体投射阴影投射
      child.castShadow = true;
      // 设置物体接收阴影
      child.receiveShadow = true;
    }
  });
  scene.add(model);
});

// 创建水面 
const waterGeometry = new THREE.CircleGeometry(300, 32);
const water = new Water(waterGeometry, {
  textureWidth: 1024, // 宽
  textureHeight: 1024, // 高
  color: 0xeeeeff, // 水面颜色
  flowDirection: new THREE.Vector2(1, 1), // 水流方向
  scale: 100, // 比较远地方看得见
});
water.rotation.x = -Math.PI / 2; // 竖着的，旋转90度
water.position.y = -0.4; // 水面高低，降一点，别淹了房子
scene.add(water);

// 添加平行光
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(0, 50, 0);
scene.add(light);

// 加入锐和我
// 添加平面
let palneTexture = new THREE.TextureLoader().load("./images/ruime9.jpg");
const planeGeometry = new THREE.PlaneGeometry(10, 14);
const planeMaterial = new THREE.MeshBasicMaterial({
  map: palneTexture
});
const plane = new THREE.Mesh(planeGeometry, planeMaterial);
plane.position.y = 6;
plane.position.x = 10;

let palneTexture2 = new THREE.TextureLoader().load("./images/ruime6.jpg");
const planeGeometry2 = new THREE.PlaneGeometry(5, 5);
const planeMaterial2 = new THREE.MeshBasicMaterial({
  // color: 0xffffff,
  map: palneTexture2
});
const plane2 = new THREE.Mesh(planeGeometry2, planeMaterial2);
plane2.position.y = 2;
plane2.position.z = 8;
plane2.rotation.y = Math.PI/2;

let palneTexture3 = new THREE.TextureLoader().load("./images/ruime2.jpg");
const planeGeometry3 = new THREE.PlaneGeometry(8, 8);
const planeMaterial3 = new THREE.MeshBasicMaterial({
  // color: 0xffffff,
  map: palneTexture3
});
const plane3 = new THREE.Mesh(planeGeometry3, planeMaterial3);
plane3.position.y = -14;
plane3.position.x = 0;

// 视频纹理
const video = document.createElement("video");
video.src = "./images/ruime8.mp4";
// video.src = "rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mov"
video.loop = true;

// const video = document.createElement("img");
// video.dynsrc = "./images/ruime5.avi";
// video.loop = true;

let palneTexture4 = new THREE.TextureLoader().load("./images/ruime7.jpg");
const planeGeometry4 = new THREE.PlaneGeometry(10, 12);
const planeMaterial4 = new THREE.MeshBasicMaterial({
  // color: 0xffffff,
  map: palneTexture4
});
const plane4 = new THREE.Mesh(planeGeometry4, planeMaterial4);
plane4.position.y = 2;
plane4.position.z = 16;
plane4.rotation.y = Math.PI + Math.PI / 2;
const tipText = ref('单击有惊喜呦！')

const clickEvent = (e) => {
  if (index.value === scenes.length - 1) {
    if (video.paused) {
      tipText.value = "单击暂停播放呦！"
      video.play();
      let texture = new THREE.VideoTexture(video);
      planeMaterial4.map = texture;
      planeMaterial4.map.needsUpdate = true;
    } else {
      tipText.value = "单击有惊喜呦！"
      video.pause()
    }
  }
}

window.addEventListener("click", (e) => {
  // 当鼠标移动的时候播放视频
  //   判断视频是否处于播放状态
  console.log('index.value', index.value, scenes.length)
  timer && clearTimeout(timer);
  timer = setTimeout(clickEvent, 500);
});



// 添加点光源 房子中安装灯泡，在房子中
const pointLight = new THREE.PointLight(0xffffff, 50);
pointLight.position.set(0.1, 2.4, 0);
pointLight.castShadow = true; // 点光源需要投射阴影 注意渲染器要允许投射阴影 renderer.shadowMap.enabled = true;
scene.add(pointLight);

// 创建点光源组 圣诞树围绕
const pointLightGroup = new THREE.Group();
pointLightGroup.position.set(-8, 2.5, -1.5); // 整个组 从房子中移出来
let radius = 3; // 球体围绕旋转的半径
let pointLightArr = [];
for (let i = 0; i < 3; i++) {
  // 创建球体当灯泡
  const sphereGeometry = new THREE.SphereGeometry(0.2, 32, 32); // 创建球的半径是 0.2
  const sphereMaterial = new THREE.MeshStandardMaterial({ // 球的材质 原来是 MeshBasicMaterial 不够亮，改为MeshStandardMaterial标准材质
    color: 0xffffff,// 白色
    emissive: 0xffffff, // 设置为白色
    emissiveIntensity: 10, // 强度是比较大的强度
  });
  const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial); // 球体网格组合，球体+材质
  pointLightArr.push(sphere);
  sphere.position.set(
    radius * Math.cos((i * 2 * Math.PI) / 3), // X轴 半径乘以圆对应的角度 就是x值
    Math.cos((i * 2 * Math.PI) / 3), // Y轴 上下波动，角度动
    radius * Math.sin((i * 2 * Math.PI) / 3) // Z轴的值
  );

  let pointLight = new THREE.PointLight(0xffffff, 50); // 添加点光源
  sphere.add(pointLight); // 给当前球体添加点光源

  pointLightGroup.add(sphere); // 将当前球 放入点光源组中，主要是为了位置统一调配
}
scene.add(pointLightGroup); // 场景加入点光源组

// 使用补间函数，从0到2π，使灯泡旋转
let options = {
  angle: 0,
};
gsap.to(options, {
  angle: Math.PI * 2, // 从0到2π
  duration: 10, // 十秒循环一次
  repeat: -1, // 无限循环
  ease: "linear", // 线性的
  onUpdate: () => { // 每次更新
    pointLightGroup.rotation.y = options.angle; // 动起来，但是只是转圈，不会上下
    pointLightArr.forEach((item, index) => {
      item.position.set(  // 动画效果可以再思考一下
        radius * Math.cos((index * 2 * Math.PI) / 3),
        Math.cos((index * 2 * Math.PI) / 3 + options.angle * 5),
        radius * Math.sin((index * 2 * Math.PI) / 3)
      );
    });
  },
});
function render() {
  requestAnimationFrame(render); // 请求动画帧
  renderer.render(scene, camera); // 渲染器，渲染场景和相机
  controls.update(); // 控制器更新
}
render();

// 使用补间动画移动相机
let timeLine1 = gsap.timeline(); // 看向什么位置
let timeline2 = gsap.timeline(); // 去哪里

// 定义相机移动函数
function translateCamera(position, target) {
  timeLine1.to(camera.position, { // 相机的位置
    x: position.x,
    y: position.y,
    z: position.z,
    duration: 1,
    ease: "power2.inOut",
  });

  timeline2.to(controls.target, { // 控制器的目标，聚焦到那个点 相机目标
    x: target.x,
    y: target.y,
    z: target.z,
    duration: 1,
    ease: "power2.inOut",
  });
}

// 创建一个动画数组
let scenes = [
  {
    text: "老婆锐!（双击屏幕）",
    callback: () => {
      scene.remove(plane4);
      // 执行函数切换位置
      translateCamera(
        new THREE.Vector3(-3.23, 3, 4.06), // 相机去到的位置
        new THREE.Vector3(-8, 2, 0) // 相机目标 -8是圣诞树的位置
      );
    }
  },
  {
    text: "今天是2024年11月30日,我们在一起的第九个纪念日以及萄萄的生日！",
    callback: () => {
      scene.add(plane);
      // 执行函数切
      if(isMobile) {
        translateCamera(new THREE.Vector3(10, 0, 40), new THREE.Vector3(0, 0, 0)); // 看小木屋
      } else {
        translateCamera(new THREE.Vector3(7, 0, 23), new THREE.Vector3(0, 0, 0)); // 看小木屋
      }
      
    },
  },
  {
    text: "感谢在这么大的世界里遇见了你",
    callback: () => {
      scene.remove(plane);
      scene.add(plane2);
      // 执行函数切
      if(isMobile) {
        translateCamera(new THREE.Vector3(10, 3, 10), new THREE.Vector3(5, 2, 8));
      } else {
        translateCamera(new THREE.Vector3(10, 3, 0), new THREE.Vector3(5, 2, 0));
      }
    },
  },
  {
    text: "纪念日快乐，让我们一起祝沐凡生日快乐！",
    callback: () => {
      scene.remove(plane2);
      scene.add(plane3);
      // 执行函数切
      translateCamera(new THREE.Vector3(7, 0, 23), new THREE.Vector3(0, 0, 0));
      makeHeart();
    },
  },
  {
    text: "愿我们永远健康快乐的在一起(单击有惊喜！)",
    callback: () => {
      scene.remove(plane3);
      scene.add(plane4);
      // 执行函数切
      if(isMobile) {
        translateCamera(
          new THREE.Vector3(-20, 1.3, 16.6),
          new THREE.Vector3(5, 2, 15)
        );
      } else {
        translateCamera(
          new THREE.Vector3(-20, 1.3, 6.6),
          new THREE.Vector3(5, 2, 0)
        );
      }
    },
    tip: true
  },
];

let index = ref(0);
let isAnimate = false; // 防止滑动过快，没有切换完成，间隔
const dblclickEvent = (e) => {
  if (isAnimate) return;
  isAnimate = true;
  // if (e.deltaY > 0) {
  //   index.value++;
  //   if (index.value > scenes.length - 1) {
  //     index.value = 0;
  //     restoreHeart();
  //   }
  // }
  console.log(e)
  if (e) {
    index.value++;
    if (index.value > scenes.length - 1) {
      index.value = 0;
      restoreHeart();
    }
  }
  scenes[index.value].callback();
  setTimeout(() => {
    isAnimate = false;  // 间隔一秒，防止滚动多次
  }, 1000);
}
// 监听鼠标滚轮事件
window.addEventListener(
  "dblclick",
  (e) => {
    timer && clearTimeout(timer);
    timer = setTimeout(dblclickEvent(e), 500);
  },
  false
);

// 实例化创建漫天星星
// 必须渲染大量具有相同几何形状和材质但具有不同世界变换的对象，请使用InstancedMesh。InstancedMesh的使用将帮助您减少绘图调用的次数，从而提高应用程序的整体渲染性能。
let starsInstance = new THREE.InstancedMesh(
  new THREE.SphereGeometry(0.1, 32, 32), // 球形体
  new THREE.MeshStandardMaterial({ // 材质
    color: 0xffffff,
    emissive: 0xffffff,
    emissiveIntensity: 10,
  }),
  100 // 数量
);

// 星星随机到天上 模型顶部的星星
let starsArr = [];  // 开始的星星位置，心形 
let endArr = []; // 结束的时候位置

for (let i = 0; i < 100; i++) {
  let x = Math.random() * 100 - 50;
  let y = Math.random() * 100 - 50;
  let z = Math.random() * 100 - 50;
  starsArr.push(new THREE.Vector3(x, y, z));

  let matrix = new THREE.Matrix4();  // 位移、缩放、旋转 都可以通过矩阵来解决
  matrix.setPosition(x, y, z);
  starsInstance.setMatrixAt(i, matrix); // 100个星星，对应i传递给他矩阵
}
scene.add(starsInstance);

// 创建爱心路径 使用贝塞尔曲线 canvas课程有讲到
let heartShape = new THREE.Shape(); // 创建曲线
heartShape.moveTo(25, 25);
heartShape.bezierCurveTo(25, 25, 20, 0, 0, 0);
heartShape.bezierCurveTo(-30, 0, -30, 35, -30, 35);
heartShape.bezierCurveTo(-30, 55, -10, 77, 25, 95);
heartShape.bezierCurveTo(60, 77, 80, 55, 80, 35);
heartShape.bezierCurveTo(80, 35, 80, 0, 50, 0);
heartShape.bezierCurveTo(35, 0, 25, 25, 25, 25);

// 根据爱心路径获取点
let center = new THREE.Vector3(0, 2, 10); // 设置中心点，各个点往中心点靠近
for (let i = 0; i < 100; i++) {
  let point = heartShape.getPoint(i / 100); // 0就是第一点，100就是最后一个点, 获取曲线上的点
  // endArr.push(new THREE.Vector3(point.x, point.y, point.z)) // 保存好点 但是心太大了
  endArr.push(
    new THREE.Vector3( // 记住要传向量
      point.x * 0.1 + center.x, // 往中心点的x轴靠近
      point.y * 0.1 + center.y, // 往中心点的y轴靠近
      center.z
    )
  );
}

// 创建爱心动画 拿着位置进行生成 补间函数（补齐时间范围内的函数）
function makeHeart() {
  let params = {
    time: 0,
  };

  gsap.to(params, {
    time: 1, // 时间是1
    duration: 1, // 1秒完成
    onUpdate: () => {
      for (let i = 0; i < 100; i++) {
        // 每一个都是从结束点减去开始点，乘以当前进度，加上开始的点，慢慢就到达end点
        let x = starsArr[i].x + (endArr[i].x - starsArr[i].x) * params.time;
        let y = starsArr[i].y + (endArr[i].y - starsArr[i].y) * params.time;
        let z = starsArr[i].z + (endArr[i].z - starsArr[i].z) * params.time;
        let matrix = new THREE.Matrix4();
        matrix.setPosition(x, y, z); // 通过xyz得到 矩阵
        starsInstance.setMatrixAt(i, matrix); // 矩阵给设置到星星上
      }
      starsInstance.instanceMatrix.needsUpdate = true; // 实例的矩阵需要更新
    },
  });
}
// 星星四散开来 星星复原
function restoreHeart() {
  let params = {
    time: 0,
  };

  gsap.to(params, {
    time: 1,
    duration: 1,
    onUpdate: () => {
      for (let i = 0; i < 100; i++) {
        // 每一个都是从开始点减去结束点，乘以当前进度，加上开始的点，慢慢就到达start点
        let x = endArr[i].x + (starsArr[i].x - endArr[i].x) * params.time;
        let y = endArr[i].y + (starsArr[i].y - endArr[i].y) * params.time;
        let z = endArr[i].z + (starsArr[i].z - endArr[i].z) * params.time;
        let matrix = new THREE.Matrix4();
        matrix.setPosition(x, y, z);
        starsInstance.setMatrixAt(i, matrix);
      }
      starsInstance.instanceMatrix.needsUpdate = true;
    },
  });
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
canvas {
  width: 100vw;
  height: 100vh;
  position: fixed;
  left: 0;
  top: 0;
}
</style>
