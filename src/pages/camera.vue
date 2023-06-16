<template>
    <div ref="threeDom"></div>
</template>

<script>
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
  import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';

  import { onBeforeMount, onMounted, ref, render } from 'vue';

  export default {
      components: {
      },
      setup() {
          let threeDom = ref(null);

          let SCREEN_WIDTH = window.innerWidth;
          let SCREEN_HEIGHT = window.innerHeight;
          let aspect = SCREEN_WIDTH / SCREEN_HEIGHT;
          const frustumSize = 600;

          let mixers = [];
          let scene = new THREE.Scene();
          let clock = new THREE.Clock();
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

          //光源辅助线
          // let directionalLightHelper = new THREE.DirectionalLightHelper(directionalLight);
          // scene.add(directionalLightHelper);

          // 环境光
          let ambient = new THREE.AmbientLight('#FFD9B8', 1);
          scene.add(ambient);

          // 相机
          let camera = new THREE.PerspectiveCamera(30, aspect, 1, 1000);
          camera.position.set(50, 80, 260);
          camera.lookAt(0, 0, 0);   

          let cameraPerspective = new THREE.PerspectiveCamera(30, 0.5 * aspect, 1, 3000 );
          cameraPerspective.rotation.y = Math.PI;
          scene.add( cameraPerspective );

          
          // let cameraPerspectiveHelper = new THREE.CameraHelper( cameraPerspective );
          // scene.add( cameraPerspectiveHelper );
          
          // 
          // let cameraOrtho = new THREE.OrthographicCamera( 0.5 * frustumSize * aspect / - 2, 0.5 * frustumSize * aspect / 2, frustumSize / 2, frustumSize / - 2, 150, 1000 );
          // cameraOrtho.rotation.y = Math.PI;
          // scene.add( cameraOrtho );

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

          function animate() {
              let mixerUpdateDelta = clock.getDelta();
              mixers.forEach((mixer) => mixer.update(mixerUpdateDelta));
              
              requestAnimationFrame(animate);
              threeDom.value.appendChild(renderer.domElement);
              render();
          }

          function render() {
              const r = Date.now() * 0.0005;
              const mesh = gltf.scene;

              mesh.position.x = 10 * Math.cos( r );
              mesh.position.z = 10 * Math.sin( r );
              mesh.position.y = 10 * Math.sin( r );

              mesh.children[0].position.x = 70 * Math.cos( 2 * r );
              mesh.children[0].position.z = 70 * Math.sin( r );

              // cameraOrtho.far = mesh.position.length();
              // cameraOrtho.updateProjectionMatrix();
              cameraPerspective.fov = 300 * Math.sin( 0.5 * r );
              cameraPerspective.far = mesh.position.length();
              cameraPerspective.updateProjectionMatrix();

              camera.lookAt( mesh.position );

              renderer.clear();
              renderer.render( scene, cameraPerspective );
              renderer.render( scene, camera );
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
              // gltf.scene.position.set(0, 0, 0);
              gltf.scene.scale.setScalar(0.1);
              console.log(gltf)
              // 添加场景
              scene.add(gltf.scene);
              animate();
          })

          return {
              threeDom
          }
      }
  };
</script>

