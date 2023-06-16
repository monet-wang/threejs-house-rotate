<template>
    <div ref="threeDom"></div>
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
          let mixer = null;
          let animateAction = null;
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

          let curve = new THREE.CubicBezierCurve3(
              new THREE.Vector3(80, 70, -300),
              new THREE.Vector3(-50, 60, 70),
              new THREE.Vector3(-20, 10, 100),
              new THREE.Vector3(0, 0, 0)
          );
          let points = curve.getPoints(100);
          let duration = points.length;
          let times = new Float32Array(Array.from({ length: duration }, (v, i) => i));
          let posArr = [];
          points.forEach(ele => {
              posArr.push(ele.x, ele.y, ele.z)
          })
          let values = new Float32Array(posArr);
          let posTrack = new THREE.KeyframeTrack('.position', times, values);
          let clip = new THREE.AnimationClip('default', duration, [posTrack]);
          let clock = new THREE.Clock();

          function animate() {
              requestAnimationFrame(animate);
              threeDom.value.appendChild(renderer.domElement);
              renderer.render( scene, camera );

              mixer.update(clock.getDelta());
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

              mixer = new THREE.AnimationMixer(gltf.scene);
              animateAction = mixer.clipAction(clip);
              animateAction.timeScale = 110;
              animateAction.loop = THREE.LoopOnce;
              animateAction.clampWhenFinished = true;
              animateAction.play();
              animate();
          })

          return {
              threeDom
          }
      }
  };
</script>

