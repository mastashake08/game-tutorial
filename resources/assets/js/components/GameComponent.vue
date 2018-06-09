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
           var that = this;
          this.scene.registerBeforeRender(this.beforeRenderFunction);
          this.engine.runRenderLoop(function(){

            that.scene.render();
          });
            console.log('Component mounted.')


                  // the canvas/window resize event handler
                  window.addEventListener('resize', function(){
                      this.engine.resize();
                  });
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
            skyboxMaterial: {},
            shadowGenerator: {},
            assetsManager: {},
            meshTask: {}
          }
        },
        methods: {
          createSkyBox: function(scene){
            this.skybox = BABYLON.Mesh.CreateBox("skyBox", 1000.0, scene);
            this.skyboxMaterial = new BABYLON.StandardMaterial("skyBox", scene);
            this.skyboxMaterial.backFaceCulling = false;
            this.skyboxMaterial.disableLighting = true;
            this.skybox.material = this.skyboxMaterial;
            this.skybox.infiniteDistance = true;
            this.skyboxMaterial.disableLighting = true;
            this.skyboxMaterial.reflectionTexture = new BABYLON.CubeTexture("skybox/skybox", scene);
            this.skyboxMaterial.reflectionTexture.coordinatesMode = BABYLON.Texture.SKYBOX_MODE;


          },
          createScene: function(){
            // Create a basic BJS Scene object
            var scene = new BABYLON.Scene(this.engine);

            // Create a FreeCamera, and set its position to {x: 0, y: 5, z: -10}
            this.camera = new BABYLON.FreeCamera('camera1', new BABYLON.Vector3(10, 5, -10), scene);
            // Target the camera to scene origin
            this.camera.setTarget(BABYLON.Vector3.Zero());
            // Attach the camera to the canvas
            this.camera.attachControl(this.canvas, false);
            // Create a basic light, aiming 0, 1, 0 - meaning, to the sky
            this.light = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(0, 1, 0), scene);
            this.light.shadowsEnabled = true;
          // Create a built-in "ground" shape; its constructor takes 6 params : name, width, height, subdivision, scene, updatable
            this.ground = BABYLON.Mesh.CreateGroundFromHeightMap("ground", "heightmap.jpg",  200, 200, 250, 0, 10, scene, false, this.successCallback);
            this.createMaterial(scene);
            //this.createSkyBox(scene);
            this.ground.receiveShadows = true;
            this.setCollisions();

            return scene;

        },
        createMaterial: function(scene){

         this.groundMaterial = new BABYLON.StandardMaterial("ground", scene);
         this.groundMaterial.diffuseTexture = new BABYLON.Texture("earth_land.jpg", scene);

         this.assetsManager = new BABYLON.AssetsManager(scene);
         this.meshTask = this.assetsManager.addMeshTask("dragon task", "", "./meshes/dragon/", "alduin-dragon.babylon");

         this.meshTask.onSuccess = function (task) {
              task.loadedMeshes[0].position = new BABYLON.Vector3(0,0,1);
              task.loadedMeshes[0].scaling = new BABYLON.Vector3(0.02, 0.02, 0.02);
          }

          this.assetsManager.onProgress = function(remainingCount, totalCount, lastFinishedTask) {
            console.log("loading....");
              this.engine.loadingUIText = 'We are loading the scene. ' + remainingCount + ' out of ' + totalCount + ' items still need to be loaded.';
          };

          this.assetsManager.onFinish = function(tasks) {
              this.engine.runRenderLoop(function() {
                  scene.render();
              });
          };
          this.assetsManager.load();
         this.ground.material = this.groundMaterial;
         //BABYLON.SceneLoader.Append("./meshes/dragon/", "alduin-dragon.babylon", scene, function (scene) {
             // do something with the scene
          //   console.log(scene.meshes);
            // scene.meshes[1].scaling = new BABYLON.Vector3(0.02, 0.02, 0.02);
             //scene.meshes[1].position = new BABYLON.Vector3(0,0,1);
        // });
        },
        setCollisions: function(){
          this.camera.ellipsoid = new BABYLON.Vector3(1, 1, 1);
          this.scene.collisionsEnabled = true;
          this.camera.checkCollisions = true;
          this.ground.checkCollisions = true;

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

    }
  }
</script>
