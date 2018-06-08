<template>
  <canvas id="gameCanvas"></canvas>
</template>

<script>
    export default {
        mounted() {
          // Get the canvas DOM element
          this.canvas = document.getElementById('gameCanvas');
          // Load the 3D engine
          this.engine = new BABYLON.Engine(this.canvas, true, {preserveDrawingBuffer: true, stencil: true});
          this.scene = this.createScene();
          this.createSkyBox();
          this.createMaterial();
          this.setCollisions();
          var that = this;
          this.scene.registerBeforeRender(this.beforeRenderFunction);
          this.engine.runRenderLoop(function(){

            that.scene.render();
          });
            console.log('Component mounted.')
        },
        data(){
          return {
            canvas: {},
            engine: {},
            camera: {},
            light: {},
            sphere: {},
            ground: {},
            groundPlane: {},
            groundMaterial: {},
            scene: {},
            skybox: {},
            skyboxMaterial: {}
          }
        },
        methods: {
          createSkyBox: function(){
            this.skybox = BABYLON.Mesh.CreateBox("skyBox", 1000.0, this.scene);
            this.skyboxMaterial = new BABYLON.StandardMaterial("skyBox", this.scene);
            this.skyboxMaterial.backFaceCulling = false;
            this.skyboxMaterial.reflectionTexture = new BABYLON.CubeTexture("skybox/skybox", this.scene);
            this.skyboxMaterial.reflectionTexture.coordinatesMode = BABYLON.Texture.SKYBOX_MODE;
            this.skyboxMaterial.diffuseColor = new BABYLON.Color3(0, 0, 0);
            this.skyboxMaterial.specularColor = new BABYLON.Color3(0, 0, 0);
            this.skybox.material = this.skyboxMaterial;
          },
          createScene: function(){
            // Create a basic BJS Scene object
            var scene = new BABYLON.Scene(this.engine);
            // Create a FreeCamera, and set its position to {x: 0, y: 5, z: -10}
            this.camera = new BABYLON.UniversalCamera('camera1', new BABYLON.Vector3(0, 10, -10), scene);
            // Target the camera to scene origin
            this.camera.setTarget(BABYLON.Vector3.Zero());
            // Attach the camera to the canvas
            this.camera.attachControl(this.canvas, false);
            // Create a basic light, aiming 0, 1, 0 - meaning, to the sky
            this.light = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(0, 1, 0), scene);
          /*  // Create a built-in "sphere" shape; its constructor takes 6 params: name, segment, diameter, scene, updatable, sideOrientation
            this.sphere = BABYLON.Mesh.CreateSphere('sphere1', 16, 2, scene, false, BABYLON.Mesh.FRONTSIDE);
            // Move the sphere upward 1/2 of its height
            this.sphere.position.y = 1;
            */// Create a built-in "ground" shape; its constructor takes 6 params : name, width, height, subdivision, scene, updatable
            this.ground = BABYLON.Mesh.CreateGroundFromHeightMap("ground", "heightmap.jpg", 200, 200, 250, 0, 10, scene, false, this.successCallback);

            return scene;

        },
        createMaterial: function(){
          var myMaterial = new BABYLON.StandardMaterial("myMaterial", this.scene);

          myMaterial.diffuseColor = new BABYLON.Color3(1, 0, 1);
          myMaterial.specularColor = new BABYLON.Color3(0.5, 0.6, 0.87);
          myMaterial.emissiveColor = new BABYLON.Color3(1, 1, 1);
          myMaterial.ambientColor = new BABYLON.Color3(0.23, 0.98, 0.53);

          this.sphere.material = myMaterial;
          this.groundMaterial = new BABYLON.StandardMaterial("ground", this.scene);
          this.groundMaterial.diffuseTexture = new BABYLON.Texture("earth_land.jpg", this.scene);

          this.groundPlane = BABYLON.Mesh.CreatePlane("groundPlane", 200.0, this.scene);
          this.groundPlane.material = this.groundMaterial;
          this.ground.material = this.groundMaterial;
        },
        setCollisions: function(){
          this.camera.ellipsoid = new BABYLON.Vector3(1, 1, 1);
          this.scene.collisionsEnabled = true;
          this.camera.checkCollisions = true;
          this.ground.checkCollisions = true;
          /*this.sphere.checkCollisions = true;
          if(this.sphere.intersectsMesh(this.camera,false)){
            this.sphere.material.emissiveColor = new BABYLON.Color4(0, 0, 0, 0);
          }
          else {
            this.sphere.material.emissiveColor = new BABYLON.Color4(1, 1, 1, 1);
          }*/
        },
        successCallback: function(mesh){

        },
        beforeRenderFunction: function () {
            // Camera
            if (this.camera.beta < 0.1)
                this.camera.beta = 0.1;
            else if (this.camera.beta > (Math.PI / 2) * 0.9)
                this.camera.beta = (Math.PI / 2) * 0.9;

            if (this.camera.radius > 50)
                this.camera.radius = 50;

            if (this.camera.radius < 5)
                this.camera.radius = 5;
        }

    },
    created(){


      // the canvas/window resize event handler
      window.addEventListener('resize', function(){
          this.engine.resize();
      });
    }
  }
</script>
