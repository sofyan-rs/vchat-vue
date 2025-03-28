<script setup lang="ts">
import { onMounted, ref } from "vue";
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { VRMLoaderPlugin } from "@pixiv/three-vrm";

const vrmContainer = ref<HTMLDivElement | null>(null);

onMounted(() => {
  // Initialize Three.js scene
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  if (vrmContainer.value) {
    vrmContainer.value.appendChild(renderer.domElement);
  } else {
    console.error("vrmContainer is not available.");
  }

  // Add lighting
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
  directionalLight.position.set(1, 1, 1).normalize();
  scene.add(directionalLight);

  // Optionally add ambient light for softer shadows
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
  scene.add(ambientLight);

  // Load VRM model
  const loader = new GLTFLoader();
  loader.register((parser) => new VRMLoaderPlugin(parser));
  loader.load(
    "/VChatAvatar.vrm",
    (gltf) => {
      const vrm = gltf.userData.vrm;
      if (!vrm) {
        console.error("VRM data not found in the model.");
        return;
      }
      // Rotate the model 180 degrees so it faces the camera
      vrm.scene.rotation.y = Math.PI;
      scene.add(vrm.scene);
      animate();
    },
    (progress) =>
      console.log(
        `Loading model: ${((progress.loaded / progress.total) * 100).toFixed(
          2
        )}%`
      ),
    (error) => console.error("Error loading VRM model:", error)
  );

  // Position the camera so it looks at the model
  camera.position.set(0, 1.5, 10);
  camera.lookAt(new THREE.Vector3(0, 1.5, 0));

  function animate() {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
  }
});
</script>

<template>
  <div ref="vrmContainer" class="vrm-container"></div>
</template>

<style scoped>
.vrm-container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>
