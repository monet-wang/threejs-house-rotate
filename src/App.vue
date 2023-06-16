<template>
	<div ref="threeDom"></div>
</template>

<script>
	import * as TWEEN from '@tweenjs/tween.js'
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
			controls.target = new THREE.Vector3(0, 0, 0)

			let raycaster = new THREE.Raycaster();
			let mouse = new THREE.Vector2();
			let intersects = null;

			renderer.domElement.addEventListener('dblclick', function (event) {
				mouse.x = (event.clientX / SCREEN_WIDTH) * 2 - 1;
				mouse.y = -(event.clientY / SCREEN_HEIGHT) * 2 + 1;
				raycaster.setFromCamera(mouse, camera);
				intersects = raycaster.intersectObject(gltf.scene, true);
				if (intersects.length > 0) {
					let distance = 100
					let angel = Math.PI / 5 // TODO: 计算角度

					let position = {
						x: intersects[0].point.x + Math.cos(angel) * distance,
						y: intersects[0].point.y + 10,
						z: intersects[0].point.z + Math.sin(angel) * distance
					}

					let tween = new TWEEN.Tween(camera.position).to(position, 2000)
					let tween1 = new TWEEN.Tween(controls.target).to(intersects[0].point, 2000)

					controls.enabled = false;
					tween.onComplete(function () {
						controls.enabled = true;
					})
					tween.start()
					tween1.start()
				}
			});

			function animate() {
				requestAnimationFrame(animate);
				TWEEN.update();
        		controls.update();
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
				gltf.scene.position.set(0, 0, 0);
				gltf.scene.scale.setScalar(0.1);
				camera.lookAt(gltf.scene.position)

				// 添加场景
				scene.add(gltf.scene);

				threeDom.value.appendChild(renderer.domElement);
				animate();
			})

			return {
				threeDom
			}
		}
	};
</script>


