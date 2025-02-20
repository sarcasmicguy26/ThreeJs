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
let recticle;
let hitTestSource = null;
let hitTestSourceRequested = false;
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

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
  directionalLight.position.set(0, 1, 1);
  scene.add(directionalLight);

  const ambientLight = new THREE.AmbientLight(0xffffff, 0.2);
  scene.add(ambientLight);

  // HANDLE RENDERE, ANIMATE, AND RESIZE
  function animate(timestamp, frame) {
    controls.update();

    if(frame) {
      const referenceSpace = renderer.xr.getReferenceSpace();
      const session = renderer.xr.getSession();

      if(hitTestSourceRequested === false) {
        session.requestReferenceSpace('viewer').then((referenceSpace) => {
          session.requestHitTestSource({ space: referenceSpace }).then((source) => {
            hitTestSource = source;
          });
        });
        hitTestSourceRequested = true;

        session.addEventListener('end', () => {
          hitTestSourceRequested = false;
          hitTestSource = null;
        });
      }

      if(hitTestSource) {
        const hitTestResults = frame.getHitTestResults(hitTestSource);
        if(hitTestResults.length) {
          const hit = hitTestResults[0];
          recticle.visible = true;
          recticle.matrix.fromArray(hit.getPose(referenceSpace).transform.matrix);
        } else {
          recticle.visible = false;
        }
      }
    }
    // ROTATE BOX BY ELAPSED TIME
    renderer.render(scene, camera);
  }
  renderer.setAnimationLoop(animate);
  window.onresize = () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  }

  const rxController = renderer.xr.getController(0);
  rxController.addEventListener('select', onSelect);
});

function onSelect() {
  const box = new THREE.Mesh(
    new THREE.BoxGeometry(0.1, 0.1, 0.1),
    new THREE.MeshStandardMaterial({ color: 0xff0000 })
  );
  box.position.set(0, 0, -0.5).applyMatrix4(recticle.matrix);
  scene.add(box);
}

function addRecticle() {
  recticle = new THREE.Mesh(
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
