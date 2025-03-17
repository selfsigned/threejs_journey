<script lang="ts">
  import { onMount } from 'svelte';
  import * as THREE from 'three';

  let canvas: HTMLCanvasElement;

  function myScene() {
    const scene = new THREE.Scene();
    const resolution = { width: 800, height: 600 };

    // mesh, circle because I'm a hipster
    const geometry = new THREE.CircleGeometry();
    const material = new THREE.MeshBasicMaterial({ color: 'red' });
    const mesh = new THREE.Mesh(geometry, material);
    scene.add(mesh);

    // camera
    const camera = new THREE.PerspectiveCamera(75, resolution.width / resolution.height);
    camera.position.z = 2;
    scene.add(camera);

    // renderer
    const renderer = new THREE.WebGLRenderer({ canvas });
    renderer.setSize(resolution.width, resolution.height);
    renderer.render(scene, camera);
  }

  onMount(() => {
    myScene();
  });
</script>

<canvas bind:this={canvas}></canvas>
