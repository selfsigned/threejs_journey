<script lang="ts">
  import { onMount } from 'svelte';
  import * as THREE from 'three';

  let canvas: HTMLCanvasElement;

  function myScene() {
    const scene = new THREE.Scene();
    const resolution = { width: 800, height: 600 };

    const geometry = new THREE.BoxGeometry();
    const material = new THREE.MeshBasicMaterial({ color: 'red' });
    const mesh = new THREE.Mesh(geometry, material);
    scene.add(mesh);

    // helper
    const axesHelper = new THREE.AxesHelper();
    scene.add(axesHelper);

    // camera
    const camera = new THREE.PerspectiveCamera(75, resolution.width / resolution.height);
    camera.position.z = 2;
    scene.add(camera);

    // renderer
    const renderer = new THREE.WebGLRenderer({ canvas });
    renderer.setSize(resolution.width, resolution.height);

    // animation
    const clock = new THREE.Clock();
    const tick = () => {
      const elapsedTime = clock.getElapsedTime();

      camera.position.x = Math.cos(2 * elapsedTime);
      camera.lookAt(mesh.position);

      renderer.render(scene, camera);
      window.requestAnimationFrame(tick);
    };

    tick();
  }

  onMount(() => {
    myScene();
  });
</script>

<canvas bind:this={canvas}></canvas>
