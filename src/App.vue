<template>
  
</template>

<script>

import * as THREE from "three";
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
const scene = new THREE.Scene();//初始化相机
const camera = new THREE.PerspectiveCamera(75,window.innerWidth / window.innerHeight,0.1,1000);
camera.position.set(-3.23, 2.98, 4.06);
camera.updateProjectionMatrix();//初始化渲染器
const renderer = new THREE.WebGLRenderer({
        //设置抗锯齿
    antialias: true,});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 初始化控制器
const controls = new OrbitControls( camera, renderer.domElement );
controls.enableDamping = true;
export default {
  name: 'App',
  components: {
  }
}
</script>

<style>

</style>
