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

	var camera, scene, renderer, group, controls, earth_texture, group_models, Wire, wire;


//CREATE_SCENE

	function createScene() {

		scene = new THREE.Scene();
		scene.fog = new THREE.FogExp2( 0x000104, 0.0000675 );
		scene.background = new THREE.TextureLoader().load( 'background_map.jpg' );
		group = new THREE.Object3D();
		group_models = new THREE.Object3D();

		camera = new THREE.PerspectiveCamera( fov, width / height, near, far );
		camera.position.set( pos_x, pos_y, pos_z );

		//let backgroundSceneMap = new THREE.TextureLoader().load( 'background_map.jpg' );
		//var backgroundSceneMesh = new THREE.Mesh( new THREE.PlaneGeometry(2, 2, 0),	new THREE.MeshBasicMaterial( { map: backgroundSceneMap } ) );

		//backgroundSceneMesh.material.depthTest = false;
		//backgroundSceneMesh.material.depthWrite = false;

		//backgroundScene = new THREE.Scene();
		//backgroundCamera = new THREE.Camera();

		//backgroundScene.add(backgroundCamera);
		//backgroundScene.add(backgroundSceneMesh);

		scene.add(group);
		scene.add(group_models);
		scene.add(camera);

	};

	function createRenderer() {

		renderer = new THREE.WebGLRenderer( { canvas: canvas, antialias: true } );

		renderer.setPixelRatio(window.devicePixelRatio > 1 ? 2 : 1);
		renderer.setSize(width, height);
		renderer.shadowMap.enabled = false;
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
			transparent: true,
			opacity: 0.94,
			color: 0x1F2A44,
			//diffuse: color_of_bright_side,
			//emissive: color_of_dark_side,
			//specular: color_of_direct_light,
			//shininess: 1,
			//shading: THREE.FlatShading
			//normalMap: new THREE.TextureLoader().load('Voda_normali.jpg'),
			//normalMapScale: 1.65,
		});

			earth_texture = new THREE.Mesh(earth_texture_geometry, earth_texture_material);
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
			side: THREE.DoubleSide,
			shininess: 0.0,
			transparent: true,
			opacity: 1.0,
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
		earth_contour_shadow_material.depthWrite = true;

		earth_contour_shadow_material.blending = THREE.SubstractiveBlending;
		var earth_contour_shadow = new THREE.Mesh( earth_contour_shadow_geometry, earth_contour_shadow_material );

		//group.add( earth_contour_shadow );
		//EARTH_CONTOURR_SHADOW_END
		//EARTH_CONTOUR
		//605
		let earth_contour_geometry = new THREE.SphereGeometry(615, 50, 50, 0, Math.PI*2, 0, Math.PI);
		let earth_contour_material = new THREE.MeshPhongMaterial({
			map: new THREE.TextureLoader().load('EarthMap_transparent.png'),
			side: THREE.DoubleSide,
			shininess: 0.0,
			transparent: true,
			opacity: 1.0,
			color: 0xffffff,
			diffuse: 0xffffff,
			emissive: 0xffffff,
			specular: 0xffffff,
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

			var earth_shadow_points_texture = earth_shadow_points_loader.load('contour2_15_1.png');
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
						if (length(v.rgb) > 0.6) discard;

						
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
			earth_shadow_points_material.depthWrite = false;
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

			var texture = loader.load('contour2_15_1.png');
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

						gl_FragColor = vec4(1.0, 1.0, 1.0, 1.0);
						gl_FragColor.a = 0.8;
						vec4 shapeData = texture2D( shape, gl_PointCoord );
						if (shapeData.a < 0.1) discard;
						gl_FragColor = gl_FragColor * shapeData;
						
					}
				`,
				transparent: true,
				lights: false,
				opacity: 1.0
				});
			var points = new THREE.Points(geom, points_material);
				//points_material.depthWrite = false;
				group.add(points);
		//EARTH_POINTS_END

	};

	function addLights() {
		var ambiColor = "#ffffff";
		var ambientLight = new THREE.AmbientLight(ambiColor, 0.5);
		scene.add(ambientLight);

	};

	function addDirLight(){
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
		directionalLight.shadow.mapSize.height = 1024;
		directionalLight.shadow.mapSize.width = 1024;
		camera.add(directionalLight);

	};

	function addClouds(){

		//CLOUDS_SPHERE_SHADOW
		let clouds_shadow_texture = new THREE.TextureLoader().load('clouds_new_1.png');
		let clouds_shadow_texture_alpha = new THREE.TextureLoader().load('clouds_new_1.png');
		let clouds_shadow_geometry = new THREE.SphereGeometry(603, 50, 50, 0, 2*Math.PI, 0, Math.PI);
		let clouds_shadow_material = new THREE.MeshLambertMaterial({
			map: clouds_shadow_texture,
			alphaMap: clouds_shadow_texture_alpha,
			blending: 1,
			side: THREE.DoubleSide,
			transparent:true,
			opacity: 0.5,
			color: 0x000000,
			emissive: 0x000000,
		});
			//clouds_material.map.generateMipalphaMaps = false;
			//clouds_material.map.magFilter = THREE.LinearFilter;
			//clouds_material.map.minFilter = THREE.LinearFilter;
			//clouds_material.needsUpdate = true;
			//clouds_material.fog = false;
		var clouds_shadow = new THREE.Mesh( clouds_shadow_geometry, clouds_shadow_material );


		group.add( clouds_shadow );
		//CLOUDS_SPHERE_SHADOW_END
		//CLOUDS_SPHERE
		let clouds_texture = new THREE.TextureLoader().load('clouds_new_1.png');
		let clouds_texture_alpha = new THREE.TextureLoader().load('clouds_new_1.png');
		let clouds_geometry = new THREE.SphereGeometry(618, 500, 500, 0, 2*Math.PI, 0, Math.PI);
		let clouds_material = new THREE.MeshLambertMaterial({
			map: clouds_texture,
			alphaMap: clouds_texture_alpha,
			blending: 2,
			side: THREE.DoubleSide,
			transparent:true,
			opacity: 0.6,
			color: 0x505050,
			emissive: 0x090909
		});
			clouds_material.map.generateMipalphaMaps = false;
			clouds_material.map.magFilter = THREE.LinearFilter;
			clouds_material.map.minFilter = THREE.LinearFilter;
			clouds_material.needsUpdate = true;
			clouds_material.fog = false;
		var clouds = new THREE.Mesh( clouds_geometry, clouds_material );


		group.add( clouds );
		//CLOUDS_SPHERE_END

	};

	function addAureole(){

		//EARTH_AUREOLE
		var customMaterial = new THREE.ShaderMaterial( 
		{
		    uniforms: 
			{ 
				"c":   { type: "f", value: 0.4 },
				"p":   { type: "f", value: 1.6 },
				glowColor: { type: "c", value: new THREE.Color(0x304a62) },
				viewVector: { type: "v3", value: camera.position }
			},
			vertexShader: `
				uniform vec3 viewVector;
				uniform float c;
				uniform float p;
				varying float intensity;
				void main() 
				{
					vec3 vNormal = normalize( normalMatrix * normal );
					vec3 vNormel = normalize( normalMatrix * viewVector );
					intensity = pow( c - dot(vNormal, vNormel), p );

					gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );
				}
			`,
			fragmentShader: `
				uniform vec3 glowColor;
				varying float intensity;
				void main() 
				{
					vec3 glow = glowColor * intensity;
					gl_FragColor = vec4( glow, 1.0 );
				}
			`,
			side: THREE.BackSide,
			blending: THREE.AdditiveBlending,
			transparent: true
		}   );
		let moonGlow_geometry = new THREE.SphereGeometry(635, 300, 300, 0, Math.PI*2, 0, Math.PI);
		var moonGlow = new THREE.Mesh( moonGlow_geometry, customMaterial );
		//moonGlow.scale.multiplyScalar(1.2);
				customMaterial.fog = false;
		customMaterial.depthWrite = false;
		scene.add( moonGlow );
		//EARTH_AUREOLE_END

	};

	function addAureole_1(){
		//EARTH_AUREOLE
		var luminiGeometry = new THREE.SphereBufferGeometry(618, 300, 150);
		var lumini_loader = new THREE.TextureLoader();
			lumini_loader.setCrossOrigin('');

		var lumini_texture = lumini_loader.load('contour2_15_1.png');
		lumini_texture.wrapS = THREE.RepeatWrapping;
		lumini_texture.wrapT = THREE.RepeatWrapping;
		lumini_texture.repeat.set(1, 1);


		var luminiMaterial = new THREE.ShaderMaterial( 
		{
			vertexColors: THREE.VertexColors,
		    uniforms: 
			{ 
				visibility: {
					value: lumini_texture
				},
				shift: {
					value: 0
				},
				size: {
					value: 45
				},
				scale: {
					value: window.innerHeight / 8
				},
				"c":   { type: "f", value: 1.0 },
				"p":   { type: "f", value: 0.95 },
				glowColor: { type: "c", value: new THREE.Color(0xffffff) },
				viewVector: { type: "v3", value: camera.position }
			},
			vertexShader: `
				uniform float scale;
				uniform float size;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 viewVector;
				uniform float c;
				uniform float p;
				varying float intensity;
				void main() 
				{
					vUv = uv;
					vColor = color;
					vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );

					vec3 vNormal = normalize( normalMatrix * normal );
					vec3 vNormel = normalize( normalMatrix * viewVector );
					intensity = pow( c - dot(vNormal, vNormel), p );

					gl_PointSize = size * ( scale / length( mvPosition.xyz ));
					gl_Position = projectionMatrix * mvPosition;
				}
			`,
			fragmentShader: `
				uniform sampler2D visibility;
				uniform float shift;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 glowColor;
				varying float intensity;

				void main() 
				{
					vec2 uv = vUv;
					uv.x += shift;
					vec4 v = texture2D(visibility, uv);
					if (length(v.rgb) > 0.6) discard;

					vec3 glow = glowColor * intensity;
					gl_FragColor = vec4( glow, 1.0 );

				}
			`,
			side: THREE.FrontSide,
			blending: THREE.AdditiveBlending,
			transparent: true
		}   );
		//luminiMaterial.uniforms.visibility.value.generateMipalphaMaps = false;
		//luminiMaterial.uniforms.visibility.value.magFilter = THREE.LinearFilter;
		//luminiMaterial.uniforms.visibility.value.minFilter = THREE.LinearFilter;
		//luminiMaterial.needsUpdate = true;
		luminiMaterial.depthWrite = true;
		var lumini = new THREE.Mesh(luminiGeometry, luminiMaterial);
		//let moonGlow_geometry = new THREE.SphereGeometry(635, 300, 300, 0, Math.PI*2, 0, Math.PI);
		//var moonGlow = new THREE.Mesh( moonGlow_geometry, customMaterial );
		//moonGlow.scale.multiplyScalar(1.2);
		group.add( lumini );
		//EARTH_AUREOLE_END

	};

	function addAureole_2(){
		//EARTH_AUREOLE
		var luminiGeometry_1 = new THREE.SphereBufferGeometry(619, 300, 150);
		var lumini_loader = new THREE.TextureLoader();
			lumini_loader.setCrossOrigin('');

		var lumini_texture = lumini_loader.load('contour2_15_1.png');
		lumini_texture.wrapS = THREE.RepeatWrapping;
		lumini_texture.wrapT = THREE.RepeatWrapping;
		lumini_texture.repeat.set(1, 1);


		var luminiMaterial_1 = new THREE.ShaderMaterial( 
		{
			vertexColors: THREE.VertexColors,
		    uniforms: 
			{ 
				visibility: {
					value: lumini_texture
				},
				shift: {
					value: 0
				},
				size: {
					value: 25
				},
				scale: {
					value: window.innerHeight / 8
				},
				"c":   { type: "f", value: 1.0 },
				"p":   { type: "f", value: 0.95 },
				glowColor: { type: "c", value: new THREE.Color(0xffffff) },
				viewVector: { type: "v3", value: camera.position }
			},
			vertexShader: `
				uniform float scale;
				uniform float size;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 viewVector;
				uniform float c;
				uniform float p;
				varying float intensity;
				void main() 
				{
					vUv = uv;
					vColor = color;
					vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );

					vec3 vNormal = normalize( normalMatrix * normal );
					vec3 vNormel = normalize( normalMatrix * viewVector );
					intensity = pow( c - dot(vNormal, vNormel), p );

					gl_PointSize = size * ( scale / length( mvPosition.xyz ));
					gl_Position = projectionMatrix * mvPosition;
				}
			`,
			fragmentShader: `
				uniform sampler2D visibility;
				uniform float shift;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 glowColor;
				varying float intensity;

				void main() 
				{
					vec2 uv = vUv;
					uv.x += shift;
					vec4 v = texture2D(visibility, uv);
					if (length(v.rgb) > 0.6) discard;

					vec3 glow = glowColor * intensity;
					gl_FragColor = vec4( glow, 1.0 );

				}
			`,
			side: THREE.FrontSide,
			blending: THREE.AdditiveBlending,
			transparent: true
		}   );
		//luminiMaterial.uniforms.visibility.value.generateMipalphaMaps = false;
		//luminiMaterial.uniforms.visibility.value.magFilter = THREE.LinearFilter;
		//luminiMaterial.uniforms.visibility.value.minFilter = THREE.LinearFilter;
		//luminiMaterial.needsUpdate = true;
		luminiMaterial_1.depthWrite = true;
		var lumini_1 = new THREE.Mesh(luminiGeometry_1, luminiMaterial_1);
		//let moonGlow_geometry = new THREE.SphereGeometry(635, 300, 300, 0, Math.PI*2, 0, Math.PI);
		//var moonGlow = new THREE.Mesh( moonGlow_geometry, customMaterial );
		//moonGlow.scale.multiplyScalar(1.2);
		group.add( lumini_1 );
		//EARTH_AUREOLE_END

	};

	function addAureole_3(){
		//EARTH_AUREOLE
		var luminiGeometry_2 = new THREE.SphereBufferGeometry(620, 300, 150);
		var lumini_loader = new THREE.TextureLoader();
			lumini_loader.setCrossOrigin('');

		var lumini_texture = lumini_loader.load('contour2_15_1.png');
		lumini_texture.wrapS = THREE.RepeatWrapping;
		lumini_texture.wrapT = THREE.RepeatWrapping;
		lumini_texture.repeat.set(1, 1);


		var luminiMaterial_2 = new THREE.ShaderMaterial( 
		{
			vertexColors: THREE.VertexColors,
		    uniforms: 
			{ 
				visibility: {
					value: lumini_texture
				},
				shift: {
					value: 0
				},
				size: {
					value: 25
				},
				scale: {
					value: window.innerHeight / 8
				},
				"c":   { type: "f", value: 1.0 },
				"p":   { type: "f", value: 0.95 },
				glowColor: { type: "c", value: new THREE.Color(0xffffff) },
				viewVector: { type: "v3", value: camera.position }
			},
			vertexShader: `
				uniform float scale;
				uniform float size;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 viewVector;
				uniform float c;
				uniform float p;
				varying float intensity;
				void main() 
				{
					vUv = uv;
					vColor = color;
					vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );

					vec3 vNormal = normalize( normalMatrix * normal );
					vec3 vNormel = normalize( normalMatrix * viewVector );
					intensity = pow( c - dot(vNormal, vNormel), p );

					gl_PointSize = size * ( scale / length( mvPosition.xyz ));
					gl_Position = projectionMatrix * mvPosition;
				}
			`,
			fragmentShader: `
				uniform sampler2D visibility;
				uniform float shift;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 glowColor;
				varying float intensity;

				void main() 
				{
					vec2 uv = vUv;
					uv.x += shift;
					vec4 v = texture2D(visibility, uv);
					if (length(v.rgb) > 0.6) discard;

					vec3 glow = glowColor * intensity;
					gl_FragColor = vec4( glow, 1.0 );

				}
			`,
			side: THREE.FrontSide,
			blending: THREE.AdditiveBlending,
			transparent: true
		}   );
		//luminiMaterial.uniforms.visibility.value.generateMipalphaMaps = false;
		//luminiMaterial.uniforms.visibility.value.magFilter = THREE.LinearFilter;
		//luminiMaterial.uniforms.visibility.value.minFilter = THREE.LinearFilter;
		//luminiMaterial.needsUpdate = true;
		luminiMaterial_2.depthWrite = true;
		var lumini_2 = new THREE.Mesh(luminiGeometry_2, luminiMaterial_2);
		//let moonGlow_geometry = new THREE.SphereGeometry(635, 300, 300, 0, Math.PI*2, 0, Math.PI);
		//var moonGlow = new THREE.Mesh( moonGlow_geometry, customMaterial );
		//moonGlow.scale.multiplyScalar(1.2);
		group.add( lumini_2 );
		//EARTH_AUREOLE_END

	};

	function addAureole_4(){
		//EARTH_AUREOLE
		var luminiGeometry_3 = new THREE.SphereBufferGeometry(612, 300, 150);
		var lumini_loader = new THREE.TextureLoader();
			lumini_loader.setCrossOrigin('');

		var lumini_texture = lumini_loader.load('contour2_15_1.png');
		lumini_texture.wrapS = THREE.RepeatWrapping;
		lumini_texture.wrapT = THREE.RepeatWrapping;
		lumini_texture.repeat.set(1, 1);


		var luminiMaterial_3 = new THREE.ShaderMaterial( 
		{
			vertexColors: THREE.VertexColors,
		    uniforms: 
			{ 
				visibility: {
					value: lumini_texture
				},
				shift: {
					value: 0
				},
				size: {
					value: 25
				},
				scale: {
					value: window.innerHeight / 8
				},
				"c":   { type: "f", value: 1.0 },
				"p":   { type: "f", value: 0.95 },
				glowColor: { type: "c", value: new THREE.Color(0xffffff) },
				viewVector: { type: "v3", value: camera.position }
			},
			vertexShader: `
				uniform float scale;
				uniform float size;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 viewVector;
				uniform float c;
				uniform float p;
				varying float intensity;
				void main() 
				{
					vUv = uv;
					vColor = color;
					vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );

					vec3 vNormal = normalize( normalMatrix * normal );
					vec3 vNormel = normalize( normalMatrix * viewVector );
					intensity = pow( c - dot(vNormal, vNormel), p );

					gl_PointSize = size * ( scale / length( mvPosition.xyz ));
					gl_Position = projectionMatrix * mvPosition;
				}
			`,
			fragmentShader: `
				uniform sampler2D visibility;
				uniform float shift;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 glowColor;
				varying float intensity;

				void main() 
				{
					vec2 uv = vUv;
					uv.x += shift;
					vec4 v = texture2D(visibility, uv);
					if (length(v.rgb) > 0.6) discard;

					vec3 glow = glowColor * intensity;
					gl_FragColor = vec4( glow, 1.0 );

				}
			`,
			side: THREE.FrontSide,
			blending: THREE.AdditiveBlending,
			transparent: true
		}   );
		//luminiMaterial.uniforms.visibility.value.generateMipalphaMaps = false;
		//luminiMaterial.uniforms.visibility.value.magFilter = THREE.LinearFilter;
		//luminiMaterial.uniforms.visibility.value.minFilter = THREE.LinearFilter;
		//luminiMaterial.needsUpdate = true;
		luminiMaterial_3.depthWrite = true;
		var lumini_3 = new THREE.Mesh(luminiGeometry_3, luminiMaterial_3);
		//let moonGlow_geometry = new THREE.SphereGeometry(635, 300, 300, 0, Math.PI*2, 0, Math.PI);
		//var moonGlow = new THREE.Mesh( moonGlow_geometry, customMaterial );
		//moonGlow.scale.multiplyScalar(1.2);
		group.add( lumini_3 );
		//EARTH_AUREOLE_END

	};

	function addAureole_5(){
		//EARTH_AUREOLE
		var luminiGeometry_4 = new THREE.SphereBufferGeometry(611, 300, 150);
		var lumini_loader = new THREE.TextureLoader();
			lumini_loader.setCrossOrigin('');

		var lumini_texture = lumini_loader.load('contour2_15_1.png');
		lumini_texture.wrapS = THREE.RepeatWrapping;
		lumini_texture.wrapT = THREE.RepeatWrapping;
		lumini_texture.repeat.set(1, 1);


		var luminiMaterial_4 = new THREE.ShaderMaterial( 
		{
			vertexColors: THREE.VertexColors,
		    uniforms: 
			{ 
				visibility: {
					value: lumini_texture
				},
				shift: {
					value: 0
				},
				size: {
					value: 25
				},
				scale: {
					value: window.innerHeight / 8
				},
				"c":   { type: "f", value: 1.0 },
				"p":   { type: "f", value: 0.95 },
				glowColor: { type: "c", value: new THREE.Color(0xffffff) },
				viewVector: { type: "v3", value: camera.position }
			},
			vertexShader: `
				uniform float scale;
				uniform float size;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 viewVector;
				uniform float c;
				uniform float p;
				varying float intensity;
				void main() 
				{
					vUv = uv;
					vColor = color;
					vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );

					vec3 vNormal = normalize( normalMatrix * normal );
					vec3 vNormel = normalize( normalMatrix * viewVector );
					intensity = pow( c - dot(vNormal, vNormel), p );

					gl_PointSize = size * ( scale / length( mvPosition.xyz ));
					gl_Position = projectionMatrix * mvPosition;
				}
			`,
			fragmentShader: `
				uniform sampler2D visibility;
				uniform float shift;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 glowColor;
				varying float intensity;

				void main() 
				{
					vec2 uv = vUv;
					uv.x += shift;
					vec4 v = texture2D(visibility, uv);
					if (length(v.rgb) > 0.6) discard;

					vec3 glow = glowColor * intensity;
					gl_FragColor = vec4( glow, 1.0 );

				}
			`,
			side: THREE.FrontSide,
			blending: THREE.AdditiveBlending,
			transparent: true
		}   );
		//luminiMaterial.uniforms.visibility.value.generateMipalphaMaps = false;
		//luminiMaterial.uniforms.visibility.value.magFilter = THREE.LinearFilter;
		//luminiMaterial.uniforms.visibility.value.minFilter = THREE.LinearFilter;
		//luminiMaterial.needsUpdate = true;
		luminiMaterial_4.depthWrite = true;
		var lumini_4 = new THREE.Mesh(luminiGeometry_4, luminiMaterial_4);
		//let moonGlow_geometry = new THREE.SphereGeometry(635, 300, 300, 0, Math.PI*2, 0, Math.PI);
		//var moonGlow = new THREE.Mesh( moonGlow_geometry, customMaterial );
		//moonGlow.scale.multiplyScalar(1.2);
		group.add( lumini_4 );
		//EARTH_AUREOLE_END

	};

	function addAureole_6(){
		//EARTH_AUREOLE
		var luminiGeometry_5 = new THREE.SphereBufferGeometry(610, 300, 150);
		var lumini_loader = new THREE.TextureLoader();
			lumini_loader.setCrossOrigin('');

		var lumini_texture = lumini_loader.load('contour2_15_1.png');
		lumini_texture.wrapS = THREE.RepeatWrapping;
		lumini_texture.wrapT = THREE.RepeatWrapping;
		lumini_texture.repeat.set(1, 1);


		var luminiMaterial_5 = new THREE.ShaderMaterial( 
		{
			vertexColors: THREE.VertexColors,
		    uniforms: 
			{ 
				visibility: {
					value: lumini_texture
				},
				shift: {
					value: 0
				},
				size: {
					value: 25
				},
				scale: {
					value: window.innerHeight / 8
				},
				"c":   { type: "f", value: 1.0 },
				"p":   { type: "f", value: 0.95 },
				glowColor: { type: "c", value: new THREE.Color(0xffffff) },
				viewVector: { type: "v3", value: camera.position }
			},
			vertexShader: `
				uniform float scale;
				uniform float size;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 viewVector;
				uniform float c;
				uniform float p;
				varying float intensity;
				void main() 
				{
					vUv = uv;
					vColor = color;
					vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );

					vec3 vNormal = normalize( normalMatrix * normal );
					vec3 vNormel = normalize( normalMatrix * viewVector );
					intensity = pow( c - dot(vNormal, vNormel), p );

					gl_PointSize = size * ( scale / length( mvPosition.xyz ));
					gl_Position = projectionMatrix * mvPosition;
				}
			`,
			fragmentShader: `
				uniform sampler2D visibility;
				uniform float shift;

				varying vec2 vUv;
				varying vec3 vColor;

				uniform vec3 glowColor;
				varying float intensity;

				void main() 
				{
					vec2 uv = vUv;
					uv.x += shift;
					vec4 v = texture2D(visibility, uv);
					if (length(v.rgb) > 0.6) discard;

					vec3 glow = glowColor * intensity;
					gl_FragColor = vec4( glow, 1.0 );

				}
			`,
			side: THREE.FrontSide,
			blending: THREE.AdditiveBlending,
			transparent: true
		}   );
		//luminiMaterial.uniforms.visibility.value.generateMipalphaMaps = false;
		//luminiMaterial.uniforms.visibility.value.magFilter = THREE.LinearFilter;
		//luminiMaterial.uniforms.visibility.value.minFilter = THREE.LinearFilter;
		//luminiMaterial.needsUpdate = true;
		luminiMaterial_5.depthWrite = true;
		var lumini_5 = new THREE.Mesh(luminiGeometry_5, luminiMaterial_5);
		//let moonGlow_geometry = new THREE.SphereGeometry(635, 300, 300, 0, Math.PI*2, 0, Math.PI);
		//var moonGlow = new THREE.Mesh( moonGlow_geometry, customMaterial );
		//moonGlow.scale.multiplyScalar(1.2);
		group.add( lumini_5 );
				let conj = new THREE.Quaternion();
		conj.copy(group.quaternion);
		conj.conjugate();

		lumini_5.quaternion.multiplyQuaternions( conj, camera.quaternion);
		//EARTH_AUREOLE_END

	};

Wire = function(){

	var geom = new THREE.CylinderGeometry(50,50,2800,40,40);
	geom.applyMatrix(new THREE.Matrix4().makeRotationX(-Math.PI/2));
	geom.mergeVertices();
	var l = geom.vertices.length;

	this.waves = [];

	for (var i=0;i<l;i++){

		var v = geom.vertices[i];

		this.waves.push({y:v.y,
			x:v.x,
			z:v.z,
			ang:Math.random()*Math.PI*2,
			amp:5 + Math.random()*15,
			speed:0.016 + Math.random()*0.032
		});

	};

	var mat = new THREE.MeshPhongMaterial({
		color: 0x0000ff,
		transparent:true,
		opacity:.8,
		shading:THREE.FlatShading,
	});

	this.mesh = new THREE.Mesh(geom, mat);
	this.mesh.receiveShadow = true;

};

	Wire.prototype.moveWaves = function (){

		var verts = this.mesh.geometry.vertices;
		var l = verts.length;

		for (var i=0; i<l; i++){

			var v = verts[i];
			var vprops = this.waves[i];
			v.x =  vprops.x + Math.cos(vprops.ang)*vprops.amp;
			v.y = vprops.y + Math.sin(vprops.ang)*vprops.amp;
			vprops.ang += vprops.speed;

		};

		this.mesh.geometry.verticesNeedUpdate=true;
		wire.mesh.rotation.z += .005;

	};

	function createWire(){

		wire = new Wire();
		wire.mesh.position.y = -600;

		scene.add(wire.mesh);

	};

	init();
	animate();
	function init() {

		createScene();
		createRenderer();
		addAureole();
		addEarthTexture();
		addEarthContour();
		addEarthPoints();
		//addAureole_1();
		//addAureole_2();
		//addAureole_3();
		//addAureole_4();
		//addAureole_5();
		//addAureole_6();
		createWire();
		addClouds();
		addLights();
		addDirLight();

	};
	function animate() {
		wire.moveWaves();
		window.requestAnimationFrame(animate);
		Render();
		//for (var i = 0; i < geometry.vertices.length; i++) {
		//	let vec_1 = geometry.vertices[i];
		//	vec_1.z= 100 * 
		//}
		//group_models.rotation.y += 0.006;
	};
	function Render() {

		//camera.rotation.y = 8*(Math.PI/180);
		//camera.rotation.x = -28*(Math.PI/180);
		//camera.rotation.z = 11*(Math.PI/180);
		//camera.position.x = -900;
		//camera.position.y = 1200;
		//camera.position.z = 2300;
		//group_models.rotation.x = 90*(Math.PI/180);
		//group_models.rotation.y = 138*(Math.PI/180);
		//group_models.rotation.z = 11*(Math.PI/180);
		//group_models.position.x--;
		//group_models.position.y--;

	//	camera.position.x=width *2 / 3;
	//	camera.position.y=500;
	//	camera.position.z=height *1 / 3;
		renderer.autoClear = false;
		renderer.clear();
		//renderer.render(backgroundScene , backgroundCamera );
		renderer.render( scene, camera );
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

