<script lang="ts">
  import { onMount } from 'svelte';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

  const resolution = { width: 800, height: 600 };

  let canvas: HTMLCanvasElement;
  let cameraSelection: string = $state('PerspectiveCamera');
  let fovSelection: number = $state(75);
  let customControls = $state(true);

  let cursor = $state({ x: 0, y: 0 });
  let mousemove = (event: MouseEvent) => {
    cursor = {
      x: event.clientX / resolution.width - 0.5,
      y: event.clientY / resolution.height - 0.5
    };
  };

  function myScene() {
    const scene = new THREE.Scene();
    const aspectRatio = resolution.width / resolution.height;

    const geometry = new THREE.BoxGeometry();
    const material = new THREE.MeshBasicMaterial({ color: 'red' });
    const mesh = new THREE.Mesh(geometry, material);
    scene.add(mesh);

    // helper
    const axesHelper = new THREE.AxesHelper();
    scene.add(axesHelper);

    // camera
    let camera = new THREE.PerspectiveCamera(fovSelection, resolution.width / resolution.height);
    camera.position.z = 2;
    scene.add(camera);

    // renderer
    const renderer = new THREE.WebGLRenderer({ canvas });
    renderer.setSize(resolution.width, resolution.height);

    // animation
    const clock = new THREE.Clock();
    let lastCameraSelection = '';
    const tick = () => {
      const elapsedTime = clock.getElapsedTime();

      mesh.position.x = Math.cos(2 * elapsedTime);
      mesh.rotation.y = elapsedTime;

      if (lastCameraSelection !== cameraSelection)
        switch (cameraSelection) {
          case 'OrthographicCamera':
            camera = new THREE.OrthographicCamera(-1 * aspectRatio, 1 * aspectRatio);
            break;
          default:
            camera = new THREE.PerspectiveCamera(fovSelection, aspectRatio);
            break;
        }
      camera.fov = fovSelection;

      let controls: OrbitControls | undefined;
      if (customControls) {
        camera.position.set(cursor.x, cursor.y, camera.position.z);

        // deregister orbit
        if (controls !== undefined) {
          controls.dispose();
          controls = undefined;
        }
      } else {
        controls = new OrbitControls(camera, canvas);
        controls.enableDamping = true;
        controls.update();
      }
      camera.updateProjectionMatrix();

      renderer.render(scene, camera);
      window.requestAnimationFrame(tick);
    };

    lastCameraSelection = cameraSelection;
    tick();
  }

  onMount(() => {
    myScene();
  });
</script>

<svelte:window on:mousemove={mousemove} />

<select class="select" bind:value={cameraSelection}>
  <option selected>PerspectiveCamera</option>
  <option>OrthographicCamera</option>
</select>

<input type="number" defaultValue="75" bind:value={fovSelection} placeholder="Field of View" />
<label>
  <input type="checkbox" bind:checked={customControls} class="toggle" />
  Custom controls (unchecked orbit controls)
</label>

<canvas bind:this={canvas}></canvas>
