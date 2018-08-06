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
	var far = 20000;
	var pos_x = 0;
	var pos_y = 0;
	var pos_z = 1800;
	var color = 0x000000;
	var mouse = new THREE.Vector2(), INTERSECTED;

	var camera, scene, renderer, backgroundScene, backgroundCamera, group, controls;
	init();

//CREATE_SCENE

	function createScene() {

		scene = new THREE.Scene();
		scene.fog = new THREE.FogExp2( 0x000104, 0.0000675 );

		group = new THREE.Group();	

		camera = new THREE.PerspectiveCamera( fov, width / height, near, far );
		camera.position.set( pos_x, pos_y, pos_z );

		let backgroundSceneMap = new THREE.TextureLoader().load( 'background_map.jpg' );
		var backgroundSceneMesh = new THREE.Mesh( new THREE.PlaneGeometry(2, 2, 0),	new THREE.MeshBasicMaterial( { map: backgroundSceneMap } ) );

		backgroundSceneMesh.material.depthTest = false;
		backgroundSceneMesh.material.depthWrite = false;

		backgroundScene = new THREE.Scene();
		backgroundCamera = new THREE.Camera();

		backgroundScene.add(backgroundCamera);
		backgroundScene.add(backgroundSceneMesh);

		scene.add(group);
		scene.add(camera);

	};

	function createRenderer() {

		renderer = new THREE.WebGLRenderer( { canvas: canvas, alpha: true, antialias: true } );

		renderer.setPixelRatio(window.devicePixelRatio > 1 ? 2 : 1);
		renderer.setSize(width, height);
		renderer.shadowMap.enabled = true;
		renderer.shadowMap.type = THREE.PCFSoftShadowMap;

		controls = new OrbitControls(camera, renderer.domElement);

	};

	function addEarthTexture() {

		//let color_99 = 0x04191b;//let color_99 = 0x020f10;//let color_99 = 0x031414;//let color_99 = 0x000f11;//let color_99 = 0x010f12;
		var color_of_direct_light= 0x000c18;
		var color_of_bright_side= 0x000c18;
		var color_of_dark_side= 0x020b14;
		let earth_texture_geometry = new THREE.SphereGeometry(600, 240, 240, 0, Math.PI*2, 0, Math.PI);
		let earth_texture_material = new THREE.MeshPhongMaterial({
			map: new THREE.TextureLoader().load('Voda_normali.jpg'),
			bumpMap: new THREE.TextureLoader().load('Shum_dlya_vody_bump.jpg'),
			bumpScale: 1.65,
			shininess: 0.0,
			wireframe: false,
			transparent: false,
			opacity: 2.0,
			color: 0x000000,
			diffuse: color_of_bright_side,
			emissive: color_of_dark_side,
			specular: color_of_direct_light,
			//shininess: 1,
			//shading: THREE.FlatShading
		});

		var earth_texture = new THREE.Mesh(earth_texture_geometry, earth_texture_material);
			earth_texture.recieveShadow = false;
			earth_texture.castShadow = false;
			earth_texture_material.fog = false;
			earth_texture_material.depthWrite = true;

			//earth_texture_material.map.generateMipalphaMaps = false;
			//earth_texture_material.map.magFilter = THREE.LinearFilter;
			//earth_texture_material.map.minFilter = THREE.LinearFilter;
			earth_texture_material.needsUpdate = true;

			earth_texture.position.set(0, 0, 0);
		group.add( earth_texture );

	};

	function addEarthContour(){
		//EARTH_CONTOUR_SHADOW
		//605
		let earth_contour_shadow_geometry = new THREE.SphereGeometry(603, 50, 50, 0, Math.PI*2, 0, Math.PI);
		let earth_contour_shadow_material = new THREE.MeshPhongMaterial({
			map: new THREE.TextureLoader().load('EarthMap_transparent.png'),
			side: THREE.FrontSide,
			shininess: 0.0,
			transparent: true,
			opacity: 0.6,
			color: 0x000000,
			diffuse: 0x000000,
			emissive: 0x000000,
			specular: 0x000000,
			//specular: 0x00ff00,
			//emissive: 0x000000,
			//shininess: 1,
			//shading: THREE.FlatShading
		});

		earth_contour_shadow_material.map.generateMipalphaMaps = false;
		earth_contour_shadow_material.map.magFilter = THREE.LinearFilter;
		earth_contour_shadow_material.map.minFilter = THREE.LinearFilter;
		earth_contour_shadow_material.needsUpdate = true;
		earth_contour_shadow_material.fog = false;
		earth_contour_shadow_material.depthWrite = false;

		earth_contour_shadow_material.blending = THREE.SubstractiveBlending;
		var earth_contour_shadow = new THREE.Mesh( earth_contour_shadow_geometry, earth_contour_shadow_material );

		group.add( earth_contour_shadow );
		//EARTH_CONTOURR_SHADOW_END
		//EARTH_CONTOUR
		//605
		let earth_contour_geometry = new THREE.SphereGeometry(615, 50, 50, 0, Math.PI*2, 0, Math.PI);
		let earth_contour_material = new THREE.MeshPhongMaterial({
			map: new THREE.TextureLoader().load('EarthMap_transparent.png'),
			side: THREE.FrontSide,
			shininess: 0.0,
			transparent: true,
			opacity: 0.4,
			color: 0x000000,
			diffuse: 0x000000,
			emissive: 0x070707,
			specular: 0x070707,
			//emissive: 0x000000,
			//shininess: 1,
			//shading: THREE.FlatShading
		});

		earth_contour_material.map.generateMipalphaMaps = false;
		earth_contour_material.map.magFilter = THREE.LinearFilter;
		earth_contour_material.map.minFilter = THREE.LinearFilter;
		earth_contour_material.needsUpdate = true;
		earth_contour_material.fog = false;
		earth_contour_material.depthWrite = false;
		//earth_contour_material.recieveShadow = true;
		//earth_contour_material.castShadow = true;

		earth_contour_material.blending = THREE.SubstractiveBlending;
		var earth_contour = new THREE.Mesh( earth_contour_geometry, earth_contour_material );

		group.add( earth_contour );
		//EARTH_CONTOUR_END

	};

	function addEarthPoints(){
		//EARTH_POINTS_SHADOW
			var earth_shadow_points_geometry = new THREE.SphereBufferGeometry(603, 300, 150);

			var colors = [];
			var color = new THREE.Color(0xffffff);
			var q = ["pink", "red", "red", "red", "maroon", "maroon", "maroon"];

			var earth_shadow_points_loader = new THREE.TextureLoader();
				earth_shadow_points_loader.setCrossOrigin('');

			var earth_shadow_points_texture = earth_shadow_points_loader.load('contour2_15.png');
				earth_shadow_points_texture.wrapS = THREE.RepeatWrapping;
				earth_shadow_points_texture.wrapT = THREE.RepeatWrapping;
				earth_shadow_points_texture.repeat.set(1, 1);

			var earth_shadow_points_shape = earth_shadow_points_loader.load('030303.png');


			var earth_shadow_points_material =  new THREE.ShaderMaterial({
					vertexColors: THREE.VertexColors,
					uniforms: {
						visibility: {
							value: earth_shadow_points_texture
						},
						shift: {
							value: 0
						},
						shape: {
							value: earth_shadow_points_shape
						},
						size: {
							value: 25
						},
						scale: {
							value: window.innerHeight / 8
						}
					},
					vertexShader: `			
						uniform float scale;
						uniform float size;

						varying vec2 vUv;
						varying vec3 vColor;
			      
						void main() {
							vUv = uv;
							vColor = color;
							vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
							gl_PointSize = size * ( scale / length( mvPosition.xyz ));
							gl_Position = projectionMatrix * mvPosition;
						}
					`,
					fragmentShader: `
					uniform sampler2D visibility;
					uniform float shift;
					uniform sampler2D shape;

					varying vec2 vUv;
					varying vec3 vColor;

					void main() {
						
						vec2 uv = vUv;
						uv.x += shift;
						vec4 v = texture2D(visibility, uv);
						if (length(v.rgb) > 1.0) discard;

						
						gl_FragColor = vec4(0.1, 0.1, 0.1, 1.0);
						gl_FragColor.a = 0.0625;
						vec4 shapeData = texture2D( shape, gl_PointCoord );
						if (shapeData.a < 0.1) discard;
						gl_FragColor = gl_FragColor * shapeData;
						
					}
				`,
					transparent: true,
					lights: false
				});
			var earth_shadow_points = new THREE.Points(earth_shadow_points_geometry,earth_shadow_points_material);
			earth_shadow_points_material.depthWrite = true;
				//gl_FragColor = vec4( vColor, 1.0 );
				//group.add(earth_shadow_points);
		//EARTH_POINTS_SHADOW_END
		//EARTH_POINTS
			var geom = new THREE.SphereBufferGeometry(615, 300, 150);

			var colors = [];
			var color = new THREE.Color(0xffffff);
			var q = ["pink", "red", "red", "red", "maroon", "maroon", "maroon"];

			var loader = new THREE.TextureLoader();
				loader.setCrossOrigin('');

			var texture = loader.load('contour2_15.png');
				texture.wrapS = THREE.RepeatWrapping;
				texture.wrapT = THREE.RepeatWrapping;
				texture.repeat.set(1, 1);	

			var disk = loader.load('circle_1.png');



			var points_material = new THREE.ShaderMaterial({
				vertexColors: THREE.VertexColors,
				uniforms: {
					visibility: {
						value: texture
					},
					shift: {
						value: 0
					},
					shape: {
						value: disk
					},
					size: {
						value: 25
					},
					scale: {
						value: window.innerHeight / 8
					}
				},
				vertexShader: `			
					uniform float scale;
					uniform float size;

					varying vec2 vUv;
					varying vec3 vColor;
		      
					void main() {
						vUv = uv;
						vColor = color;
						vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
						gl_PointSize = size * ( scale / length( mvPosition.xyz ));
						gl_Position = projectionMatrix * mvPosition;
					}
				`,
				fragmentShader: `
					uniform sampler2D visibility;
					uniform float shift;
					uniform sampler2D shape;

					varying vec2 vUv;
					varying vec3 vColor;

					void main() {
						
						vec2 uv = vUv;
						uv.x += shift;
						vec4 v = texture2D(visibility, uv);
						if (length(v.rgb) > 1.0) discard;

						gl_FragColor = vec4(0.9, 0.9, 0.9, 1.0);
						gl_FragColor.a = 0.0625;
						vec4 shapeData = texture2D( shape, gl_PointCoord );
						if (shapeData.a < 0.1) discard;
						gl_FragColor = gl_FragColor * shapeData;
						
					}
				`,
				transparent: true,
				lights: false
				});
			var points = new THREE.Points(geom, points_material);
				points_material.depthWrite = false;
				group.add(points);
		//EARTH_POINTS_END

	};

	function addProvod() {

		var mtlLoader = new MTLLoader();
		mtlLoader.load('Batareya.mtl', function (materials) {

			materials.preload();

			var objLoader = new THREE.OBJLoader();
			objLoader.setMaterials(materials);
			objLoader.load('Batareya.obj', function (object) {

				group.add(object);
				object.position.set(-3000, 0, -2000);
				console.log(object);
			});

		});

	};

	function addLights() {

		//LIGHTS_AMBIENT
		var ambiColor = "#ffffff";
		var ambientLight = new THREE.AmbientLight(ambiColor, 0.5);
		ambientLight.wrapAround = true;
		scene.add(ambientLight);
		
		//LIGHTS_AMBIENT_END
		//LIGHTS_DIRECTIONAL_LIGHT
		//var pointColor = "#4C709A";
		//var directionalLightColor = new THREE.Color("hsl(29, 100%, 95%)");
		var directionalLightColor = new THREE.Color(0xffffff);
		var directionalLight = new THREE.DirectionalLight(directionalLightColor);
		directionalLight.position.set(300, 80, -2000);
		directionalLight.castShadow = true;
		directionalLight.shadow.camera.near = 0.5;
		directionalLight.shadow.camera.far = 1300;
		directionalLight.shadow.camera.left = -600;
		directionalLight.shadow.camera.right = 600;
		directionalLight.shadow.camera.top = 600;
		directionalLight.shadow.camera.bottom = -600;

		directionalLight.distance = 10;
		directionalLight.intensity = 14.0;
		directionalLight.shadow.mapSize.height = 1024;
		directionalLight.shadow.mapSize.width = 1024;
		//var target = new THREE.Object3D();
		//directionalLight.target = earth_texture;
		camera.add( directionalLight.target );
		directionalLight.target.position.set(150, 40, -2000);
		camera.add(directionalLight);
		var helper = new THREE.DirectionalLightHelper( directionalLight, 5 );

		camera.add( helper );
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

	};

	function init() {

		createScene();
		createRenderer();
		addEarthTexture();
		addEarthContour();
		addEarthPoints();
		addProvod();
		addLights();

		Render();

	};

	function Render() {

		window.requestAnimationFrame(Render);

		renderer.autoClear = false;
		renderer.clear();
		renderer.render(backgroundScene , backgroundCamera );
		renderer.render(scene, camera);

	};

	function onResize() {

		renderer.setSize(window.innerWidth, window.innerHeight);
		camera.aspect = (window.innerWidth / window.innerHeight);
		camera.updateProjectionMatrix();

	};

window.addEventListener('resize', onResize, false);

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

