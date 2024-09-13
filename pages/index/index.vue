<template>
	<view class="content">
		<view id='panoParent' class="full"></view>
		<div id="joycon" ref="joycon" v-show="isShowNipple"></div>
	</view>
</template>

<script>
	// import Constant from '@/static/constant.js';
	import nipplejs from 'nipplejs';
	import * as THREE from 'three';
	import {
		GLTFLoader
	} from 'three/examples/jsm/loaders/GLTFLoader.js';
	import {
		OrbitControls
	} from "three/examples/jsm/controls/OrbitControls";
	import {Capsule} from 'three/examples/jsm/math/Capsule';
	import { Octree } from 'three/examples/jsm/math/Octree';
	import { OctreeHelper } from 'three/examples/jsm/helpers/OctreeHelper';

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
	let roleWalk;
	
	let role_originPoint= new THREE.Vector3();
	
	// 碰撞 胶囊
	let roleCapsule = new Capsule(
	    new THREE.Vector3(0, 0.5, 0),
	    new THREE.Vector3(0, 1.5, 0),
	    0.3,
	);
	let roleCapsuleCenter = new THREE.Vector3();
	let roleCapsuleV3 = new THREE.Vector3();
	// 碰撞体
	let worldOctree = new Octree();
	let octree_ground = new Octree();
	let octree_wall = new Octree();
	
	
	let speed = 0.03;
	
	let roleVelocity = new THREE.Vector3();
	let gravity = new THREE.Vector3(0, -9.8, 0); // 引力
	
	let key_left, key_right, key_front, key_back;

	let lastTime = 0;
	
	let raycaster = new THREE.Raycaster();
	let dir = new THREE.Vector3();
	let particles= null;
	let _count= 0;
	const SEPARATION = 100, AMOUNTX = 50, AMOUNTY = 30;
	
	function getAverage(arr) {
	    if (arr.length === 0) return 0; // 如果数组为空，返回0
	    const sum = arr.reduce((acc, current) => acc + current, 0); // 累加数组元素
	    return sum / arr.length; // 计算平均值
	}
	export default {
		data() {
			return {
				isLoading:true,
				isShowNipple:false,
				roleId:''
			}
		},
		created() {
			self = this;
			window.addEventListener("resize", self.throttle(self.resizeWindow));
			self.keyLinstenerKeyDown();
			self.keyLinstenerKeyUp();

		},
		onLoad(option) {
			// console.log(option);
			this.roleId = option.roleID;
		},
		mounted() {
			self = this;
			self.startLoad();
			var myElement = document.getElementById('panoParent');
			myElement.addEventListener('contextmenu', function(event) {
			  event.preventDefault();
			  event.stopPropagation();
			}, false);
			
			
			// console.log(nipplejs);
			// import('nipplejs').then((nipplejs) => {
				// console.log(nipplejs);
				const joystick = nipplejs.create({
					zone: document.getElementById('joycon'),
					mode: 'static',
					size: 100,
					dynamicPage: true,
					color: 'white', // 操纵杆的颜色
					position: {
						left: '50%',
						top: '50%',
					},
				});
				joystick.on('move', function(evt, data) {
					if (data.direction) {
						// console.log("start move", data.direction.angle);
						switch (data.direction.angle) {
							case "up": //W
								roleModel.rotation.y = Math.PI + controls.getAzimuthalAngle();
								key_front = true;
								key_left = false;
								key_back = false;
								key_right = false;
								break;
							case "left": //A
								roleModel.rotation.y = -Math.PI / 2 + controls.getAzimuthalAngle();
								key_left = true;
								key_front = false;
								key_back = false;
								key_right = false;
								break;
							case "down": //S
								roleModel.rotation.y = Math.PI * 2 + controls.getAzimuthalAngle();
								key_back = true;
								key_front = false;
								key_left = false;
								key_right = false;
								break;
							case "right": //D
								roleModel.rotation.y = Math.PI / 2 + controls.getAzimuthalAngle();
								key_right = true;
								key_front = false;
								key_left = false;
								key_back = false;
								break;
						}
					}
					roleIdle.stop();
					roleRun.play();
				});
				joystick.on('start end', function(evt, data) {
					// console.log("start end",data);
					key_front = false;
					key_left = false;
					key_back = false;
					key_right = false;
					roleIdle.play();
					roleRun.stop();
				});
			// })
			
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
			initShaders() {
			  // Vertex Shader
			  const vertexShader = `
				attribute float scale;
				void main() {
				  vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
				  gl_PointSize = scale * ( 300.0 / -mvPosition.z );
				  gl_Position = projectionMatrix * mvPosition;
				}
			  `;
		
			  // Fragment Shader
			  const fragmentShader = `
				uniform vec3 color;
				void main() {
				  if ( length( gl_PointCoord - vec2( 0.5, 0.5 ) ) > 0.475 ) discard;
				  gl_FragColor = vec4( color, 1.0 );
				}
			  `;
		
			  return { vertexShader, fragmentShader };
			},
			createPano() {
				const shaders = self.initShaders()
				container = document.querySelector("#sceneMain");
				// console.log(container);
				//create scene
				scene = new THREE.Scene();

				const fov = 40;
				const aspect = container.clientWidth / container.clientHeight;
				//camera setup
				camera = new THREE.PerspectiveCamera(fov, aspect);
				
				// // light
				// light = new THREE.DirectionalLight(0xffffff, 2.5);
				// light.position.set(10, 10, 10);
				// scene.add(light);
				// const light1 = new THREE.PointLight( 0xff0000, 1, 100 );
				// light1.position.set( 10, 10, 10 );
				// scene.add( light1 );

				//renderer
				renderer = new THREE.WebGLRenderer({
					antialias: true, // 抗锯齿
					depth: true ,// 开启深度缓冲
					// logarithmicDepthBuffer: true
				});
				
				renderer.setSize(container.clientWidth, container.clientHeight);
				renderer.setPixelRatio(window.devicePixelRatio);
				// renderer.logarithmicDepthBuffer = true;
				container.appendChild(renderer.domElement);
				


				// controls
				controls = new OrbitControls(camera, renderer.domElement);
				// 是否放大缩小
				// controls.enableZoom = false;
				// 右键拖动
				controls.enablePan = false;
				controls.maxDistance = 3;
				controls.minDistance = 1;
				controls.minPolarAngle = Math.PI * (45 / 180);
				controls.maxPolarAngle = Math.PI * (100 / 180);
				controls.target.set(0, 1.5, 0);
				
				controls.addEventListener( 'change', ()=>{
					dir.copy(controls.target).sub(controls.object.position);
					raycaster.far = dir.length();
					raycaster.set(controls.object.position, dir.normalize());
					const intersections = raycaster.intersectObjects( sceneModel.children );
					// console.log(intersections)
					const intersection = ( intersections.length ) > 0 ? intersections[ 0 ] : null;
					if(intersection && intersection) {
						controls.object.position.copy(intersection.point);
					}
				} );
				const numParticles = AMOUNTX * AMOUNTY;
				      // 创建粒子
				const positions = new Float32Array( numParticles * 3 );
				const scales = new Float32Array( numParticles );
				      // ... 初始化粒子位置和大小 ...
				let i = 0, j = 0;
				for ( let ix = 0; ix < AMOUNTX; ix ++ ) {
					for ( let iy = 0; iy < AMOUNTY; iy ++ ) {
						positions[ i ] = ix * SEPARATION - ( ( AMOUNTX * SEPARATION ) / 2 ); // x
						positions[ i + 1 ] = 0; // y
						positions[ i + 2 ] = iy * SEPARATION - ( ( AMOUNTY * SEPARATION ) / 2 ); // z
						scales[ j ] = 1;
						i += 3;
						j ++;
					}
				}
				
			  const geometry = new THREE.BufferGeometry();
			  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
			  geometry.setAttribute('scale', new THREE.BufferAttribute(scales, 1));
		
			  const material = new THREE.ShaderMaterial({
				uniforms: { color: { value: new THREE.Color(0xffffff) } },
				vertexShader: shaders.vertexShader,
				fragmentShader: shaders.fragmentShader,
			  });
		
			  particles = new THREE.Points(geometry, material);
			  scene.add(particles);
				
				setTimeout(()=>{
					self.loadScene();
				},2500)
				self.animate();
			},
			loaderHdr(){
				
				// TextureLoader
				TextureLoader = new THREE.TextureLoader();
				// console.log(TextureLoader);
				// 加载一个资源
				TextureLoader.load(
					// 'https://mob.hexntc.com/web0902/static/hdr/texture.jpg',
					'../../static/hdr/texture.jpg',
					// onLoad回调
					function(texture) {
						texture.mapping = THREE.EquirectangularReflectionMapping;
						// scene.background = texture; // 给场景添加背景图
						scene.environment = texture;
					}
					
				);
				// 加载一个资源
				TextureLoader.load(
					// 资源URL
					// 'https://mob.hexntc.com/web0902/static/hdr/meadow_2_1k.jpg',
					'../../static/hdr/meadow_2_1k.jpg',
					// onLoad回调
					function(texture) {
						texture.mapping = THREE.EquirectangularReflectionMapping;
						scene.background = texture; // 给场景添加背景图
						// scene.environment = texture;
					}
				);
			},
			loadScene() {
				loader = new GLTFLoader();
				loader.load(
				// 'https://mob.hexntc.com/web0902/static/model/classjs2.glb',
				'../../static/model/classjs6.glb',
				 function(node) {
					// console.log(node)
					scene.add(node.scene);
					sceneModel = node.scene;
					// console.log(sceneModel.children)
					const wall01 = sceneModel.getObjectByName('wall01');
					wall01.visible = false;
					const wall01_1 = sceneModel.getObjectByName('wall01_1');
					wall01_1.visible = false;
					
					const Box13 = sceneModel.getObjectByName('Box013');
					Box13.visible = false;
					const Object_16 = sceneModel.getObjectByName('Object_16');
					Object_16.visible = false;

					sceneModel.scale.set(0.95, 0.95, 0.95); // 将物体的尺寸缩小到原来的一半

					// 添加碰撞体
					worldOctree.fromGraphNode(sceneModel);
					octree_wall.fromGraphNode(wall01_1);
					
					// 创建一个八叉树助手
					// const octreeHelper = new OctreeHelper(octree_wall);
					// 将八叉树助手添加到场景中
					// scene.add(octreeHelper);
					
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
							controls.object.position.set(-6.5, 2.1, 0.9);
							controls.update();
						  }
					});
					// node.scenes[0].children[2].intensity = 0.8;
					
					if(self.roleId == 1){
						self.loaderHdr();
						self.loadRoleMale();
						controls.target.y += 0.15
						controls.object.position.set(-6.5, 2.1, 0.9);
						controls.update();
						// 加载完成隐藏粒子
						self.isLoading = false;
					}
					else if(self.roleId == 2){
						self.loaderHdr();
						self.loadRoleFemale();
						// 加载完成隐藏粒子
						self.isLoading = false;
					}
					else{
						// 加载完成隐藏粒子
						// self.isLoading = false;
						scene.remove(node.scene);
					}


					
				})
			},
			loadRoleFemale() {
				loader.load(
				'../../static/model/shaonvblanderdonghuajs01.glb',
				 function(node) {
					// console.log(node)
					scene.add(node.scene);
					
					roleModel = node.scene;
					roleAction = new THREE.AnimationMixer(roleModel);
					roleIdle = roleAction.clipAction(node.animations[0]);
					roleRun = roleAction.clipAction(node.animations[3]);
					roleWalk = roleAction.clipAction(node.animations[1]);
					roleIdle.play();
					roleModel.rotateY(Math.PI/2);
					roleModel.scale.set(1.2, 1.2, 1.2);

					const _face = roleModel.getObjectByName('网格006_1');
					_face.material.depthWrite = true;
					// console.log(_face)
					// const _tear = roleModel.getObjectByName('Female_8_1_TearShape');
					//  _tear.material.color.set(0x965454)
					 
					// 设置人物模型初始位置为出生点
					  if (birthPoint) {
						  // console.log(birthPoint)
						roleModel.position.copy(birthPoint.position);
						// controls.position(birthPoint.position)
					  } else {
						console.warn('未找到出生点！');
						roleModel.position.set(0, 0, 0);
					  }
					  
					  const material = new THREE.MeshBasicMaterial({
					    color: 0xff0000,
					    side: THREE.DoubleSide, // 使用单面材质
					    depthWrite: true, // 开启深度写入
					    depthTest: true, // 开启深度测试
					  });
					  self.isShowNipple = true;
				})
			},
			loadRoleMale() {
				loader.load(
				'../../static/model/donghuahebingjs01.glb',
				 function(node) {
					// console.log(node)
					scene.add(node.scene);
					
					roleModel = node.scene;
					roleAction = new THREE.AnimationMixer(roleModel);
					roleIdle = roleAction.clipAction(node.animations[0]);
					roleRun = roleAction.clipAction(node.animations[3]);
					roleWalk = roleAction.clipAction(node.animations[1]);
					roleIdle.play();
					roleModel.rotateY(Math.PI/2);
					roleModel.scale.set(1.05, 1.05, 1.05);
			
					// const _face = roleModel.getObjectByName('网格006_1');
					// _face.material.depthWrite = true;
					// console.log(_face)
					// const _tear = roleModel.getObjectByName('Female_8_1_TearShape');
					//  _tear.material.color.set(0x965454)
					 
					// 设置人物模型初始位置为出生点
					  if (birthPoint) {
						  // console.log(birthPoint)
						roleModel.position.copy(birthPoint.position);
						// controls.position(birthPoint.position)
					  } else {
						console.warn('未找到出生点！');
						roleModel.position.set(0, 0, 0);
					  }
					  
					  // const material = new THREE.MeshBasicMaterial({
					  //   color: 0xff0000,
					  //   side: THREE.DoubleSide, // 使用单面材质
					  //   depthWrite: true, // 开启深度写入
					  //   depthTest: true, // 开启深度测试
					  // });
					  self.isShowNipple = true;
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
				
				if(self.isLoading){
					camera.position.y = 300;
					camera.position.z = 1200;
					camera.lookAt( scene.position );
					const positions = particles.geometry.attributes.position.array;
					const scales = particles.geometry.attributes.scale.array;
					particles.geometry.attributes.position.needsUpdate = true;
					particles.geometry.attributes.scale.needsUpdate = true;
					let i = 0, j = 0;
					for ( let ix = 0; ix < AMOUNTX; ix ++ ) {
						for ( let iy = 0; iy < AMOUNTY; iy ++ ) {
							positions[ i + 1 ] = ( Math.sin( ( ix + _count ) * 0.3 ) * 20 ) +
											( Math.sin( ( iy + _count ) * 0.5 ) * 20 );
							scales[ j ] = ( Math.sin( ( ix + _count ) * 0.3 ) + 1 ) * 20 +
											( Math.sin( ( iy + _count ) * 0.5 ) + 1 ) * 20;
							i += 3;
							j ++;
						}
					}
					  _count += 0.1;
				}else{
					// 从场景中移除粒子
					scene.remove(particles);
								
					// 销毁几何体和材质
					// particles.geometry.dispose();
					// if (particles.material.map) 
					// particles.material.map.dispose();
					// particles.material.dispose();
								
					// 清除粒子引用
					particles = null;
				}

				
				// 获取当前时间
				let now = performance.now();
				// 计算时间差
				let deltaTime = now - lastTime;
				lastTime = now;
				if (roleAction) {
					roleAction.update(deltaTime / 1000);
				}
				renderer.render(scene, camera);
				// console.log(controls.object.position)
				// console.log(camera.position)

				if (roleModel) {

				// let radian_angle = controls.getAzimuthalAngle() * 180 / Math.PI;
				// console.log(collisionResult.depth)
				let max_multiple = 20;
				let min_multiple = 2;
				
				if (key_front) {
					const radian_cos = Math.cos(roleModel.rotation.y);
					const radian_sin = Math.sin(roleModel.rotation.y);
					const v3_1 = new THREE.Vector3();
					const v3_2 = new THREE.Vector3();
					const v3_3 = new THREE.Vector3();

					v3_1.copy(camera.position)
					v3_2.copy(controls.target)
					v3_3.copy(roleModel.position)
					
					camera.position.z += (speed*min_multiple * radian_cos);
					controls.target.z += (speed*min_multiple * radian_cos);
					camera.position.x += (speed*min_multiple * radian_sin);
					controls.target.x += (speed*min_multiple * radian_sin);
					roleModel.position.z += (speed*min_multiple * radian_cos);
					roleModel.position.x += (speed*min_multiple * radian_sin);
					
					// capsule
					roleCapsule.getCenter(roleCapsuleCenter);
					roleCapsuleV3.subVectors(role_originPoint, roleCapsuleCenter);
					roleCapsuleV3.setY(roleCapsuleV3.y);
					roleCapsule.translate(roleCapsuleV3);
					roleModel.updateMatrixWorld();
					roleModel.getWorldPosition(role_originPoint);
					const collisionResult = octree_wall.capsuleIntersect(roleCapsule);
					if (collisionResult) {
						// console.log(collisionResult.depth)
						camera.position.copy(v3_1)
						controls.target.copy(v3_2)
						roleModel.position.copy(v3_3)
					}
				}
				if (key_back) {
					const radian_cos = Math.cos(roleModel.rotation.y);
					const radian_sin = Math.sin(roleModel.rotation.y);
					const v3_1 = new THREE.Vector3();
					const v3_2 = new THREE.Vector3();
					const v3_3 = new THREE.Vector3();
					v3_1.copy(camera.position)
					v3_2.copy(controls.target)
					v3_3.copy(roleModel.position)
					camera.position.z += (speed*min_multiple * radian_cos);
					controls.target.z += (speed*min_multiple * radian_cos);
					camera.position.x += (speed*min_multiple * radian_sin);
					controls.target.x += (speed*min_multiple * radian_sin);
					roleModel.position.z += (speed*min_multiple * radian_cos);
					roleModel.position.x += (speed*min_multiple * radian_sin);
					// capsule
					roleCapsule.getCenter(roleCapsuleCenter);
					roleCapsuleV3.subVectors(role_originPoint, roleCapsuleCenter);
					roleCapsuleV3.setY(roleCapsuleV3.y);
					roleCapsule.translate(roleCapsuleV3);
					roleModel.updateMatrixWorld();
					roleModel.getWorldPosition(role_originPoint);
					const collisionResult = octree_wall.capsuleIntersect(roleCapsule);
					if (collisionResult) {
						// console.log(collisionResult.depth)
						camera.position.copy(v3_1)
						controls.target.copy(v3_2)
						roleModel.position.copy(v3_3)
					}
				}
				if (key_left) {
					const radian_cos = Math.cos(roleModel.rotation.y);
					const radian_sin = Math.sin(roleModel.rotation.y);
					const v3_1 = new THREE.Vector3();
					const v3_2 = new THREE.Vector3();
					const v3_3 = new THREE.Vector3();
					v3_1.copy(camera.position)
					v3_2.copy(controls.target)
					v3_3.copy(roleModel.position)
					camera.position.x += (speed*min_multiple * radian_sin);
					controls.target.x += (speed*min_multiple * radian_sin);
					camera.position.z += (speed*min_multiple * radian_cos);
					controls.target.z += (speed*min_multiple * radian_cos);
					roleModel.position.x += (speed*min_multiple * radian_sin);
					roleModel.position.z += (speed*min_multiple * radian_cos);
					// capsule
					roleCapsule.getCenter(roleCapsuleCenter);
					roleCapsuleV3.subVectors(role_originPoint, roleCapsuleCenter);
					roleCapsuleV3.setY(roleCapsuleV3.y);
					roleCapsule.translate(roleCapsuleV3);
					roleModel.updateMatrixWorld();
					roleModel.getWorldPosition(role_originPoint);
					const collisionResult = octree_wall.capsuleIntersect(roleCapsule);
					if (collisionResult) {
						// console.log(collisionResult.depth)
						camera.position.copy(v3_1)
						controls.target.copy(v3_2)
						roleModel.position.copy(v3_3)
					}


				}
				if (key_right) {
					const radian_cos = Math.cos(roleModel.rotation.y);
					const radian_sin = Math.sin(roleModel.rotation.y);
					const v3_1 = new THREE.Vector3();
					const v3_2 = new THREE.Vector3();
					const v3_3 = new THREE.Vector3();
					v3_1.copy(camera.position)
					v3_2.copy(controls.target)
					v3_3.copy(roleModel.position)
					camera.position.x += (speed*min_multiple * radian_sin);
					controls.target.x += (speed*min_multiple * radian_sin);
					camera.position.z += (speed*min_multiple * radian_cos);
					controls.target.z += (speed*min_multiple * radian_cos);
					roleModel.position.x += (speed*min_multiple * radian_sin);
					roleModel.position.z += (speed*min_multiple * radian_cos);
					// capsule
					roleCapsule.getCenter(roleCapsuleCenter);
					roleCapsuleV3.subVectors(role_originPoint, roleCapsuleCenter);
					roleCapsuleV3.setY(roleCapsuleV3.y);
					roleCapsule.translate(roleCapsuleV3);
					roleModel.updateMatrixWorld();
					roleModel.getWorldPosition(role_originPoint);
					const collisionResult = octree_wall.capsuleIntersect(roleCapsule);
					if (collisionResult) {
						// console.log(collisionResult.depth)
						camera.position.copy(v3_1)
						controls.target.copy(v3_2)
						roleModel.position.copy(v3_3)
					}
				}
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
		position: relative;
	}
	@media screen and (max-width:999px) {
		#joycon {
			position: absolute;
			left: 50%;
			top: 50%;
			width: 200rpx;
			height: 200rpx;
			transform: translateX(-190%) translateY(-15%);
			/* background-color: rgba(25, 25, 25, 0.1); */
		}
	}
	@media screen and (min-width:1000px) {
		#joycon {
			display: none;
			position: absolute;
			left: 0%;
			bottom: 50%;
			width: 200rpx;
			height: 200rpx;
			transform: translateX(0%) translateY(120%);
			/* background-color: rgba(25, 25, 25, 0.1); */
		}
	}
</style>