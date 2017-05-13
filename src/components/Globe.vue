<template>
  <div class="globe-view">
    <div ref="canvas" class="canvas"></div>
  </div>
</template>

<script>
import * as three from 'three';

export default {
  name: 'globe',
  data() {
    return {
      // The threejs render
      renderer: null,

      // The main scene, where everything is rendered
      scene: null,

      // The main camera
      camera: null,

      globe: null,
      globeGeometry: null,
      globeTexture: null,

      // The light representing the sun
      sun: null,

      clock: null,
    };
  },

  methods: {

    drawGlobe() {
      this.globeGeometry = new three.SphereGeometry(5, 32, 32);
      this.globeGeometry.x = 0;
      this.globeGeometry.y = 0;
      this.globeGeometry.z = 0;

      this.globeTexture = new three.MeshStandardMaterial({
        color: 0x223355,
      });

      this.globe = new three.Mesh(this.globeGeometry, this.globeTexture);
      this.scene.add(this.globe);
    },

    /**
     * Draw elements into the scene
     */
    drawScene() {
      const cube = new three.BoxGeometry(1, 1, 1);
      cube.x = 0;
      cube.y = 0;
      cube.z = 0;

      const material = new three.MeshStandardMaterial({
        color: 0x223355,
      });

      const mesh = new three.Mesh(cube, material);
      this.scene.add(mesh);
    },

    drawSun() {
      // Create a light
      this.sun = new three.DirectionalLight(0xffffff, 4);
      this.sun.shadow = 1;

      this.sun.position.x = 0;
      this.sun.position.y = 10;
      this.sun.position.z = 100;

      this.sun.lookAt(new three.Vector3(0, 0, 0));
      this.scene.add(this.sun);
    },

    run() {
      this.renderer.render(this.scene, this.camera);
      const time = this.clock.getElapsedTime();

      this.sun.position.x = 100 * Math.sin(time);
      this.sun.position.z = 100 * Math.cos(time);
      console.log(Math.sin(time));

      requestAnimationFrame(this.run);
    },
  },

  mounted() {
    this.renderer = new three.WebGLRenderer({
      antialias: true,
    });

    const container = this.$refs.canvas;
    this.renderer.setSize(container.offsetWidth, container.offsetHeight);

    container.appendChild(this.renderer.domElement);

    // Setup resizing
    window.addEventListener('resize', () => {
      this.camera.aspect = container.offsetWidth / container.offsetHeight;
      this.camera.updateProjectionMatrix();

      this.renderer.setSize(container.offsetWidth, container.offsetHeight);
    }, false);

    this.scene = new three.Scene();

    // Create the camera
    this.camera = new three.PerspectiveCamera(45,
      container.offsetWidth / container.offsetHeight,
      1,
      1000,
    );

    this.camera.position.z = 10;
    this.camera.position.x = 10;
    this.camera.position.y = 10;
    this.camera.lookAt(new three.Vector3(0, 0, 0));

    this.scene.add(this.camera);

    // this.drawScene();
    this.drawGlobe();
    this.drawSun();

    this.renderer.render(this.scene, this.camera);

    this.clock = new three.Clock();
    this.run();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.globe-view {
  height: 100%;
}

.canvas {
  height: 100%;
  width: 100%;
}

</style>
