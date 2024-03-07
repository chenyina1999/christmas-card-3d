<template>
    <div 
        class="scenes" 
        style="
            position: fixed;
            left: 0;
            top: 0; 
            z-index: 10;
            pointer-events: none;
            transition: all 1s;"
        :style="{
            transform: `translate3d(0, ${-index * 100}vh, 0)`
        }">
        <div v-for="item in scenes" :key="item.index" style="width: 100vh; height: 100vh;">
            <h1 style="padding: 100vh 50px; font-size: 50px;color: #fff;">{{ item.text }}</h1>
        </div>
    </div>
</template>

<script setup>
let scenes = ref([]);
let index = ref(0);
import * as THREE from "three";
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";//rebe加载器	
import { Water } from "three/examples/jsm/objects/Water2";
import gsap from "gsap";
import {ref} from 'vue';
const scene = new THREE.Scene();//初始化相机
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(-3.23, 3, 4.06);
camera.updateProjectionMatrix();//初始化渲染器
const renderer = new THREE.WebGLRenderer({
    //设置抗锯齿
    antialias: true,
    physicallyCorrectLights: true
});
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

const axesHelper = new THREE.AxesHelper(5);
// size 表示代表轴的线段长度，默认为1
scene.add(axesHelper);
// 初始化控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.target.set(-8, 2, 0);
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
            const model = gltf.scene;
            model.traverse((child) => {
                if(child.name == "Plane") {
                    child.visible = false;
                }
                // 允许物体接收阴影
                if(child.isMesh) {
                    child.castShadow = true;
                    child.receiveShadow = true;
                }
            })

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
// 添加环境光源
function setDirectionLight() {
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(0, 50, 0);
    scene.add(light);
}

// 添加点光源
function setLightHouse() {
    const pointlight = new THREE.PointLight(0xffffff, 10)
    pointlight.position.set(0.1, 2.2, 0);
    pointlight.castShadow = true;
    scene.add(pointlight);
}

// 创建水面
makeWater();
// 添加点光源
setLightHouse();
// 添加环境光源
setDirectionLight();

// 创建三个小球 放在 点光源
// 然后循环跳动
let radius = 3;

let lightGroup = new THREE.Group();
lightGroup.position.set(-8, 3.5, -1.5);
let sphereMeshArr = [];
for(let i = 0; i < 3; i++) {
    let shpereGeometry = new THREE.SphereGeometry(0.2, 32, 32);
    let sphereMaterial = new THREE.MeshStandardMaterial({
        color: 0xffffff,
        emissive: 0xffffff,
        emissiveInensity: 10,
        
    });
    let sphereMesh = new THREE.Mesh(shpereGeometry, sphereMaterial);
    sphereMeshArr.push(sphereMesh);
    sphereMesh.position.set(
        radius * Math.cos((i * 2 * Math.PI) / 3), // (2pi/3)*i
        Math.cos((i * 2 * Math.PI) / 3),
        radius * Math.sin((i * 2 * Math.PI) / 3)
    )

    let pointlight = new THREE.PointLight(0xffffff, 40);
    sphereMesh.add(pointlight);
    lightGroup.add(sphereMesh);
};

scene.add(lightGroup);




(async function () {
    // 添加模型
    let gltfUrl = './model/scene.glb';
    await loadGLTFModel(gltfUrl);
    // 添加环境贴图
    let envUrl = './textures/sky.hdr';
    await loadRGBEModel(envUrl);
    
}());

function render() {
    requestAnimationFrame(render);
    renderer.render(scene, camera);
    controls.update();
}

// 使用补间函数，从0到360°，使灯泡旋转
let options = {
    angle: 0,
}
gsap.to(options, {
    angle: Math.PI * 2,
    duration: 10,
    repeat: -1,
    ease: "linear",
    onUpdate: () => {
        lightGroup.rotation.y = options.angle;
        sphereMeshArr.forEach((item, index) => {
            item.position.set(
                radius * Math.cos((index * 2 * Math.PI) / 3),
                Math.cos((index * 2 * Math.PI) / 3 + options.angle * 5),
                radius * Math.sin((index * 2 * Math.PI) / 3)
            )
        })
    }
})

// 使用补间动画移动相机
let timeLine1 = gsap.timeline();
// 使用补间动画移动target
let timeLine2 = gsap.timeline();

// 定义相机移动函数
function translateCamera(position, target) {
    // 相机移动到position的位置，历时一秒、使用的效果是power2.inOut
    timeLine1.to(camera.position, {
        x: position.x,
        y: position.y,
        z: position.z,
        duration: 1,
        ease: "power2.inOut",
    });
    // 控制器的目标，要聚焦的点
    timeLine2.to(controls.target, {
        x: target.x,
        y: target.y,
        z: target.z,
        duration: 1,
        ease: "power2.inOut",
    })
}

// 相机位置与文字切屏

