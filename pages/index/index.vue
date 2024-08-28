<template>
	<view class="content">
		<view id='panoParent' class="full"></view>
	</view>
</template>

<script>
	// import Constant from '@/static/constant.js';
	import * as THREE from 'three';
	import {
		GLTFLoader
	} from 'three/examples/jsm/loaders/GLTFLoader.js';
	import {
		OrbitControls
	} from "three/examples/jsm/controls/OrbitControls";

	let self;
	let container;

	let scene;
	let camera;
	let renderer;
	let composer;

	let controls;
	let light;

	let loader;
	let TextureLoader;

	let sceneModel;

	let roleModel;
	let roleBox;

	let roleAction;
	let roleIdle;
	let roleRun;
	let roleSit;

	let lastTime = 0;

	let moveSpeed = 0.01;
	let direction = new THREE.Vector3(0, 0, 0);

	const raycaster = new THREE.Raycaster();
	const mouse = new THREE.Vector2();

	export default {
		data() {
			return {

			}
		},
		created() {
			self = this;
			window.addEventListener("resize", this.throttle(this.resizeWindow));
			document.addEventListener('keydown', function(event) {
				// 键盘控制
					const speed = 0.1;
					switch (event.key) {
						case 'w':
							roleModel.position.z -= speed;
							break;
						case 's':
							roleModel.position.z += speed;
							break;
						case 'a':
							roleModel.position.x -= speed;
							break;
						case 'd':
							roleModel.position.x += speed;
							break;
					}

					// 更新碰撞盒
					roleBox.setFromObject(roleModel);

					// 碰撞检测
					sceneModel.children.forEach((child) => {
						if (child !== roleModel && child.type === 'Mesh') {
							const childBox = new THREE.Box3().setFromObject(child);
							if (roleBox.intersectsBox(childBox)) {
								// 碰撞处理
								console.log('碰撞！');
								// 例如，将人物模型位置回退到碰撞前的状态
								roleModel.position.set(roleModel.position.x - speed, roleModel.position.y,
								roleModel.position.z - speed);
							}
						}
					});
			});

		},
		onLoad() {

		},
		mounted() {
			self = this;
			self.startLoad();
		},
		methods: {
			startLoad() {
				var parent = document.getElementById('panoParent');
				var div = document.createElement('div');
				div.id = 'sceneMain';
				div.style.width = '100vw';
				div.style.height = '100vh';
				div.style.backgroundColor = "rgba(0,0,0,0)";
				parent.appendChild(div);

				self.createPano();
			},
			createPano() {
				container = document.querySelector("#sceneMain");
				// console.log(container);
				//create scene
				scene = new THREE.Scene();

				const fov = 30;
				const aspect = container.clientWidth / container.clientHeight;
				const near = 1;
				const far = 3000;
				//camera setup
				// camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
				camera = new THREE.PerspectiveCamera(fov, aspect);
				camera.position.set(0, 1.5, 5)
				camera.lookAt(0, 100, 0);
				// light
				light = new THREE.DirectionalLight(0xffffff, 2.5);
				light.position.set(10, 10, 10);
				scene.add(light);
				// const light1 = new THREE.PointLight( 0xff0000, 1, 100 );
				// light1.position.set( 10, 10, 10 );
				// scene.add( light1 );

				//renderer
				renderer = new THREE.WebGLRenderer();

				renderer.setSize(container.clientWidth, container.clientHeight);
				renderer.setPixelRatio(window.devicePixelRatio);
				container.appendChild(renderer.domElement);

				// controls
				controls = new OrbitControls(camera, renderer.domElement);
				// 是否放大缩小
				// controls.enableZoom = false;
				controls.enablePan = false;
				// controls.maxDistance = 5;
				// controls.minDistance = 3;
				// controls.minPolarAngle = Math.PI * (45 / 180);
				// controls.maxPolarAngle = Math.PI * (100 / 180);
				console.log(controls);
				controls.target.set(0, 1.5, 0);

				self.loadScene();
				self.loadRole();

				self.animate();

			},
			loadScene() {
				loader = new GLTFLoader();
				// loader.load('/static/model/zhanhuijs.glb',function(node){
				loader.load('/static/model/classjs2.glb', function(node) {
					// loader.load('/static/model/yanhuiting.glb',function(node){
					// console.log(node)
					scene.add(node.scene);
					sceneModel = node.scene;
					const Box001 = node.scene.getObjectByName('wall01');
					Box001.visible = false;

					// 将场景中的所有对象设置为双面
					scene.traverse(function(node) {
						// 检查节点是否是具有材质的网格（Mesh）
						if (node.isMesh) {
							// 遍历所有的材质
							node.material = node.material.clone(); // 克隆材质以保留原始设置
							node.material.side = THREE.DoubleSide; // 设置为双面
						}
					});


					// TextureLoader
					TextureLoader = new THREE.TextureLoader();
					// console.log(TextureLoader);
					// 加载一个资源
					TextureLoader.load(
						// 资源URL
						'/static/hdr/DU_World_SKYBALL.jpg',
						// https://h5test.ophyer.cn/techg2022/static/model/0822/HDR/GHallBG.jpg
						// onLoad回调
						function(texture) {
							// in this example we create the material when the texture is loaded
							texture.mapping = THREE.EquirectangularReflectionMapping;
							// scene.background = texture; // 给场景添加背景图
							scene.environment = texture;
						}
					);

				})
			},
			loadRole() {
				loader.load('/static/model/role.glb', function(node) {
					// loader.load('/static/model/glb1.glb',function(node){
					// console.log(node)
					scene.add(node.scene);
					roleModel = node.scene;
					roleAction = new THREE.AnimationMixer(node.scene);
					roleIdle = roleAction.clipAction(node.animations[0]);
					roleRun = roleAction.clipAction(node.animations[1]);
					roleSit = roleAction.clipAction(node.animations[2]);
					roleIdle.play();
					// node.scene.rotateY(Math.PI);
					node.scene.position.set(0, 0, 0);
					// 创建碰撞盒
					roleBox = new THREE.Box3().setFromObject(roleModel);
				})
			},
			checkCollision() {
				// 将鼠标位置转换为向量
				mouse.x = (window.innerWidth / 2) / window.innerWidth * 2 - 1;
				mouse.y = -(window.innerHeight / 2) / window.innerHeight * 2 + 1;

				// 更新射线投射器的射线方向
				raycaster.setFromCamera(mouse, camera);

				// 计算与场景中对象的交点
				const intersects = raycaster.intersectObjects(scene.children);

				if (intersects.length > 0) {
					// 处理碰撞
					console.log('Collision detected');
				}
			},
			resizeWindow() {
				camera.aspect = container.clientWidth / container.clientHeight;
				camera.updateProjectionMatrix();
				renderer.setSize(container.clientWidth, container.clientHeight);
			},
			throttle(fn, wait = 200) {
				let timer = null;
				return function() {
					if (timer === null) {
						timer = setTimeout(() => {

							fn.apply(this);
							timer = null;
						}, wait);
					}
				};
			},
			animate() {
				requestAnimationFrame(self.animate);
				// 获取当前时间
				let now = performance.now();
				// 计算时间差
				let deltaTime = now - lastTime;
				lastTime = now;
				if (roleAction) {
					roleAction.update(deltaTime / 1000);
				}
				// if(roleModel){
				// 	// 更新相机和轨道控制器的跟随
				// 	camera.position.copy(roleModel.position)
				// 	controls.target.copy(roleModel.position);
				// 	controls.update();
				// }

				renderer.render(scene, camera);
			},
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
</style>