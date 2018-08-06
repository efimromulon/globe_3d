<template>
	<section class="container">
    <canvas id="basicScene"></canvas>
    <script id="vertexShader" type="x-shader/x-vertex">
      precision mediump float;
      precision mediump int;

      uniform mat4 modelViewMatrix; 
      uniform mat4 projectionMatrix; 

      attribute vec3 position;
      attribute vec4 color;

      varying vec3 vPosition;

      void main() {
        vPosition = position;

        gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );
      }
    </script>
    <script id="fragmentShader" type="x-shader/x-fragment">
      precision mediump float;
      precision mediump int;

      uniform float time;

      varying vec3 vPosition;

      void main() {

        float border = 0.2;
        float radius = 0.5;
        vec4 color0 = vec4(0.0, 0.0, 0.0, 0.0);
        vec4 color1 = vec4(1.0, 1.0, 1.0, 1.0);
        vec2 m = vPosition.xy;
        float dist = radius - sqrt(m.x * m.x + m.y * m.y);
        float t = 0.0;
        if (dist > border)
          t = 1.0;
        else if (dist > 0.0)
          t = dist / border;
        gl_FragColor = mix(color0, color1, t);
      }
    </script>
	</section>
</template>
<script>
import * as THREE from 'three';
import * as MTLLoader from 'three-mtl-loader';
import * as OBJLoader from 'three-obj-loader';
let OrbitControls = require('three-orbit-controls')(THREE);
OBJLoader(THREE);
MTLLoader(THREE);
export default {
data(){
	return {

	}
},
mounted(){
	var THREE = require('three')
	var OBJLoader = require('three-obj-loader');
	var MTLLoader = require('three-mtl-loader');
	OBJLoader(THREE);
	MTLLoader(THREE);
//VARIABLES
	let canvas = document.getElementById('basicScene');
	let width = window.innerWidth;
	let height = window.innerHeight;
	var fov = 75;
	var near = 0.1;
	var far = 10000;
	var pos_x = 0;
	var pos_y = 0;
	var pos_z = 1800;
	var color = 0x000000;
	var mouse = new THREE.Vector2(), INTERSECTED;
//VARIABLES_END
//SCENE_GROUP_CAMERA
	let scene = new THREE.Scene();
	let group = new THREE.Group();	
	scene.fog = new THREE.FogExp2( 0x000104, 0.0000675 );
	var camera = new THREE.PerspectiveCamera(fov, width / height, near, far);
		camera.position.set(pos_x, pos_y, pos_z);
//SCENE_GROUP_CAMERA_END
//RENDERER
	let renderer = new THREE.WebGLRenderer({
		canvas: canvas,
		antialias: true,
	});

	renderer.setPixelRatio(window.devicePixelRatio > 1 ? 2 : 1);
	renderer.setSize(width, height);
	renderer.shadowMap.enabled = true;
	renderer.shadowMap.type = THREE.PCFSoftShadowMap;
//RENDERER_END
//CONTROLS
	var controls = new OrbitControls(camera, renderer.domElement);
	//var controls = new OrbitControls(camera, renderer.domElement);
//CONTROLS_END
//BACKGROUND_SCENE
	let backgroundSceneMap = new THREE.TextureLoader().load('background_map.jpg');

	var backgroundSceneMesh = new THREE.Mesh(
	    new THREE.PlaneGeometry(2, 2, 0),
	    new THREE.MeshBasicMaterial({
	        map: backgroundSceneMap
	    }));

	backgroundSceneMesh.material.depthTest = false;
	backgroundSceneMesh.material.depthWrite = false;

	var backgroundScene = new THREE.Scene();
	var backgroundCamera = new THREE.Camera();

	backgroundScene.add(backgroundCamera);
	backgroundScene.add(backgroundSceneMesh);
//BACKGROUND_SCENE_END
//ADD_GROUP_&_CAMERA
	scene.add(group);
	scene.add(camera);
//ADD_GROUP_&_CAMERA_END
//EARTH_TEXTURE
	//let color_99 = 0x04191b;
	//let color_99 = 0x020f10;
	//let color_99 = 0x031414;
	//let color_99 = 0x000f11;
	let color_99 = 0x010f12;
	let earth_texture_geometry = new THREE.SphereGeometry(600, 240, 240, 0, Math.PI*2, 0, Math.PI);
	let earth_texture_material = new THREE.MeshPhongMaterial({
		map: new THREE.TextureLoader().load('Voda_normali.jpg'),
		bumpMap: new THREE.TextureLoader().load('Shum_dlya_vody_bump.jpg'),
		bumpScale: 12.65,
		shininess: 0.0,
		wireframe: false,
		transparent: false,
		opacity: 1.00,
		color: color_99
		//specular: 0xff0000,
		//shininess: 1,
		//shading: THREE.FlatShading
	});

	var earth_texture = new THREE.Mesh(earth_texture_geometry, earth_texture_material);
		earth_texture.recieveShadow = false;
		earth_texture.castShadow = false;
		earth_texture_material.fog = false;
		earth_texture_material.depthWrite = false;

		earth_texture_material.map.generateMipalphaMaps = false;
		earth_texture_material.map.magFilter = THREE.LinearFilter;
		earth_texture_material.map.minFilter = THREE.LinearFilter;
		earth_texture_material.needsUpdate = true;


	group.add( earth_texture );

	var mtlLoader = new MTLLoader();
	mtlLoader.load('Batareya.mtl', function (materials) {

	    materials.preload();

	    var objLoader = new THREE.OBJLoader();
	    objLoader.setMaterials(materials);
	    objLoader.load('Batareya.obj', function (object) {

	        group.add(object);
	        object.position.y = 0;
	        object.scale.multiplyScalar(100.2);
	        console.log(object);
	    });

	});

//LIGHTS
	//LIGHTS_AMBIENT
		var ambiColor = "#ffffff";
		var ambientLight = new THREE.AmbientLight(ambiColor, 2.5);
		ambientLight.wrapAround = true;
		scene.add(ambientLight);
		
	//LIGHTS_AMBIENT_END
	//LIGHTS_DIRECTIONAL_LIGHT
		//var pointColor = "#4C709A";
		var directionalLightColor = new THREE.Color("hsl(29, 100%, 95%)");
		var directionalLight = new THREE.DirectionalLight(directionalLightColor);
		directionalLight.position.set(2000, 1200, -5600);
		directionalLight.castShadow = true;
		directionalLight.shadow.camera.near = 0.5;
		directionalLight.shadow.camera.far = 1300;
		directionalLight.shadow.camera.left = -600;
		directionalLight.shadow.camera.right = 600;
		directionalLight.shadow.camera.top = 600;
		directionalLight.shadow.camera.bottom = -600;

		directionalLight.distance = 10;
		directionalLight.intensity = 10.1;
		directionalLight.shadow.mapSize.height = 1024;
		directionalLight.shadow.mapSize.width = 1024;
		//var target = new THREE.Object3D();
		//directionalLight.target = earth_texture;
		camera.add( directionalLight.target );
		directionalLight.target.position.set(-200, -300, -4000);
		camera.add(directionalLight);

	//LIGHTS_DIRECTIONAL_LIGHT_END
	//LIGHTS_DIRECTIONAL_LIGHT
		//var pointColor = "#4C709A";
		var directionalLightColor_1 = new THREE.Color("hsl(170, 20%, 62%)");
		var directionalLight_1 = new THREE.DirectionalLight(directionalLightColor_1);
		directionalLight_1.position.set(2000, 300, -2200);
		directionalLight_1.castShadow = true;
		directionalLight_1.shadow.camera.near = 0.1;
		directionalLight_1.shadow.camera.far = 1300;
		directionalLight_1.shadow.camera.left = -600;
		directionalLight_1.shadow.camera.right = 600;
		directionalLight_1.shadow.camera.top = 600;
		directionalLight_1.shadow.camera.bottom = -600;

		directionalLight_1.distance = 10;
		directionalLight_1.intensity = 0.5;
		directionalLight_1.shadow.mapSize.height = 1024;
		directionalLight_1.shadow.mapSize.width = 1024;
		//var target = new THREE.Object3D();
		//directionalLight.target = earth_texture;
		//camera.add(directionalLight_1);
	//LIGHTS_DIRECTIONAL_LIGHT_END
	//LIGHTS_SPOT_LIGHT
		var spotLight = new THREE.SpotLight( 0xffffff );
		spotLight.position.set( -1000, 100, 1800 );

		spotLight.castShadow = true;
		spotLight.intensity = 0.75;
		spotLight.shadow.mapSize.width = 1024;
		spotLight.shadow.mapSize.height = 1024;

		spotLight.shadow.camera.near = 500;
		spotLight.shadow.camera.far = 4000;
		spotLight.shadow.camera.fov = 30;

		//camera.add( spotLight );
	function Render() {

		window.requestAnimationFrame(Render);

		renderer.autoClear = false;
		renderer.clear();
		renderer.render(backgroundScene , backgroundCamera );
		renderer.render(scene, camera);
	}

	Render();
//RENDER_END
//RESIZE
	function onResize() {

		renderer.setSize(window.innerWidth, window.innerHeight);
		camera.aspect = (window.innerWidth / window.innerHeight);
		camera.updateProjectionMatrix();

	}

	window.addEventListener('resize', onResize, false);
//RESIZE_END
}//mounted end
}
</script>
<style lang="sass">
	body
		margin: 0
		padding: 0
		height: 100vh
		width: 100vw
	.basicScene
		height: 100vh
		width: 100vw
</style>

