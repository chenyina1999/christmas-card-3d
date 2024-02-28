<template>
  
</template>

<script>

import * as THREE from "three";
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader }  from 'three/examples/jsm/loaders/DRACOLoader.js';
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";//rebe加载器	
import { Water } from "three/examples/jsm/objects/Water2";
const scene = new THREE.Scene();//初始化相机
const camera = new THREE.PerspectiveCamera(75,window.innerWidth / window.innerHeight,0.1,1000);
camera.position.set(-3.23, 2.98, 4.06);
camera.updateProjectionMatrix();//初始化渲染器
const renderer = new THREE.WebGLRenderer({
        //设置抗锯齿
    antialias: true,});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

const axesHelper = new THREE.AxesHelper( 5 ); 
// size 表示代表轴的线段长度，默认为1
scene.add(axesHelper);
// 初始化控制器
const controls = new OrbitControls( camera, renderer.domElement );
controls.enableDamping = true;
// 添加平行光
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(0, 50, 0);
scene.add(light);

// 初始化loader
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("./draco/");
dracoLoader.preload();
const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(dracoLoader);



function loadGLTFModel(url) {
    return new Promise((resolve, reject) => {
        gltfLoader.load(url, (gltf) => {
            console.log(gltf);
             // 缩放模型大小
            // gltf.scene.scale.set(0.1, 0.1, 0.1);
            scene.add(gltf.scene);
            resolve(); // onLoad callback
        }),
        undefined,  //onProcess callback 不需要可以传undefined
        (error) => { // onError callback
            reject(error); 
        }
    })
}

let rgbeLoader = new RGBELoader();
function loadRGBEModel(url) {
    return new Promise((resolve, reject) => {
        rgbeLoader.load(url, (envMap) => {
            // 设置球形贴图
            envMap.mapping = THREE.EquirectangularReflectionMapping;
            // 设置环境贴图
            scene.background = envMap;
            // 设置环境贴图
            scene.environment = envMap;
            resolve();
            // 设置plane的环境贴图
            // planeMaterial.envMap = envMap;  
            // animate();
        }),
        undefined,
        reject();
    })
}


// 创建水面
function makeWater() {
    const waterGeometry = new THREE.CircleGeometry(300, 32);
    const water = new Water(waterGeometry, {
        textureWidth: 1024,
        textureHeight: 1024,
        color: 0xeeeeff,
        flowDirection: new THREE.Vector2(1, 1),
        scale: 100
    });
    water.rotation.x = -Math.PI / 2;
    water.position.y = -0.4;
    scene.add(water);
}
makeWater();

(async function () {
   
    let gltfUrl = './model/scene.glb';
    console.log("gltfUrl", gltfUrl);
    await loadGLTFModel(gltfUrl);
    let envUrl = './textures/sky.hdr';
    await loadRGBEModel(envUrl);
   


}());

function render() {
    requestAnimationFrame(render);
    renderer.render(scene, camera);
    controls.update();
}
render();
export default {
  name: 'App',
  components: {
  }
}
</script>

<style>

</style>
