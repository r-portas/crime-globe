<template>
  <div class="globe-view">
    <div ref="canvas" class="canvas"></div>
  </div>
</template>

<script>
import * as three from 'three';
import { SpriteText2D } from 'three-text2d';
import OrbitControls from 'orbit-controls-es6';
import crimes from '../crimes.json';

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
      pointLight: null,

      globe: null,
      globeGeometry: null,
      globeTexture: null,
      globeRadius: 5000,

      crimeSpots: [],

      // The light representing the sun
      sun: null,

      clock: null,
    };
  },

  methods: {

    /**
     * Converts to the sphere coordinates
     * @param coords - A (lat, lng) array
     */
    convertToSphere(coords) {
      const phi = (90 - coords[0]) * (Math.PI / 180);
      const theta = (coords[1] + 180) * (Math.PI / 180);
      const offset = 0;

      const x = -((this.globeRadius + offset) * Math.sin(phi) * Math.cos(theta));
      const y = ((this.globeRadius + offset) * Math.cos(phi));
      const z = ((this.globeRadius + offset) * Math.sin(phi) * Math.sin(theta));

      return new three.Vector3(x, y, z);
    },

    drawAllCrime() {
      for (let i = 0; i < crimes.length; i += 1) {
        const crime = crimes[i];
        const crimeSpot = this.drawCrimeSpot(crime);
        this.crimeSpots.push(crimeSpot);
      }
    },

    drawCrimeSpot(crime) {
      const worldCoords = this.convertToSphere(crime.coords);

      const radius = crime.crime_rate * 2;

      const spotGeom = new three.SphereGeometry(radius, 10, 10);

      const color = new three.Color(crime.crime_rate / 87, 0, 0);
      const spotMaterial = new three.MeshBasicMaterial({ color });
      const spot = new three.Mesh(spotGeom, spotMaterial);

      this.scene.add(spot);

      spot.position.x = worldCoords.x;
      spot.position.y = worldCoords.y;
      spot.position.z = worldCoords.z;

      return spot;
    },

    drawLabels() {
      for (let i = 0; i < crimes.length; i += 1) {
        const crime = crimes[i];
        const worldCoords = this.convertToSphere(crime.coords);
        const sprite = new SpriteText2D(crime.city_name, { font: '200px Arial', antialias: true });
        sprite.position.x = worldCoords.x;
        sprite.position.y = worldCoords.y;
        sprite.position.z = worldCoords.z;

        this.scene.add(sprite);
      }
    },

    drawGlobe() {
      this.globeGeometry = new three.SphereGeometry(this.globeRadius, 32, 32);
      this.globeGeometry.x = 0;
      this.globeGeometry.y = 0;
      this.globeGeometry.z = 0;

      this.globeTexture = new three.MeshStandardMaterial({
        color: 0x1C6BA0,
      });

      const loader = new three.TextureLoader();

      this.globeTexture.map = loader.load('./static/earthmap8k.jpg');

      this.globeTexture.bumpMap = loader.load('./static/elev_bump_8k.jpg');
      this.globeTexture.bumpScale = 0.2;

      this.globeTexture.roughnessMap = loader.load('./static/water_8k.jpg');
      // this.globeTexture.specular = new three.Color('grey');
      this.globeTexture.metalness = 0;

      this.globe = new three.Mesh(this.globeGeometry, this.globeTexture);
      this.scene.add(this.globe);
    },

    drawSun() {
      // Create a light
      this.sun = new three.DirectionalLight(0xffffff, 2);

      this.sun.position.x = 0;
      this.sun.position.y = 10;
      this.sun.position.z = 100000;

      this.sun.lookAt(new three.Vector3(0, 0, 0));
      this.scene.add(this.sun);
    },

    drawCamera(container) {
      // Create the camera
      this.camera = new three.PerspectiveCamera(45,
        container.offsetWidth / container.offsetHeight,
        1,
        100000,
      );


      // Create the ambient light
      this.pointLight = new three.PointLight(0xaaaaaa, 3, 100);
      this.pointLight.position.set(5000, 3000, 5000);
      // this.camera.add(this.pointLight);
      this.scene.add(this.pointLight);

      this.camera.position.set(10000, 10000, 10000);
      this.camera.lookAt(new three.Vector3(0, 0, 0));

      this.scene.add(this.camera);
    },

    run() {
      this.renderer.render(this.scene, this.camera);
      const time = this.clock.getElapsedTime();

      this.sun.position.x = 100 * Math.sin(time);
      this.sun.position.z = 100 * Math.cos(time);

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

    // this.drawScene();
    this.drawCamera(container);
    this.drawGlobe();
    this.drawSun();

    // this.drawLabels();
    this.drawAllCrime();

    // Setup the mouse movement
    const controls = new OrbitControls(this.camera, this.renderer.domElement);
    controls.addEventListener('change', () => {
      this.renderer.render(this.scene, this.camera);
      this.pointLight.position.copy(this.camera.position);
    });

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
