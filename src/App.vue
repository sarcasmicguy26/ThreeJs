<template>
  <div>
    <canvas @dblclick="resetCamera" id="canvas"></canvas>

    <button style="position: fixed; top: 5px; left: 5px;" @click="$refs.filePick.click()">Upload</button>
    <input style="display: none;" type="file" @change="loadFile" ref="filePick"/>
  </div>
</template>

<script setup>

import * as THREE from "three";
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls";
import { onMounted } from "vue";
import {DragControls} from "three/examples/jsm/controls/DragControls";
import {RGBELoader} from "three/examples/jsm/loaders/RGBELoader";
import GUI from 'lil-gui'; 
import {ARButton} from 'three/examples/jsm/webxr/ARButton.js';
// import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

let controls;
let scene;
let photoFrame;
const gui = new GUI();
gui.add({ resetCamera }, "resetCamera").name("Reset Camera");
onMounted(() => {
  const canvas = document.getElementById("canvas");
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
  
  scene = new THREE.Scene();
  // SET ENVIRONMENT COLOR
  new RGBELoader()
  .load(new URL("@/assets/env.hdr", import.meta.url).href, (texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping;
    scene.environment = texture;
    scene.background = texture;
    scene.backgroundIntensity = 0.5;
    scene.environmentIntensity = 0.18;

    gui.add(scene, "backgroundIntensity", 0, 1).name("Back Intensity");
    gui.add(scene, "environmentIntensity", 0, 1).name("Env Intensity");
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    renderer.outputEncoding = THREE.sRGBEncoding;
  });

  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.z = 15;
  camera.position.y = 4;
  camera.position.x = 0;
  
  const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
  renderer.shadowMap.enabled = true;

  // const loader = new THREE.ObjectLoader();

  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  
  // loader.load("obj3.json",
  //   (obj) => {
  //     obj.traverse((child) => {
  //       if (child.isMesh) {
  //         child.castShadow = true;  // ✅ Explicitly enable shadow casting
  //         child.receiveShadow = true;  // ✅ Ensure the ground receives shadows
  //       }
  //       if (child.isDirectionalLight) {
  //         child.castShadow = true;
  //         child.shadow.mapSize.width = 1024;
  //         child.shadow.mapSize.height = 1024;
  //         child.shadow.camera.near = 0.5;
  //         child.shadow.camera.far = 50;
  //       }
  //     });
  //     scene.add(obj);
  //   },
  //   () => {},
  //   (error) => {
  //     console.error("An error happened.", error);
  //   }
  // );

  // const gltfLoader = new GLTFLoader();
  // gltfLoader.load("suzan.glb",
  //   (gltf) => {
  //     gltf.scene.traverse((child) => {

  //       if (child.isMesh) {
  //         if(child.name === "Suzanne") {
  //           child.position.y=1.05
  //         }
  //         child.castShadow = true;  // ✅ Explicitly enable shadow casting
  //         child.receiveShadow = true;  // ✅ Ensure the ground receives shadows
  //       }
  //       if (child.isDirectionalLight) {
  //         child.castShadow = true;
  //         child.shadow.mapSize.width = 1024;
  //         child.shadow.mapSize.height = 1024;
  //         child.shadow.camera.near = 0.5;
  //         child.shadow.camera.far = 50;
  //       }
  //     });
  //     scene.add(gltf.scene);
  //   },
  //   () => {},
  //   (error) => {
  //     console.error("An error happened.", error);
  //   }
  // );

  // const light = new THREE.DirectionalLight(0xffffff, 1);
  // light.position.set(4.6, 6.06, 10);
  // light.castShadow = true;
  // light.shadow.mapSize.width = 1024;
  // light.shadow.mapSize.height = 1024;
  // light.shadow.camera.near = 0.5;
  // light.shadow.camera.far = 50;
  // scene.add(light);
  // const direLight = gui.addFolder("Directional Light");
  // direLight.add(light.position, "x", -10 , 10).name("Position X");
  // direLight.add(light.position, "y", -10 , 10).name("Position Y");
  // direLight.add(light.position, "z", -10 , 10).name("Position Z");
  // direLight.addColor(light, "color").name("Color");
  // direLight.add(light, "intensity", 0, 100).name("Intensity");
  // direLight.close();

  const ambient = new THREE.AmbientLight(0xffffff, 0.5);
  scene.add(ambient);

  const polygon = new THREE.Mesh(
    new THREE.ExtrudeGeometry(
      new THREE.Shape([
        new THREE.Vector2(0, 0),
        new THREE.Vector2(0.5, 0.5),
        new THREE.Vector2(1, 0),
        new THREE.Vector2(0, 0)
      ]),
      { depth: 0.1, bevelEnabled: true, bevelThickness: 0.05, bevelSize: 0.025 }
    ),
    new THREE.MeshStandardMaterial({ color: 0x00ff00 })
  );
  polygon.position.x = -0.45;
  polygon.position.z = 3.05;
  polygon.position.y = 3.5;
  polygon.castShadow = true;
  polygon.receiveShadow = true;
  scene.add(polygon);

  const polyFolder = gui.addFolder("Polygon");
  polyFolder.add(polygon.position, "x", -10 , 10).name("Position X");
  polyFolder.add(polygon.position, "y", -10 , 10).name("Position Y");
  polyFolder.add(polygon.position, "z", -10 , 10).name("Position Z");
  polyFolder.add(polygon.rotation, "x", 0, 360).name("Rotation X");
  polyFolder.add(polygon.rotation, "y", 0, 360).name("Rotation Y");
  polyFolder.add(polygon.rotation, "z", 0, 360).name("Rotation Z");
  polyFolder.addColor(polygon.material, "color").name("Color");
  polyFolder.close();

  createRack();

  // WALL FOR FRAME
  // const wallTexture = new THREE.TextureLoader().load(new URL("@/assets/wall.jpg", import.meta.url).href)
  // wallTexture.wrapS = THREE.RepeatWrapping;
  // wallTexture.wrapT = THREE.RepeatWrapping;
  // wallTexture.repeat.set( 2.2, 1.8 );
  // const wallGeometry = new THREE.BoxGeometry(10, 5, 0.5);
  // const wallMaterial = new THREE.MeshStandardMaterial({map: wallTexture});
  // const wallMesh = new THREE.Mesh(wallGeometry, wallMaterial);
  // wallMesh.position.y = 2.5;
  // wallMesh.position.z = 5.65
  // wallMesh.receiveShadow = true;
  // wallMesh.castShadow = true;
  // const wallFol = gui.addFolder("Plane Texture");
  // wallFol.add(wallTexture.repeat, "x", 0, 10).name("Repeat X");
  // wallFol.add(wallTexture.repeat, "y", 0, 10).name("Repeat Y");
  // wallFol.add(wallTexture.offset, "x", 0, 10).name("Offset X");
  // wallFol.add(wallTexture.offset, "y", 0, 10).name("Offset Y");
  // scene.add(wallMesh);

  const spotLight = new THREE.SpotLight("white", 2, 10, 1.03);
  spotLight.castShadow = true;
  spotLight.position.set(0, 4, 6.2)
  scene.add(spotLight);
  const spotFolder = gui.addFolder("SpotLight");
  spotFolder.add(spotLight.position, "x", 0, 10).name("x");
  spotFolder.add(spotLight.position, "y", 0, 10).name("y");
  spotFolder.add(spotLight.position, "z", 0, 10).name("z");
  spotFolder.add(spotLight, "intensity", 0, 100).name("intensity");
  spotFolder.add(spotLight, "angle", 0, 6.28319).name("Angle");
  spotFolder.addColor(spotLight, "color").name("Color");
  spotFolder.add(spotLight, "distance", 0, 100).name("distance");

  // PLANE
  const img = new URL("@/assets/texture.jpg", import.meta.url).href;
 
  const texture = new THREE.TextureLoader().load(img);

  photoFrame= new THREE.Mesh(
    new THREE.BoxGeometry(1.1, 1.4, 0.1),
    new THREE.MeshStandardMaterial({ map: texture })
  );
  photoFrame.position.y = 2.3;
  photoFrame.position.z = 5.95;
  photoFrame.receiveShadow = true;
  photoFrame.castShadow = true;
  scene.add(photoFrame);

  const planeFol = gui.addFolder("Plane Texture");
  planeFol.add(texture.repeat, "x", 0, 10).name("Repeat X");
  planeFol.add(texture.repeat, "y", 0, 10).name("Repeat Y");
  planeFol.add(texture.offset, "x", 0, 10).name("Offset X");
  planeFol.add(texture.offset, "y", 0, 10).name("Offset Y");
  planeFol.add(photoFrame.material, "roughness", 0, 1).name("Roughness");
  planeFol.add(photoFrame.material, "metalness", 0, 1).name("metalness");


  const dragControls = new DragControls([polygon, photoFrame], camera, renderer.domElement);
  dragControls.rotateSpeed = 0.5; 
  dragControls.addEventListener("dragstart", (event) => {
    controls.enabled = false;

    event.object.rotation.x = 0;
    event.object.rotation.z = 0;
    event.object.material.transparent = true;
    event.object.material.transparency = 0.5;
  });
  dragControls.addEventListener("dragend", (event) => {
    event.object.material.transparency = 0.5;
    controls.enabled = true;
  });
  dragControls.deactivate();
  dragControls.activate();

  renderer.xr.enabled = true;
  document.body.appendChild(ARButton.createButton(renderer, {
    requiredFeatures: ["hit-test"]
  }));

  addRecticle();


  function animate() {
    controls.update();
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

function loadFile(e) {
  const reader = new FileReader();
  reader.onload = (e) => {
    const data = e.target.result;

    photoFrame.material.map = new THREE.TextureLoader().load(data);
  };
  reader.readAsDataURL(e.target.files[0]);
}

function rackBase() {
  const geometry = new THREE.BoxGeometry(0.3, 0.2, 2.5);
  const material = new THREE.MeshStandardMaterial({ color: 0x6f1f1f });
  const cube = new THREE.Mesh(geometry, material);
  cube.castShadow = true;
  cube.receiveShadow = true;

  return cube;
}

const boxes = new THREE.Group();
function createRack() {
  const rack = new THREE.Group();
  for (let i = 0; i < 4; i++) {
    const base = rackBase();
    base.position.x = i * 0.4;
    rack.add(base);
  }

  rack.position.x = 2;
  rack.position.y = 0.2;
  rack.position.z = 6;

  const rackFolder = gui.addFolder("Rack");
  rackFolder.addColor(rackBase().material , "color").name("Color").onChange((color) => {
    rack.children.forEach((child) => {
      child.material.color.set(color);
    });
  });
  rackFolder.add(rack.rotation, "y", 0 , 6.283188000000001).name("Rotation").onChange((rotation) => {
    rack.rotation.y = rotation
  });
  rackFolder.close();
  scene.add(rack);

  boxes.position.x = 0;
  boxes.position.y = 0;
  boxes.position.z = 0;

  const boxesFolder = gui.addFolder("Boxes");
  boxesFolder.add({ addBox }, "addBox").name("Add Box");
  boxesFolder.add({ removeBox }, "removeBox").name("Remove Box");
  boxesFolder.add(boxes.position, "x", -10 , 10).name("Position X");
  boxesFolder.add(boxes.position, "y", -10 , 10).name("Position Y");
  boxesFolder.add(boxes.position, "z", -10 , 10).name("Position Z");
  boxesFolder.close();
  rack.add(boxes);
  addBox();
}

function addBox() {
  const boxTexture = new THREE.TextureLoader().load(new URL("@/assets/box.jpg", import.meta.url).href);
  const boxNormalTexture = new THREE.TextureLoader().load(new URL("@/assets/NormalMap.png", import.meta.url).href);
  const geometry = new THREE.BoxGeometry(0.7, 0.7, 0.7,10,10,10);
  const material = new THREE.MeshStandardMaterial({ map: boxTexture, normalMap: boxNormalTexture, normalMapType: THREE.ObjectSpaceNormalMap, color: "#f97a71"});
  const cube = new THREE.Mesh(geometry, material);
  const boxLen = boxes.children.length;
  const isOdd = boxLen % 2 !== 0;
  const newBase = boxLen % 6 === 0;
  if(boxes.children.length === 0 || (boxLen % 6 === 0 && boxLen !== 0)) {
    const prevFirst = boxes.children[boxLen-6];
    cube.position.x = 0.2;
    cube.position.y = (boxLen ? prevFirst.position.y : 0.05) + (boxLen && newBase ? 0.7 : 0);
    cube.position.z = 0.8;
  }
  else if(boxes.children.length > 0) {
    const prevBox = newBase ? boxes.children[boxLen - 6] : boxes.children[boxLen - (isOdd ? 1 : 2)];
    cube.position.x = prevBox.position.x + (isOdd ? 0.8 : 0);
    cube.position.y = prevBox.position.y + (newBase ? 0.8 : 0);
    cube.position.z = prevBox.position.z + (!isOdd ? -0.8 : 0);
  }
  cube.castShadow = true;
  cube.receiveShadow = true;
  cube.name = "Cube"+boxLen
  boxes.add(cube);
  boxes.position.y = 0.4;
}

function removeBox() {
  boxes.children.pop();
}

function resetCamera() {
  controls.reset();
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
