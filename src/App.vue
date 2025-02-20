<template>
  <div>
    <canvas id="canvas"></canvas>
  </div>
</template>

<script setup>

import * as THREE from "three";
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls";
import { onMounted } from "vue";
// import GUI from 'lil-gui'; 
import {ARButton} from 'three/examples/jsm/webxr/ARButton.js';

let controls;
let scene;
// const gui = new GUI();
onMounted(() => {
  const canvas = document.getElementById("canvas");
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;

  // ADD SCENE
  scene = new THREE.Scene();

  // ADD CAMERA
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.set(0, 1, 2);
  
  // INIT RENDERER
  const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
  renderer.shadowMap.enabled = true;

  // ORBITAL CONTROLS
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;

  // ENABLE AR/VR
  renderer.xr.enabled = true;
  document.body.appendChild(ARButton.createButton(renderer, {
    requiredFeatures: ["hit-test"]
  }));

  // ADD RECTICLE
  addRecticle();

  const box = new THREE.Mesh(
    new THREE.BoxGeometry(1, 1, 1),
    new THREE.MeshStandardMaterial({ color: 0x00ff00 })
  );
  scene.add(box);


  const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
  directionalLight.position.set(0, 1, 1);
  scene.add(directionalLight);

  const ambientLight = new THREE.AmbientLight(0xffffff, 0.2);
  scene.add(ambientLight);

  // HANDLE RENDERE, ANIMATE, AND RESIZE
  function animate() {
    controls.update();
    // ROTATE BOX BY ELAPSED TIME
    box.rotation.y += 0.01;
    renderer.render(scene, camera);
  }
  renderer.setAnimationLoop(animate);
  window.onresize = () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  }
});

function addRecticle() {
  const recticle = new THREE.Mesh(
    new THREE.RingGeometry(0.15, 0.2, 32),
    new THREE.MeshBasicMaterial({ color: 0xffffff, side: THREE.DoubleSide })
  );
  recticle.position.z = -0.5;
  recticle.position.y = 0.1;
  recticle.rotation.x = Math.PI / 2;
  recticle.matrixAutoUpdate = false;
  recticle.visible = false;
  scene.add(recticle);
}

</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

body {
  margin: 0;
  overflow: hidden;
}
</style>
