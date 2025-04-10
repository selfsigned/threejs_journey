<script lang="ts">
  import { onMount } from 'svelte';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

  let canvas: HTMLCanvasElement;

  const resolution = { width: 800, height: 600 }; // fallback values

  // threejs vars
  let camera: THREE.PerspectiveCamera;
  let renderer: THREE.WebGLRenderer;

  function myScene() {
    // set initial resolution and create Scene
    resolution.height = window.innerHeight;
    resolution.width = window.innerWidth;
    const scene = new THREE.Scene();

    // mesh, circle because I'm a hipster
    const geometry = new THREE.BoxGeometry(1, 1, 1);
    const material = new THREE.MeshBasicMaterial({ color: 'red' });
    const mesh = new THREE.Mesh(geometry, material);
    scene.add(mesh);

    // camera
    camera = new THREE.PerspectiveCamera(75, resolution.width / resolution.height);
    camera.position.z = 2;
    scene.add(camera);

    // orbit controls
    const controls = new OrbitControls(camera, canvas);
    controls.enableDamping = true;

    // renderer
    renderer = new THREE.WebGLRenderer({ canvas });
    renderer.setSize(resolution.width, resolution.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // retina, limited for mobile

    // render scene
    const tick = () => {
      // update controls
      controls.update();
      // render
      renderer.render(scene, camera);
      // call tick again on the next frame
      window.requestAnimationFrame(tick);
    };
    tick();
  }

  let handleResize = () => {
    resolution.height = window.innerHeight;
    resolution.width = window.innerWidth;

    // Camera update
    camera.aspect = resolution.width / resolution.height;
    camera.updateProjectionMatrix();

    // Renderer update
    renderer.setSize(resolution.width, resolution.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // useful when screen changes
  };

  let toggleFullscren = () => {
    if (document.fullscreenElement) {
      document.exitFullscreen();
    } else {
      canvas.requestFullscreen();
    }
  };

  onMount(() => {
    myScene();
  });
</script>

<svelte:window on:resize={handleResize} on:dblclick={toggleFullscren} />

<canvas bind:this={canvas}></canvas>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    overflow: hidden;
  }
  canvas {
    position: absolute;
    top: 0;
    left: 0;
    outline: none;
  }
</style>
