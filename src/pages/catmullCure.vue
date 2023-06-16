<template>
    <div class="wrap">
      <div ref="threeDom"></div>
      <button class="btn1" @click="house">aaa</button>
  </div>
</template>

<script>
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
  import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';

  import { onMounted, ref } from 'vue';

  export default {
      components: {
      },
      setup() {
          let threeDom = ref(null);

          const SCREEN_WIDTH = window.innerWidth;
          const SCREEN_HEIGHT = window.innerHeight;
          const aspect = SCREEN_WIDTH / SCREEN_HEIGHT;

          let scene = new THREE.Scene();
          let loader = new GLTFLoader();
          let dracoLoader = new DRACOLoader();
          let gltf = null;

          // 辅助观察的坐标系
          let axesHelper = new THREE.AxesHelper(200);
          scene.add(axesHelper);

          // 光源设置
          let directionalLight = new THREE.DirectionalLight('#F7E4D4', 3);
          directionalLight.position.set(1400, 100, -220);
          scene.add(directionalLight);
          // 环境光
          let ambient = new THREE.AmbientLight('#FFD9B8', 1);
          scene.add(ambient);

          // 相机
          let camera = new THREE.PerspectiveCamera(30, aspect, 1, 1000);
          camera.position.set(50, 80, 260);
          camera.lookAt(0, 0, 0);  

          // 渲染器
          let renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
          renderer.setClearColor('#B2A281');  // 设置背景色和透明度
          renderer.setSize(SCREEN_WIDTH, SCREEN_HEIGHT);
          renderer.setPixelRatio(window.devicePixelRatio);
          renderer.shadowMap.enabled = true; // 开启渲染
          renderer.shadowMap.type = THREE.PCFSoftShadowMap;
          renderer.outputEncoding = THREE.sRGBEncoding; //解决加载gltf格式模型颜色偏差问题

          // 设置相机控件轨道控制器OrbitControls
          let controls = new OrbitControls(camera, renderer.domElement);
          controls.update(); // 轨道控制器的更新

          const initialPoints = [
              // { x: 10, y: 10, z: -10 },
              // { x: 10, y: 10, z: 10 },
              { x: -40, y: 0, z: 10 },
              { x: 0, y: 0, z: 0 }
          ];

          const addCube = (pos) => {
              const geometry = new THREE.BoxBufferGeometry(5, 5, 5);
              const material = new THREE.MeshBasicMaterial(0xffffff);
              const cube = new THREE.Mesh(geometry, material);
              cube.position.copy(pos);
              scene.add(cube);
              return cube
          }

          const cubeList = initialPoints.map(pos => addCube(pos));
          const curve = new THREE.CatmullRomCurve3(
              cubeList.map((cube) => cube.position) // 直接绑定方块的position以便后续用方块调整曲线
          );
          curve.curveType = 'chordal'; // 曲线类型
          curve.closed = true; // 曲线是否闭合

          const points = curve.getPoints(100); // 50等分获取曲线点数组
          const line = new THREE.LineLoop(
              new THREE.BufferGeometry().setFromPoints(points),
              new THREE.LineBasicMaterial({ color: 0x00ff00 })
          ); // 绘制实体线条，仅用于示意曲线，后面的向量线条同理，相关代码就省略了

          scene.add(line);

          function changePosition (t) {
              const position = curve.getPointAt(t); // t: 当前点在线条上的位置百分比，后面计算
              gltf.scene.position.copy(position);
          }

          function changeLookAt (t) {
              const tangent = curve.getTangentAt(t);
              const position = curve.getPointAt(t); // t: 当前点在线条上的位置百分比，后面计算
              const lookAtVec = tangent.add(position); // 位置向量和切线向量相加即为所需朝向的点向量
              gltf.scene.lookAt(lookAtVec);
          }

          function animate() {
              const loopTime = 10 * 1000; // loopTime: 循环一圈的时间
              let time = Date.now();
              let t = (time % loopTime) / loopTime; // 计算当前时间进度百分比
              renderer.render( scene, camera );

              changePosition(t);
              changeLookAt(t);

              requestAnimationFrame(animate);
          }

          function loadGltf() {
              // 加载模型并载入动画
              dracoLoader.setDecoderPath('/draco/');
              loader.setDRACOLoader(dracoLoader);
              return new Promise((resolve) => {
                  loader.load('/design/design_house_asset.glb', function (gltf) {
                      resolve(gltf);
                  }, undefined, function (error) {
                      console.error(error);
                  });
              })
          }

          onMounted(async () => {
              gltf = await loadGltf();
              gltf.scene.position.set(0, 0, 0);
              gltf.scene.scale.setScalar(0.1);

              // 添加场景
              scene.add(gltf.scene);
              threeDom.value.appendChild(renderer.domElement);
              animate();
          })

          function house() {
              // gltf.scene.lookAt(-80, 20, 0);
              // gltf.scene.scale.setScalar(0.2);
          }
          return {
              threeDom,
              house
          }
      }
  };
</script>

<style>
.wrap {
  position: relative;
}

.btn1 {
  position: absolute;
  top: 0;
  right: 0;
}
</style>


