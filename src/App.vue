<template>
  <h1>3D Vue Workshop Demo</h1>
  <!-- Three.js Canvas will be attached here in init()-->
</template>

<script>
import * as THREE from "three";

export default {
  name: "app",
  data: function() {
    return {
      // Scene: What and where to render.
      scene: new THREE.Scene(),

      // Camera: Where to view from.
      camera: new THREE.PerspectiveCamera(
        75, // Field of View
        window.innerWidth / window.innerHeight, //Aspect Ratio
        0.1, // Near Clipping Plane
        1000 // Far Clipping Plane
      ),

      // Renderer: Displays scenes.
      renderer: new THREE.WebGLRenderer(),

      // Cube: cube object to render.
      cube: {},

      // Light: gives scene light sources.
      light1: new THREE.PointLight(0xffffff, 1, 4000),
      light2: new THREE.PointLight(0xffffff, 1, 4000),
      lightAmbient: new THREE.AmbientLight(0x404040),

      // Audio Input needed for animate loop.
      analyser: {},
      audioArray: []
    };
  },
  methods: {
    // init does inital set-up for three.js rendering
    init: function() {
      // Canvas: Render area in the window.
      this.$data.renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(this.$data.renderer.domElement); // attach to html

      // camera defaults to middle, move it out a bit
      this.$data.camera.position.z = 5;
    },
    // createCube creates a cube & adds to canvas
    createCube: function() {
      // vertices and faces
      let geometry = new THREE.BoxGeometry(1, 1, 1);

      // material to display
      let material = new THREE.MeshPhongMaterial({ color: 0xffffff });

      // applies material & geometry
      this.$data.cube = new THREE.Mesh(geometry, material);

      this.$data.scene.add(this.$data.cube);
    },
    animate: function() {
      // creates a loop 60 times/sec of the animate function
      requestAnimationFrame(this.animate);

      // increment done to cube, 60 times/sec
      this.$data.cube.rotation.x += 0.02;
      this.$data.cube.rotation.y += 0.01;

      // change color depending on audio values
      this.$data.analyser.getByteFrequencyData(this.$data.dataArray);
      let redVal = this.$data.dataArray[3] / 255;
      let greenVal = this.$data.dataArray[5] / 255;
      let blueVal = this.$data.dataArray[7] / 255;
      this.cube.material.color.setRGB(redVal, greenVal, blueVal);

      // render scene with camera.
      this.$data.renderer.render(this.$data.scene, this.$data.camera);
    },
    // addLight creates three different lights & adds to scene
    addLight: function() {
      this.$data.light1.position.set(-100, 800, 800);
      this.$data.light2.position.set(50, 0, 0);
      this.$data.scene.add(
        this.$data.light1,
        this.$data.light2,
        this.$data.lightAmbient
      );
    },
    addAudioInput: function() {
      // AudioContext: provides interface for source & analyser to communicate through
      let audioCtx = new (window.AudioContext || window.webkitAudioContext)();

      // UserMedia: used inside Promise below to get audio input from user.
      let userMedia =
        navigator.webkitGetUserMedia ||
        navigator.mozGetUserMedia ||
        navigator.msGetUserMedia; // varies depending on browser type
      let source; // source of audio input (defined later)

      // Analyser: will process audio and give values.
      this.$data.analyser = audioCtx.createAnalyser();
      this.$data.analyser.smoothingTimeConstant = 0.9; // smoothes audio values.
      this.$data.analyser.fftSize = 32; // size of AudioArray

      navigator.mediaDevices.getUserMedia = constraints =>
        new Promise(function(resolve, reject) {
          // .call will execute immediately and resolve with an
          // audio stream or error (as seen below)
          userMedia.call(navigator, constraints, resolve, reject);
        });

      navigator.mediaDevices
        .getUserMedia({ audio: true }) // ask for audio permission (constraints)
        .then(
          function(stream) {
            // if successful, given an audio stream.
            source = audioCtx.createMediaStreamSource(stream);
            source.connect(this.$data.analyser);
          }.bind(this) // bind allows function access to 'this'
        )
        .catch(function(err) {
          console.log(err);
        });

      let length = this.$data.analyser.frequencyBinCount;
      this.$data.dataArray = new Uint8Array(length);
    }
  },
  // called when App component mounts.
  mounted: function() {
    this.init();
    this.createCube();
    this.addLight();
    this.addAudioInput();
    this.animate();
  }
};
</script>
