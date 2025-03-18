<script lang="ts">
  import { onMount } from 'svelte';
  import * as THREE from 'three';

  let canvas: HTMLCanvasElement;

  async function myScene() {
    const scene = new THREE.Scene();
    const resolution = { width: 800, height: 600 };

    // Mesh group
    const group = new THREE.Group();
    scene.add(group);
    for (let i = 0; i < 3; ++i) {
      // >writing redundant imperative code
      // > any year
      const cube = new THREE.Mesh(
        new THREE.BoxGeometry(1, 1, 1),
        new THREE.MeshBasicMaterial({ color: (i + 1) * 0xc00000 })
      );
      group.add(cube);

      cube.position.x += i * 2;
      cube.scale.y = i + 1;
    }
    group.position.x = -2; // move the whooe group to the left

    // Camera
    const camera = new THREE.PerspectiveCamera(75, resolution.width / resolution.height);
    camera.position.z = 3;
    scene.add(camera);
    camera.rotation.reorder('YXZ'); // have axes rotation worklike FPS games
    camera.rotation.y = Math.PI * 0.05;
    camera.rotation.x = Math.PI * 0.05;

    // Renderer
    const renderer = new THREE.WebGLRenderer({ canvas });
    renderer.setSize(resolution.width, resolution.height);
    renderer.render(scene, camera);
  }

  onMount(() => {
    myScene();
  });
</script>

<canvas bind:this={canvas}></canvas>
