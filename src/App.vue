<script setup>

import { ref } from 'vue'
import * as THREE from 'three';

import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { EXRLoader } from "three/examples/jsm/loaders/EXRLoader.js";
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader.js';


const fileInput = ref(null)
const resourceURL = ref("");

const environmentTexture = ref(null)
const sceneReference = ref(null);

const triggerFileInput = () => {
  fileInput.value.click()
}

const handleUpload = (event) => {
  const file = event.target.files[0]
  if(!file) {
    return;
  }

  let _fileTexture = null;
  let objectURL = URL.createObjectURL(file);

  if (file.type.startsWith("video/")) {
    const videoElement = document.getElementById('video')
    videoElement.src = objectURL
    _fileTexture = new THREE.VideoTexture(videoElement);
  } else if(file.type.startsWith("image/")) {
    new THREE.TextureLoader().load(objectURL, (texture) => {
      texture.mapping = THREE.EquirectangularReflectionMapping;
      texture.encoding = THREE.sRGBEncoding;

      environmentTexture.value = texture;
      sceneReference.value.background = environmentTexture.value;
    });
  } else {
    const fileName = file.name.toLowerCase();
    const reader = new FileReader();

    if (fileName.endsWith(".hdr")) {
      reader.onload = function () {
        new RGBELoader().loadAsync(objectURL).then((texture) => {
          texture.mapping = THREE.EquirectangularReflectionMapping;
          environmentTexture.value = texture;
          sceneReference.value.background = environmentTexture.value;
        });
      };
      reader.readAsArrayBuffer(file);
    } else if (fileName.endsWith(".exr")) {
      reader.onload = function () {
        new EXRLoader().loadAsync(objectURL).then((texture) => {
          texture.mapping = THREE.EquirectangularReflectionMapping;
          environmentTexture.value = texture;
          sceneReference.value.background = environmentTexture.value;
        });
      };
      reader.readAsArrayBuffer(file);
    }
  }

  environmentTexture.value = _fileTexture;
  sceneReference.value.background = environmentTexture.value;
}

const handleUrlInput = () => {
  const url = resourceURL.value.trim().toLowerCase()
  if (!url) return

  if (url.endsWith('.mp4') || url.endsWith('.webm')) {
    const videoElement = document.getElementById('video')
    videoElement.src = url
    const texture = new THREE.VideoTexture(videoElement)
    texture.mapping = THREE.EquirectangularReflectionMapping
    texture.encoding = THREE.sRGBEncoding
    environmentTexture.value = texture
    sceneReference.value.background = environmentTexture.value
  } else if (url.endsWith('.hdr')) {
    new RGBELoader().loadAsync(url).then((texture) => {
      texture.mapping = THREE.EquirectangularReflectionMapping
      environmentTexture.value = texture
      sceneReference.value.background = environmentTexture.value
    })
  } else if (url.endsWith('.exr')) {
    new EXRLoader().loadAsync(url).then((texture) => {
      texture.mapping = THREE.EquirectangularReflectionMapping
      environmentTexture.value = texture
      sceneReference.value.background = environmentTexture.value
    })
  } else {
    new THREE.TextureLoader().load(url, (texture) => {
      texture.mapping = THREE.EquirectangularReflectionMapping
      texture.encoding = THREE.sRGBEncoding
      environmentTexture.value = texture
      sceneReference.value.background = environmentTexture.value
    })
  }
}


/// THREE
const width = window.innerWidth, height = window.innerHeight;

// init

const camera = new THREE.PerspectiveCamera(70, width / height, 0.01, 10);
camera.position.z = 1;

const scene = new THREE.Scene();
sceneReference.value = scene;
scene.background = environmentTexture.value;

// const mesh = new THREE.Mesh( geometry, material );
// scene.add(mesh);

const renderer = new THREE.WebGLRenderer({
  antialias: true,
});
renderer.setSize(width, height);
renderer.setAnimationLoop(animate);

const orbit = new OrbitControls(camera, renderer.domElement)
orbit.minZoom = orbit.maxZoom = 0;
document.body.appendChild( renderer.domElement );

// animation

function animate( time) {
  // mesh.rotation.x = time / 2000;
  // mesh.rotation.y = time / 1000;
  if(environmentTexture.value != null)
    environmentTexture.value.needsUpdate = true;

  renderer.render( scene, camera );
}
</script>

<template>
  <button @click="triggerFileInput">Upload Video</button>
  <input
      type="file"
      ref="fileInput"
      accept="video/*,image/*,.hdr,.exr"
      @change="handleUpload"
      style="display: none"
  />
  <input
      v-model="resourceURL"
      @keyup.enter="handleUrlInput"
      type="text"
      placeholder="Paste image/video/HDR/EXR URL and press Enter"
      style="margin-top: 1rem; width: 100%; padding: 0.5rem;"
  />
  <video id="video" playsinline webkit-playsinline loop autoplay style="display: none"></video>

  <main>
  </main>
</template>

<style>
#app {
  position: absolute;
  z-index: 1000;
}
</style>