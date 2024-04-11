<!--
 * @Author: rk
 * @Description: 用户信息页面
 * @Date: 2023-03-17 17:48:40
 * @LastEditors: rk
 * @LastEditTime: 2024-04-11 09:17:30
-->

<template>
  <div class="app-container">
    <div class="content" style="background-color: #fff">
      <div id="container" class="container"></div>
    </div>
  </div>
</template>

<script>
// 导入 Three.js以及相关需要用的部分
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
export default {
  name: "VueTemplateIndex",
  data() {
    return {
      scene: null,
      light: null,
      camera: null,
      renderer: null,
      model: null,
      group: null,
      mixer: null,
    };
  },
  mounted() {
    this.init();
    // 加载模型
    this.loadGltf();
  },
  methods: {
    init() {
      let vm = this;
      //获取显示3D模型的DOM结构
      let container = document.getElementById("container");
      //导入vue监听指定元素变化的插件
      const elementResizeDetectorMaker = require("element-resize-detector");
      let erd = elementResizeDetectorMaker(); // 实例化插件
      erd.listenTo(container, (element) => {
        //这里面是监听变化后操作的
        const width = element.clientWidth;
        const height = element.clientHeight;
        // 变化之后重新设置渲染器大小
        vm.renderer.setSize(width, height);
        // 变化之后重新设置相机纵横比
        vm.camera.aspect = width / height;
        vm.camera.updateProjectionMatrix();
      });

      // 初始化场景
      this.scene = new THREE.Scene();
      // 加载环境光
      this.scene.add(new THREE.AmbientLight(0xffffff, 0.4));
      this.light = new THREE.DirectionalLight(0xffffff, 1.0);
      this.light.position.set(1, 1, 1).normalize();
      this.scene.add(this.light);

      // 加载网格
      const grid = new THREE.GridHelper(50, 10, 0x444444, 0x444444);
      //设置网格的透明度
      grid.material.opacity = 0.3;
      //设置网格是否可以透明
      grid.material.transparent = true;
      //设置网格方向为模型y轴方向
      grid.rotation.y = Math.PI / 2;
      //设置网格位置为模型中心位置向下一格
      grid.position.y = -1;
      //设置网格绕Y轴逆时针旋转
      grid.rotateY(-60);
      // 场景加载网格
      this.scene.add(grid);

      // 相机设置
      this.camera = new THREE.PerspectiveCamera(
        75,
        container.clientWidth / container.clientHeight,
        0.1,
        1000
      );
      // 设置相机位置
      this.camera.position.z = 2;
      this.camera.position.y = 0.5;
      this.camera.up = new THREE.Vector3(0, 1, 0);

      // 创建渲染器对象
      this.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true }); // 开启抗锯齿和背景透明
      this.renderer.setSize(container.clientWidth, container.clientHeight);
      // 开启之后画质直接起飞
      this.renderer.setPixelRatio(devicePixelRatio);

      const controls = new OrbitControls(this.camera, this.renderer.domElement);

      // 把所有加载到指定的dom结构上
      container.appendChild(this.renderer.domElement);
    },
    loadGltf() {
      let vm = this;
      //模型加载器，有了它才能加载.glb等后缀文件并解析
      const loader = new GLTFLoader();
      loader.load("/data/64f1b1fefe61576b46f28fbb.glb", function (gltf) {
        vm.model = gltf.scene;
        //设置模型初始的位置
        vm.model.position.set(0, -1, 0);
        //在场景中加载这个模型
        vm.scene.add(vm.model);

        //这个是读取.fbx文件的，有了它才能读取.fbx的动画文件
        const fbxLoader = new FBXLoader();

        // 加载动画文件
        // fbxLoader.load("/data/Mount_Rabbit.fbx", (animation) => {
        //   //加载动画混合器，将模型和读取.fbx文件得到的动画参数混合到一起
        //   vm.mixer = new THREE.AnimationMixer(vm.model);

        //   // 默认读取第一个动画
        //   const animClip = animation.animations[0];

        //   // 调用test方法
        //   let res = vm.Test(animClip);

        //   const action = vm.mixer.clipAction(res);
        //   // 开启动画
        //   action.play();
        // });
        function animate() {
          // 渲染器添加场景和相机
          vm.renderer.render(vm.scene, vm.camera);
          requestAnimationFrame(animate);
          if (vm.mixer) {
            vm.mixer.update(0.016);
          }
        }
        animate();
      });
    },
    // 这段是我从readyplayerme官方提供的react封装的方法上扣出来的
    Test(value) {
      const { tracks } = value;
      for (let i = 0; i < tracks.length; i += 1) {
        const hasMixamoPrefix = tracks[i].name.includes("mixamorig");
        if (hasMixamoPrefix) {
          tracks[i].name = tracks[i].name.replace("mixamorig", "");
        }
        if (tracks[i].name.includes(".position")) {
          for (let j = 0; j < tracks[i].values.length; j += 1) {
            // Scale the bound size down to match the size of the model
            // eslint-disable-next-line operator-assignment
            tracks[i].values[j] = tracks[i].values[j] * 0.01;
          }
        }
        return value;
      }
    },
  },
};
</script>

<style scoped lang="scss">
.content {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.container {
  width: 60%;
  height: 700px;
  background: white;
  border: 1px solid black;
}
.container :hover {
  cursor: pointer;
}
</style>
