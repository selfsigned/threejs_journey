<script lang="ts">
  import { onMount } from 'svelte';
  import * as THREE from 'three';
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
  import { animate } from 'motion';
  import GUI from 'lil-gui';

  let canvas: HTMLCanvasElement;

  const resolution = { width: 800, height: 600 }; // fallback values

  // lil-gui
  let gui: GUI;

  // threejs vars
  let camera: THREE.PerspectiveCamera;
  let renderer: THREE.WebGLRenderer;
  let mesh: THREE.Mesh;

  // spin animation
  const spinAnimate = () => {
    animate(
      mesh.rotation,
      {
        y: mesh.rotation.y + Math.PI * 2
      },
      { duration: 2, ease: 'easeInOut' }
    );
  };

  // custom gui methods
  const guiMethods = {
    color: '#a778d8',
    subdivision: 2,
    spinFunc: spinAnimate
  };

  function myScene() {
    // set initial resolution and create Scene
    resolution.height = window.innerHeight;
    resolution.width = window.innerWidth;
    const scene = new THREE.Scene();

    // mesh, circle because I'm a hipster
    const geometry = new THREE.BoxGeometry(1, 1, 1);
    const material = new THREE.MeshBasicMaterial({ color: guiMethods.color, wireframe: true });
    mesh = new THREE.Mesh(geometry, material);
    scene.add(mesh);

    // gui
    const meshFolder = gui.addFolder('Cube Mesh');
    meshFolder.add(mesh.position, 'y', -3, 3, 0.01).name('elevation');
    meshFolder.add(mesh, 'visible').name('visible');
    meshFolder.add(mesh.material as THREE.MeshBasicMaterial, 'wireframe').name('wireframe');
    meshFolder // subdvision tweak
      .add(guiMethods, 'subdivision')
      .min(1)
      .max(25)
      .step(1)
      .onFinishChange(() => {
        mesh.geometry.dispose(); // dispose old geometry from memory
        mesh.geometry = new THREE.BoxGeometry(
          ...Array(3).fill(1),
          ...Array(3).fill(guiMethods.subdivision)
        );
        console.log('update geometry');
      });
    meshFolder // color tweak with colorspace hack
      .addColor(guiMethods, 'color')
      .name('color')
      .onChange(() => {
        material.color.set(guiMethods.color);
      });
    meshFolder.add(guiMethods, 'spinFunc').name('SPIN !');

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

  let handleShortcuts = (event) => {
    if (event.key === 'f') {
      toggleFullscren();
    }
    if (event.key === 'g') {
      print(gui);
      if (gui._hidden) {
        gui.show();
      } else {
        gui.hide();
      }
    }
  };

  let toggleFullscren = () => {
    if (document.fullscreenElement) {
      document.exitFullscreen();
    } else {
      canvas.requestFullscreen();
    }
  };

  onMount(() => {
    gui = new GUI({
      width: 225,
      title: 'Debug UI',
      closeFolders: false
    });
    // gui.hide()

    myScene();
  });
</script>

<svelte:window
  on:resize={handleResize}
  on:dblclick={toggleFullscren}
  on:keypress={handleShortcuts}
/>

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