scenes.value = [
    {
        text: "圣诞快乐",
        callback: () => {
            // 执行函数切换位置
            translateCamera(
                new THREE.Vector3(-3.23, 3, 4.06),
                new THREE.Vector3(-8, 2, 0)
            );
        },
    },
    {
        text: "感谢在这么大的世界里遇见了你",
        callback: () => {
            // 执行函数切换位置
            translateCamera(
                new THREE.Vector3(7, 0, 23),
                new THREE.Vector3(0, 0, 0)
            );
        }
    },
    {
        text: "愿与你探寻世界的每一个角落",
        callback: () => {
            translateCamera(
                new THREE.Vector3(10, 3, 0),
                new THREE.Vector3(5, 2, 0) // 宝藏的位置
            );
        }
    },
    {
        text: "愿将天上的星星送给你",
        callback: () => {
            translateCamera(
                new THREE.Vector3(7, 0, 23),
                new THREE.Vector3(0, 0, 0)
            );
            makeHeart();
        }
    },
    {
        text: "愿疫情结束，大家健康快乐",
        callback: () => {
            translateCamera(
                new THREE.Vector3(-20, 1.3, 6.6),
                new THREE.Vector3(5, 2, 0)
            );
            // restoreHeart();
        }
    }   
];

let isAnimate = false;
// 监听鼠标滚轮事件
window.addEventListener("wheel", (e) => {
    if(isAnimate) return;
    console.log(scenes.value, index.value);
    isAnimate = true;
    if(e.deltaY > 0) {
        index.value++;
        if(index.value > scenes.value.length -1) {
            index.value = 0;
            restoreHeart();
        }
    }
    scenes.value[index.value].callback();
    setTimeout(() => {
        isAnimate = false;
    }, 1000);
})

//  实例化漫天星星化成爱心
let starsInstance = new THREE.InstancedMesh(
    new THREE.SphereGeometry(0.1, 32, 32),
    new THREE.MeshStandardMaterial({
        color: 0xffffff,
        emissive: 0xffffff,
        emissiveIntensity: 10
    }), 
    100
);

// 星星随机到天上
let starsArr = [];
let endArr = [];
for(let i = 0; i < 100; i++) {
    // -50到50的区间
    let x = Math.random() * 100 - 50;
    let y= Math.random() * 100 - 50;
    let z = Math.random() * 100 - 50;
    starsArr.push(new THREE.Vector3(x, y, z));

    let matrix = new THREE.Matrix4();
    matrix.setPosition(x, y, z);
    starsInstance.setMatrixAt(i, matrix);

}
scene.add(starsInstance);

// 创建爱心路径
let heartShape = new THREE.Shape();
heartShape.moveTo(25, 25);
heartShape.bezierCurveTo(25, 25, 20, 0, 0, 0);
heartShape.bezierCurveTo(-30, 0, -30, 35, -30, 35);
heartShape.bezierCurveTo(-30, 55, -10, 77, 25, 95);
heartShape.bezierCurveTo(60, 77, 80, 55, 80, 35);
heartShape.bezierCurveTo(80, 35, 80, 0, 50, 0);
heartShape.bezierCurveTo(35, 0, 25, 25, 25, 25);

//　根据爱心路径获取点
let center = new THREE.Vector3(0, 2, 10);
for(let i = 0; i < 100; i++) {
    let point = heartShape.getPoint(i / 100);
    endArr.push(new THREE.Vector3(point.x * 0.1, point.y * 0.1 + center.y, point.z));
}

// how to understand
// 创建爱心动画
function makeHeart() {
    let params = {
        time: 0
    };
    gsap.to(
        params,
        {
            time: 1,
            duration: 1,
            onUpdate: () => {
                for(let i = 0; i < 100; i++) { 
                    let x = starsArr[i].x + (endArr[i].x - starsArr[i].x) * params.time;
                    let y = starsArr[i].y + (endArr[i].y - starsArr[i].y) * params.time;
                    let z = starsArr[i].z + (endArr[i].z - starsArr[i].z) * params.time;
                    let matrix = new THREE.Matrix4();
                    matrix.setPosition(x, y, z);
                    starsInstance.setMatrixAt(i, matrix);

                }
                starsInstance.instanceMatrix.needsUpdate = true;
            }
        }
    )
}

function restoreHeart() {
  let params = {
    time: 0,
  };

  gsap.to(params, {
    time: 1,
    duration: 1,
    onUpdate: () => {
      for (let i = 0; i < 100; i++) {
        let x = endArr[i].x + (starsArr[i].x - endArr[i].x) * params.time;
        let y = endArr[i].y + (starsArr[i].y - endArr[i].y) * params.time;
        let z = endArr[i].z + (starsArr[i].z - endArr[i].z) * params.time;
        let matrix = new THREE.Matrix4();
        matrix.setPosition(x, y, z);
        starsInstance.setMatrixAt(i, matrix);
      }
      starsInstance.instanceMatrix.needsUpdate = true;
    },
  });
}


render();
// export default {
//     name: 'App',
//     components: {
//     }
// }
</script>

<style>
.scenes {

}
</style>
