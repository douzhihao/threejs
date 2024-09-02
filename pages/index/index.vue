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
	import {Capsule} from 'three/examples/jsm/math/Capsule';
	import {Octree} from 'three/examples/jsm/math/Octree';

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
	let birthPoint;

	let roleModel;
	let roleBox;

	let roleAction;
	let roleIdle;
	let roleRun;
	let roleSit;
	
	let role_originPoint= new THREE.Vector3();
	// 碰撞 胶囊
	let roleCapsule = new Capsule(
	    new THREE.Vector3(0, 0.35*150, 0),
	    new THREE.Vector3(0, 1.55*150, 0),
	    0.35*150,
	);
	let roleCapsuleCenter = new THREE.Vector3();
	let roleCapsuleV3 = new THREE.Vector3();
	// 碰撞体
	let octree_ground = new Octree();
	let octree_wall = new Octree();
	
	let speed = 0.07;
	
	let key_left, key_right, key_front, key_back;

	let lastTime = 0;
	
	const raycaster = new THREE.Raycaster();
	const dir = new THREE.Vector3();
	
	function getAverage(arr) {
	    if (arr.length === 0) return 0; // 如果数组为空，返回0
	    const sum = arr.reduce((acc, current) => acc + current, 0); // 累加数组元素
	    return sum / arr.length; // 计算平均值
	}
	export default {
		data() {
			return {

			}
		},
		created() {
			self = this;
			window.addEventListener("resize", this.throttle(this.resizeWindow));
			self.keyLinstenerKeyDown();
			self.keyLinstenerKeyUp();

		},
		onLoad() {

		},
		mounted() {
			self = this;
			self.startLoad();
		},
		methods: {
			keyLinstenerKeyDown() {
				//监听键盘按钮
				document.onkeydown = function(event) {
					var e = event || window.event;
					var keyCode = e.keyCode || e.which;
					switch (keyCode) {
						case 87: //W
							roleModel.rotation.y = Math.PI + controls.getAzimuthalAngle();
							key_front = true;
							key_left = false;
							key_back = false;
							key_right = false;
							break;
						case 65: //A
							roleModel.rotation.y = -Math.PI / 2 + controls.getAzimuthalAngle();
							key_left = true;
							key_front = false;
							key_back = false;
							key_right = false;
							break;
						case 83: //S
							roleModel.rotation.y = Math.PI*2 + controls.getAzimuthalAngle();
							key_back = true;
							key_front = false;
							key_left = false;
							key_right = false;
							break;
						case 68: //D
							roleModel.rotation.y = Math.PI / 2 + controls.getAzimuthalAngle();
							key_right = true;
							key_front = false;
							key_left = false;
							key_back = false;
							break;
					}
					
					roleIdle.stop();
					roleRun.play();
				}
			},
			keyLinstenerKeyUp() {
				//监听键盘按钮
				document.onkeyup = function(event) {
					var e = event || window.event;
					var keyCode = e.keyCode || e.which;
					// Male_action.stop();
					switch (keyCode) {
						case 87: //W
							key_front = false;
							break;
						case 65: //A
							key_left = false;
							break;
						case 83: //S
							key_back = false;
							break;
						case 68: //D
							key_right = false;
							break;
					}
					var allfalse = !key_front && !key_left && !key_back && !key_right;
					if(allfalse) {
						roleIdle.play();
						roleRun.stop();
					}
				}
			},
			
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

				const fov = 40;
				const aspect = container.clientWidth / container.clientHeight;
				const near = 1;
				const far = 3000;
				//camera setup
				// camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
				camera = new THREE.PerspectiveCamera(fov, aspect);
				camera.position.set(0, 1.5, 5)
				camera.lookAt(0, 100, 0);
				// // light
				// light = new THREE.DirectionalLight(0xffffff, 2.5);
				// light.position.set(10, 10, 10);
				// scene.add(light);
				// const light1 = new THREE.PointLight( 0xff0000, 1, 100 );
				// light1.position.set( 10, 10, 10 );
				// scene.add( light1 );

				//renderer
				renderer = new THREE.WebGLRenderer();
				renderer.setSize(container.clientWidth, container.clientHeight);
				renderer.setPixelRatio(window.devicePixelRatio);
				container.appendChild(renderer.domElement);
				
				// TextureLoader
				TextureLoader = new THREE.TextureLoader();
				// console.log(TextureLoader);
				// 加载一个资源
				TextureLoader.load(
					// 'https://mob.hexntc.com/web0902/static/hdr/texture.jpg',
					'./static/hdr/texture.jpg',
					// onLoad回调
					function(texture) {
						// in this example we create the material when the texture is loaded
						texture.mapping = THREE.EquirectangularReflectionMapping;
						// scene.background = texture; // 给场景添加背景图
						scene.environment = texture;
					}
				);
				// 加载一个资源
				TextureLoader.load(
					// 资源URL
					// 'https://mob.hexntc.com/web0902/static/hdr/meadow_2_1k.jpg',
					'./static/hdr/meadow_2_1k.jpg',
					// onLoad回调
					function(texture) {
						// in this example we create the material when the texture is loaded
						texture.mapping = THREE.EquirectangularReflectionMapping;
						scene.background = texture; // 给场景添加背景图
						// scene.environment = texture;
					}
				);

				// controls
				controls = new OrbitControls(camera, renderer.domElement);
				// 是否放大缩小
				// controls.enableZoom = false;
				// controls.enablePan = false;
				controls.maxDistance = 5;
				controls.minDistance = 1;
				controls.minPolarAngle = Math.PI * (45 / 180);
				controls.maxPolarAngle = Math.PI * (100 / 180);
				controls.target.set(0, 1.5, 0);
				
				controls.addEventListener( 'change', ()=>{
					dir.copy(controls.target).sub(controls.object.position);
					raycaster.far = dir.length();
					raycaster.set(controls.object.position, dir.normalize());
					const intersections = raycaster.intersectObjects( sceneModel.children );
					const intersection = ( intersections.length ) > 0 ? intersections[ 0 ] : null;
					if(intersection && intersection) {
						controls.object.position.copy(intersection.point);
					}
				} );
				
				self.loadScene();
				self.animate();
			},
			loadScene() {
				loader = new GLTFLoader();
				loader.load(
				// 'https://mob.hexntc.com/web0902/static/model/classjs2.glb',
				'./static/model/classjs2.glb',
				 function(node) {
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
						// 查找出生点
						  if (node.name === 'birth01') {
							birthPoint = node;
							birthPoint.position.x=-4.5
							controls.target.copy(birthPoint.position);
							controls.target.y += 1.5
							controls.object.position.set(-4.5, 2.6, -4.75);
							controls.update();
						  }
					});
					node.scenes[0].children[2].intensity = 0.8;
					self.loadRole();
					
				})
			},
			loadRole() {
				loader.load(
				// 'https://mob.hexntc.com/web0902/static/model/role.glb',
				'./static/model/role.glb',
				 function(node) {
					// console.log(node)
					scene.add(node.scene);
					roleModel = node.scene;
					roleAction = new THREE.AnimationMixer(node.scene);
					roleIdle = roleAction.clipAction(node.animations[0]);
					roleRun = roleAction.clipAction(node.animations[1]);
					roleSit = roleAction.clipAction(node.animations[2]);
					roleIdle.play();
					// node.scene.rotateY(Math.PI);
					// 设置人物模型初始位置为出生点
					  if (birthPoint) {
						  // console.log(birthPoint)
						roleModel.position.copy(birthPoint.position);
						// controls.position(birthPoint.position)
					  } else {
						console.warn('未找到出生点！');
						roleModel.position.set(0, 0, 0);
					  }
					// 创建碰撞盒
					// roleBox = new THREE.Box3().setFromObject(roleModel);
					
				})
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
				renderer.render(scene, camera);
				
				let radian_cos;
				let radian_sin;
				if (roleModel) {
					 radian_cos = Math.cos(roleModel.rotation.y);
					 radian_sin = Math.sin(roleModel.rotation.y);
					 // capsule
					 roleCapsule.getCenter(roleCapsuleCenter);
					 roleCapsuleV3.subVectors(role_originPoint, roleCapsuleCenter);
					 roleCapsuleV3.setY(roleCapsuleV3.y + 0.95*150);
					 roleCapsule.translate(roleCapsuleV3);
					 
					 roleModel.updateMatrixWorld();
					 roleModel.getWorldPosition(role_originPoint);
				}
				let radian_angle = controls.getAzimuthalAngle() * 180 / Math.PI;


				if (key_front) {
					camera.position.z += (speed * radian_cos);
					controls.target.z += (speed * radian_cos);
					camera.position.x += (speed * radian_sin);
					controls.target.x += (speed * radian_sin);
					roleModel.position.z += (speed * radian_cos);
					roleModel.position.x += (speed * radian_sin);
				}
				if (key_back) {
					camera.position.z += (speed * radian_cos);
					controls.target.z += (speed * radian_cos);
					camera.position.x += (speed * radian_sin);
					controls.target.x += (speed * radian_sin);
					roleModel.position.z += (speed * radian_cos);
					roleModel.position.x += (speed * radian_sin);
				}
				if (key_left) {
					camera.position.x += (speed * radian_sin);
					controls.target.x += (speed * radian_sin);
					camera.position.z += (speed * radian_cos);
					controls.target.z += (speed * radian_cos);
					roleModel.position.x += (speed * radian_sin);
					roleModel.position.z += (speed * radian_cos);
				}
				if (key_right) {
					camera.position.x += (speed * radian_sin);
					controls.target.x += (speed * radian_sin);
					camera.position.z += (speed * radian_cos);
					controls.target.z += (speed * radian_cos);
					roleModel.position.x += (speed * radian_sin);
					roleModel.position.z += (speed * radian_cos);
				}
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