<template>
  <view id="panoParent">
    <!-- 这里将显示Three.js渲染的模型 -->
  </view>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';

export default {
  data() {
    return {
      scene: null,
      camera: null,
      renderer: null,
    };
  },
  mounted() {
    this.initThree();
    this.loadModel();
  },
  methods: {
    initThree() {
      const width = window.innerWidth;
      const height = window.innerHeight;

      this.scene = new THREE.Scene();
      this.camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
      this.renderer = new THREE.WebGLRenderer();
      this.renderer.setSize(width, height);

	  var parent = document.getElementById('panoParent');
      parent.appendChild(this.renderer.domElement);

      this.animate();
    },
    loadModel() {
      const loader = new GLTFLoader();
      loader.load('/static/model/class.glb', (gltf) => {
        this.scene.add(gltf.scene);
      }, undefined, (error) => {
        console.error(error);
      });
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.renderer.render(this.scene, this.camera);
    },
  },
};
</script>

<style>
.model-container {
  width: 100%;
  height: 100%;
  position: relative;
}
</style>