<template>
  <div class="box" ref="box"></div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
export default {
  name: "",
  data() {
    return {};
  },
  mounted() {
    this.getThreeJs();
  },
  methods: {
    getThreeJs() {
      // 创建场景
      const scene = new THREE.Scene();

      // 1、创建相机

      /**
       * PerspectiveCamera(视野角度,长宽比,近截面,远截面)
       * 视野角度是指你在显示器上能看到场景的范围；类似我们人眼视野一样；该值越大，看到的范围越大；
       * 长宽比是指物体宽（width）除以高（height）的比值；一般设置为窗口的宽高比；比例设置不当，会使得物体被压缩，或者拉伸；
       * 近截面是指在场景中能看到最近的地方，比近截面更近的地方不会被渲染（看不见）；
       * 远截面是指在场景中能看到最远的地方，比远截面更远的地方不会被渲染（看不见）；
       */
      const camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        100
      );
      // 设置相机位置
      camera.position.set(10, 10, 10);
      // 设置相机看向原点
      camera.lookAt(0, 0, 0);

      // 2、创建渲染器
      const renderer = new THREE.WebGLRenderer(); // 使用【 WebGLRenderer】构造器创建渲染器对象
      renderer.setSize(window.innerWidth, window.innerHeight); // 将输出canvas的大小调整为整个窗口的宽高比例
      const box = this.$refs.box; // 获取元素
      // 将渲染结果添加到目标元素
      box.appendChild(renderer.domElement);

      // 3、添加辅助坐标
      const axesHelper = new THREE.AxesHelper(20);
      scene.add(axesHelper);

      // 4、添加轨道控制器
      const controls = new OrbitControls(camera, renderer.domElement);
      // 设置带阻尼的惯性
      controls.enableDamping = true;
      // 设置阻尼的系数
      controls.dampingFactor = 0.05;
      // 设置自动旋转
      //   controls.autoRotate = true;

      // 5、创建球体

      // 使用【球缓冲几何体（SphereGeometry）】创建一个球体实例对象，传入三个参数，分别是半径、水平分段数、 垂直分段数；球体半径我们可以设置的稍微小一些，这样渲染出来后的效果，它几乎就是不可见的；
      const sphere = new THREE.SphereGeometry(0.0001, 32, 16);
      const sphereMaterial = new THREE.MeshBasicMaterial();
      const ball = new THREE.Mesh(sphere, sphereMaterial);
      scene.add(ball);

      // 6、创建几何体

      // 使用【立方缓冲几何体（BoxGeometry）】创建立方体，可传入 “width”、“height”、“depth” 参数设置立方体的长宽高属性
      const geometry = new THREE.BoxGeometry(2, 2, 2);
      // 使用【基础网格材质(MeshBasicMaterial)】，来创建几何体的材质；
      const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
      // 使用【网格（Mesh）】，来创建立方体实例，需要传入几何体的【形状和材质】两个参数；
      const cube = new THREE.Mesh(geometry, material);
      cube.position.set(4, 0, 0);
      // 将这个立方体添加到ball中，让ball球体作为它的父元素；
      ball.add(cube);

      // 7、渲染
      function animate() {
        // 球体旋转指定度数
        ball.rotation.y += Math.PI / 180; // + 为逆时针, - 为顺时针
        // 更新控制器
        controls.update();
        // 【requestAnimationFrame(animate)】指定浏览器在下一次循环之前执行指定函数（animate）；
        requestAnimationFrame(animate);
        // 【renderer.render(scene, camera)】用我们创建的渲染器实例对象renderer，将场景实例对象scene中内容，在相机实例对象camera中的投影，渲染在Canvas上；
        renderer.render(scene, camera);
      }
      animate();

      // 监听窗口变化
      window.addEventListener("resize", () => {
        // 重置渲染器宽高比
        renderer.setSize(window.innerWidth, window.innerHeight);
        // 重置相机宽高比
        camera.aspect = window.innerWidth / window.innerHeight;
        // 更新相机的投影矩阵
        camera.updateProjectionMatrix();
      });
    },
  },
};
</script>

<style scoped lang="scss">
.box {
  width: 100%;
  height: calc(100vh - 120px);
  overflow: hidden;
}
</style>
