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

    // mesh
    const cubeGeometry = new THREE.BoxGeometry(1, 1, 1, 3, 2, 2);
    const wireframeMaterial = new THREE.MeshBasicMaterial({
      color: 'red',
      wireframe: true
    });
    const cubeMesh = new THREE.Mesh(cubeGeometry, wireframeMaterial);
    const CircleGeometry = new THREE.SphereGeometry(0.5, 12, 32);
    const circleMesh = new THREE.Mesh(CircleGeometry, wireframeMaterial);
    circleMesh.position.x = -1;
    const basicGeometries = new THREE.Group();
    basicGeometries.add(cubeMesh);
    basicGeometries.add(circleMesh);
    scene.add(basicGeometries);

    // custom geometry
    const customGeometry = new THREE.BufferGeometry();
    const positionArray = new Float32Array([0, 0, 0, 0, 1, 0, 1, 0, 0]);
    const positionAttribute = new THREE.BufferAttribute(positionArray, 3);
    customGeometry.setAttribute('position', positionAttribute);
    const customMesh = new THREE.Mesh(customGeometry, wireframeMaterial);
    scene.add(customMesh);

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
    const clock = new THREE.Clock();
    const tick = () => {
      // create new custom meshes
      if (clock.getElapsedTime() >= 0.25) {
        const count = 5;
        const randomPositions = new Float32Array(count * 3 * 3);
        for (let i = 0; i < count * 3 * 3; i++) {
          randomPositions[i] = Math.random() - 0.5;
        }
        const randomPositionsAttribute = new THREE.BufferAttribute(randomPositions, 3);
        const randomGeometry = new THREE.BufferGeometry();
        randomGeometry.setAttribute('position', randomPositionsAttribute);
        const randomMesh = new THREE.Mesh(randomGeometry, wireframeMaterial);
        scene.add(randomMesh);

        // reset the clock
        clock.start();
      }

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
