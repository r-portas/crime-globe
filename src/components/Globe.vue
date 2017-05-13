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
    };
  },

  methods: {
    /**
     * Draw elements into the scene
     */
    drawScene() {
      const cube = new three.BoxGeometry(1, 1, 1);
      cube.x = 0;
      cube.y = 0;
      cube.z = 0;

      const material = new three.MeshBasicMaterial({
        color: 0x223355,
      });

      const mesh = new three.Mesh(cube, material);
      this.scene.add(mesh);
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

    this.camera.position.z = 5;
    this.camera.position.x = 5;
    this.camera.position.y = 5;
    this.camera.lookAt(new three.Vector3(0, 0, 0));

    this.scene.add(this.camera);

    this.drawScene();

    this.renderer.render(this.scene, this.camera);
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
