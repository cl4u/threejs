<!--
 * @Author: rk
 * @Description: 角色管理页面
 * @Date: 2023-03-17 17:48:40
 * @LastEditors: rk
 * @LastEditTime: 2024-04-10 17:25:40
-->

<template>
  <div id="role"></div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { LineMaterial } from "three/examples/jsm/lines/LineMaterial";
import { Line2 } from "three/examples/jsm/lines/Line2";
import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass";
import { ShaderPass } from "three/examples/jsm/postprocessing/ShaderPass";
import { UnrealBloomPass } from "three/examples/jsm/postprocessing/UnrealBloomPass";
import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer";
import { FXAAShader } from "three/examples/jsm/shaders/FXAAShader";
import * as dat from "three/examples/jsm/libs/lil-gui.module.min.js";
import * as d3 from "d3";
import { MeshLine, MeshLineMaterial, MeshLineRaycast } from "three.meshline";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";

//场景  ShaderPass UnrealBloomPass  EffectComposer
let scene;
//渲染器
let renderer;
//相机
let camera;
//地图
let map;
let ambientLight;
//贴图
let material;

//混合器
let composer;
const ENTIRE_SCENE = 0, // 全部的，整个的场景
  BLOOM_SCENE = 1; // 光晕场景
const bloomLayer = new THREE.Layers(); // 光晕层次-创建一个图层对象
bloomLayer.set(BLOOM_SCENE); // 先把光晕层次设置光晕场景的层次1
const darkMaterial = new THREE.MeshBasicMaterial({ color: "black" }); // 跟辉光光晕有关的变量

const materials = {}; // 跟辉光光晕有关的变量
const params = {
  exposure: 0, // 暴露
  bloomStrength: 0.78, // 光晕强度
  bloomThreshold: 0, // 光晕阈值
  bloomRadius: 0.1, // 光晕半径
};
let bloomPass;
let finalComposer;
let bloomComposer;
let controller;
let line2; //线条
let province;
const newArr = function (arr) {
  return arr.reduce(
    (pre, cur) => pre.concat(Array.isArray(cur) ? newArr(cur) : cur),
    []
  );
};
let gui = {
  offsetX: 0,
  offsetY: 0,
  repeatX: 0.006,
  repeatY: 0.006,
  rotation: -0.1,
  centerX: 0.508,
  centerY: 0.398,
  RepeatWrapping: false,
  tr: 0,
};
let guisd = new dat.GUI();
let raycaster;
let mouse = {
  x: 0,
  y: 0,
};
let canvasPlane;
let special = [];
let canvas;
//更新纹理贴图的方法
function updateUV() {
  material.map.matrix
    .identity() //矩阵重置
    .translate(-gui.centerX, -gui.centerY) //设置中心点
    .rotate(gui.rotation) // 旋转
    .scale(gui.repeatX, gui.repeatY) //缩放
    .translate(gui.centerX, gui.centerY) //设置中心点
    .translate(gui.offsetX, gui.offsetY); //偏移
}
export default {
  name: "VueTemplateIndex",
  data() {
    return {
      centerX: 106.5525,
      centerY: 34.3227,
      AnimationFrameId: null,
      materials: null,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      // 创建场景
      this.getScene();
      // 创建渲染器
      this.getRenderer();
      // 创建相机
      this.getCamera();
      // 导入模型;
      this.GetGLTFLoader();
      //控制器
      this.setController();
      //获取地图数据并创建地图
      this.getMapData();
      //分层渲染
      this.feng();
      //创建射线
      this.setRaycaster();
      //创建灯光
      this.setLight();
      //启动
      this.animate();

      window.addEventListener("resize", function () {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
      });
    },
    // 场景
    getScene() {
      scene = new THREE.Scene();
      //设置背景
      new THREE.TextureLoader().load("/data/sceneBack.png", (texture) => {
        scene.background = texture;
      });
    },
    setRaycaster() {
      let onMouseMove = (event) => {
        var Sx = event.clientX; //鼠标单击位置横坐标
        var Sy = event.clientY; //鼠标单击位置纵坐标
        //屏幕坐标转WebGL标准设备坐标
        var x = (Sx / window.innerWidth) * 2 - 1; //WebGL标准设备横坐标
        var y = -(Sy / window.innerHeight) * 2 + 1; //WebGL标准设备纵坐标
        //创建一个射线投射器`Raycaster`
        var raycaster = new THREE.Raycaster();
        //通过鼠标单击位置标准设备坐标和相机参数计算射线投射器`Raycaster`的射线属性.ray
        raycaster.setFromCamera(new THREE.Vector2(x, y), camera);
        //返回.intersectObjects()参数中射线选中的网格模型对象
        // 未选中对象返回空数组[],选中一个数组1个元素，选中两个数组两个元素
        //加上true 代表遍历子元素
        var intersects = raycaster.intersectObjects(scene.children, true);
        // intersects.length大于0说明，说明选中了模型
        if (intersects.length > 0) {
          if (intersects[0].object.name == "bingmayong") {
            let bmyText =
              "兵马俑，亦简称秦兵马俑或秦俑，是第一批全国重点文物保护单位、第一批中国世界遗产，位于今陕西省西安市临潼区秦始皇陵以东1.5千米处的兵马俑坑内。";

            this.setCanvas(bmyText, [
              intersects[0].point.x,
              intersects[0].point.y,
            ]);
          } else if (intersects[0].object.name.includes("Object")) {
            let ggText =
              "故宫是中国明清两代的皇家宫殿，旧称紫禁城，位于北京中轴线的中心。它占地面积约72万平方米，建筑面积约15万平方米，有大小宫殿七十多座。";

            this.setCanvas(ggText, [
              intersects[0].point.x,
              intersects[0].point.y,
            ]);
          } else {
            canvasPlane.visible = false;
          }
        }
      };
      // document.addEventListener('click', onMouseMove, false)
      document
        .getElementsByTagName("canvas")[0]
        .addEventListener("click", onMouseMove, false);
    },
    setRaycaster_hover() {
      raycaster = new THREE.Raycaster();
      mouse = new THREE.Vector2();
      const onMouseMove = (event) => {
        // 将鼠标位置归一化为设备坐标。x 和 y 方向的取值范围是 (-1 to +1)
        mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
        mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
      };
      window.addEventListener("mousemove", onMouseMove, false);
    },
    //获取故宫的模型
    GetGLTFLoader(x, y, z) {
      const loader = new GLTFLoader();
      //加载模型文件，返回gltf对象
      loader.load("/data/gugong.glb", function (gltf) {
        gltf.scene.scale.set(0.03, 0.03, 0.03);
        gltf.scene.position.set(x, y, z);
        gltf.scene.rotation.x = 1.5;
        gltf.scene.rotation.y = 1.5;
        map.add(gltf.scene);
      });
    },
    getSpiritAndGltf() {
      let spriteMaterial;
      let bingmayong = new THREE.TextureLoader().load("/data/bingmayong.png");
      special.forEach((item, index) => {
        switch (item.name) {
          case "北京市":
            this.GetGLTFLoader(item.arr[0][0], item.arr[0][1], 8);
            break;
          case "陕西省":
            spriteMaterial = new THREE.SpriteMaterial({
              map: bingmayong,
              transparent: true,
            });
            let sprite = new THREE.Sprite(spriteMaterial);
            sprite.position.set(item.arr[0][0], item.arr[0][1], 8);
            sprite.scale.set(8, 8, 8);
            sprite.name = "bingmayong";
            map.add(sprite);
            break;
        }
      });
    },
    feng() {
      // 去掉锯齿---1
      // 通过ShaderPass构造函数把FXAAShader着色器和uniforms构成的对象作为参数，创建一个锯齿通道FXAAShaderPass,然后把锯齿通道插入到composer中。
      const effectFXAA = new ShaderPass(FXAAShader);
      effectFXAA.uniforms["resolution"].value.set(
        0.6 / window.innerWidth,
        0.6 / window.innerHeight
      ); // 渲染区域Canvas画布宽高度  不一定是全屏，也可以是区域值
      effectFXAA.renderToScreen = true;
      // 去掉锯齿---1
      const renderScene = new RenderPass(scene, camera); // RenderPass这个通道会在当前场景（scene）和摄像机（camera）的基础上渲染出一个新场景，新建：
      // 添加光晕效果---2
      bloomPass = new UnrealBloomPass( // UnrealBloomPass通道可实现一个泛光效果。
        new THREE.Vector2(window.innerWidth, window.innerHeight),
        1.5,
        0.4,
        0.85
      );
      bloomPass.threshold = params.bloomThreshold;
      bloomPass.strength = params.bloomStrength;
      bloomPass.radius = params.bloomRadius;
      // 添加光晕效果---2
      // 着色器通道容器--放进容器里
      bloomComposer = new EffectComposer(renderer); // EffectComposer可以理解为着色器通道容器，着色器通道按照先后顺序添加进来并执行
      bloomComposer.renderToScreen = false;
      bloomComposer.addPass(renderScene);
      bloomComposer.addPass(bloomPass); // 添加光晕效果
      bloomComposer.addPass(effectFXAA); // 去掉锯齿
      // 着色器通道容器--放进容器里
      const finalPass = new ShaderPass(
        new THREE.ShaderMaterial({
          uniforms: {
            baseTexture: { value: null },
            bloomTexture: { value: bloomComposer.renderTarget2.texture },
          },
          vertexShader: document.getElementById("vertexshader").textContent,
          fragmentShader: document.getElementById("fragmentshader").textContent,
          defines: {},
        }),
        "baseTexture"
      );
      finalPass.needsSwap = true;
      finalComposer = new EffectComposer(renderer);
      finalComposer.addPass(renderScene);
      finalComposer.addPass(finalPass);
      finalComposer.addPass(effectFXAA);
    },
    //渲染器
    getRenderer() {
      let that = this;
      renderer = new THREE.WebGLRenderer({
        antialias: true,
        alpha: true,
        // preserveDrawingBuffer: true
      });
      //开启阴影
      //   renderer.shadowMap.enabled = true;
      // renderer.shadowMap.type = THREE.PCFSoftShadowMap;

      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(window.innerWidth, window.innerHeight);

      document.getElementById("role").appendChild(renderer.domElement);
      //    renderer.antialias = true; // 开启抗锯齿
      let canvas = renderer.context.canvas;
      canvas.addEventListener(
        "webglcontextlost",
        function (event) {
          event.preventDefault();
          setTimeout(function () {
            alert("您的运行内存不足，建议您刷新浏览器或者重启电脑");
            // cancelAnimationFrame(that.AnimationFrameId)
            // that.renderer.forceContextRestore();
          }, 1);
        },
        false
      );
    },
    //摄像机
    getCamera() {
      // 第二参数就是 长度和宽度比 默认采用浏览器  返回以像素为单位的窗口的内部宽度和高度
      camera = new THREE.PerspectiveCamera(
        70,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
      );
      camera.position.z = 100;
      camera.lookAt(scene.position);
    },
    //辅助线
    addHelper() {
      const helper = new THREE.CameraHelper(camera);
      scene.add(helper);
    },
    //控制器
    setController() {
      controller = new OrbitControls(camera, renderer.domElement);
    },

    setLight() {
      ambientLight = new THREE.AmbientLight(0xffffff); // 环境光
      //平行光 用来照亮地图
      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
      directionalLight.position.set(20, 20, 20);
      scene.add(directionalLight);
      //平行光灯光辅助器，可以看见灯光的位置
      const lightHelper = new THREE.DirectionalLightHelper(directionalLight);
      scene.add(lightHelper);
      // scene.add(ambientLight)
    },
    drawText(text, ctx, position) {
      ctx.clearRect(0, 0, canvas.width, canvas.height); // 清除整个canvas
      // 设置背景颜色为蓝色
      ctx.fillStyle = "#36FFFF";
      // 绘制一个矩形，该矩形的背景颜色为前面设置的蓝色
      ctx.fillRect(0, 0, 200, 130);
      let promise = new Promise(function (resolve, reject) {
        ctx.font = "15px Arial";
        ctx.fillStyle = "#000000";
        var lineWidth = 200; // 每行文本的最大宽度，可根据实际需求进行调整
        var lineHeight = 20; // 每行文本的高度差值，可根据实际需求进行调整
        var x = 10; // 文本起始位置的x坐标，可根据实际需求进行调整
        var y = 20; // 文本起始位置的y坐标，可根据实际需求进行调整
        var lineCount = 0; // 当前行数，用于控制换行次数，可根据实际需求进行调整
        for (var i = 0; i < text.length; i++) {
          var charWidth = ctx.measureText(text[i]).width; // 当前字符的宽度，用于判断是否需要换行，并计算该字符占用空间的大小

          if (x + charWidth > lineWidth) {
            // 如果当前字符无法容纳在当前行中，需要换行并增加一行的高度差值（即多增加一行）
            x = 10; // 重置x坐标，开始新的一行，即从下一行的起始位置开始重新绘制字符（相当于清空当前行的绘制）
            y += lineHeight; // 增加一行的高度差值
            lineCount++; // 增加一行数
          }
          ctx.fillText(text[i], x, y); // 绘制当前字符
          x += charWidth; // 更新x坐标，准备绘制下一个字符
        }
        resolve("成功");
      });
      promise.then((res) => {
        //从新创建一个canvas贴图
        let canvasTexture = new THREE.CanvasTexture(canvas);
        let newMaterial = new THREE.MeshBasicMaterial({ map: canvasTexture });
        //替换掉贴图
        canvasPlane.material = newMaterial;
        //显示
        canvasPlane.visible = true;
        //根据传过来的位置，给几何体从新定位
        canvasPlane.position.set(...position, 14);
      });
    },
    //创建canvans贴图
    getCanvas() {
      canvas = document.createElement("canvas");
      canvas.width = 195;
      canvas.height = 130;

      let canvasTexture = new THREE.CanvasTexture(canvas);
      let material = new THREE.MeshBasicMaterial({ map: canvasTexture });
      let planeGeometry = new THREE.PlaneGeometry(20, 21);
      canvasPlane = new THREE.Mesh(planeGeometry, material);
      canvasPlane.position.set(0, 0, -20);
      canvasPlane.rotation.x = -5.5;
      canvasPlane.visible = false;
      map.add(canvasPlane);
    },
    setCanvas(text, position) {
      let ctx = canvas.getContext("2d");
      this.drawText(text, ctx, position);
    },
    // 获取地图数据
    getMapData() {
      let loader = new THREE.FileLoader();
      map = new THREE.Object3D();
      loader.load("/data/map.json", (data) => {
        const jsondata = JSON.parse(data);
        this.generateGeometry(jsondata);
      });
    },
    // 加载地图
    generateGeometry(jsondata) {
      let linArr = [];
      let material_1, material_2, extrudeSettings, extrudeSettings_2;
      //新建一个Object3D对象，这个主要用来装地图几何快的  集合

      const textureLoader = new THREE.TextureLoader();
      //地图秃秃的不好看，我们给地图加个皮肤，使用three.js自带的文件加载器，加载一张图片
      //加载完成后  创建贴图
      textureLoader.load("/data/mapBack.png", (texture) => {
        //关闭 matrixAutoUpdate 属性，方便我们后面对贴图进行  矩阵转换
        texture.matrixAutoUpdate = false;

        //构建地图的 正面贴图 ，这里使用基础材质，关于材质，大家可以去详细看看api
        material = new THREE.MeshBasicMaterial({
          transparent: true,
          opacity: 1,
          map: texture,
        });

        //构建地图的 侧面贴图 ，这里使用基础材质
        material_1 = new THREE.MeshBasicMaterial({
          color: "#558BAB",
          transparent: true,
          opacity: 0.45,
        });
        material_2 = new THREE.MeshBasicMaterial({
          color: "#ffffff",
          transparent: true,
          opacity: 0,
        });
        extrudeSettings = {
          depth: 2,
          bevelEnabled: false,
        };
        extrudeSettings_2 = {
          depth: 1,
          bevelEnabled: false,
        };

        //d3是另一种3d可视化工具 ，这里我们需要引用他，用它的中心点去矫正我们的地图坐标
        //保证地图出现在我们想要的位置
        // 墨卡托投影转换
        const projection = d3
          .geoMercator()
          //this.centerX,this.centerY  代表我们屏幕的正中间出现的 地图中心点位置
          // centerX:108.5525 ,
          // centerY:34.3227,
          //随便百度的，你也可以根据你的需求来更改中心点
          .center([this.centerX, this.centerY])

          .translate([0, 0]);

        jsondata.features.forEach((elem) => {
          if (
            elem.properties.name == "陕西省" ||
            elem.properties.name == "北京市"
          ) {
            let obj = {
              name: elem.properties.name,
              arr: [projection(elem.properties.center)],
            };
            special.push(obj);
          }
          // this.pointArr=[]

          // 定一个省份3D对象
          province = new THREE.Object3D();
          // 每个的 坐标 数组
          const coordinates = elem.geometry.coordinates;
          // 循环坐标数组
          coordinates.forEach((multiPolygon) => {
            multiPolygon.forEach((polygon, index) => {
              // linArr.length=0
              //构建几何图形
              linArr.length = 0;
              const shape = new THREE.Shape();
              for (let i = 0; i < polygon.length; i++) {
                const [x, y] = projection(polygon[i]);
                //指定我们的起点
                if (i === 0) {
                  shape.moveTo(x, -y);
                }
                linArr.push(x);
                linArr.push(-y);
                linArr.push(5.7);
                //后续就开始从起点画线
                //如果你使用过canvans画线，那你肯定秒懂，他们是一个道理
                shape.lineTo(x, -y);
              }
              this.getLin(linArr);
              //这里我们把先前构造的几何图形 通过ExtrudeGeometry 拉伸成几何体
              const geometry = new THREE.ExtrudeGeometry(
                shape,
                extrudeSettings
              );
              const geometry_2 = new THREE.ExtrudeGeometry(
                shape,
                extrudeSettings_2
              );
              //把几何体和  我们两个贴图 合成一个 网格对象 mesh
              const mesh = new THREE.Mesh(geometry, [material, material_1]);
              const mesh_2 = new THREE.Mesh(geometry_2, [
                material_1,
                material_2,
              ]);
              const mesh_3 = mesh_2.clone();
              // mesh.castShadow=true
              //把网格对象加入省份的3d对象
              mesh.position.z = 4;
              mesh_2.position.z = 1;
              mesh_3.position.z = -2;
              province.add(mesh);
              province.add(mesh_2);
              province.add(mesh_3);
            });
          });

          map.add(province);
          this.getCanvas();
        });
        updateUV();
        guisd.add(gui, "offsetX", 0.0, 1.0).onChange(updateUV);
        guisd.add(gui, "offsetY", 0.0, 1.0).onChange(updateUV);
        guisd.add(gui, "repeatX", -10.0, 10.0).onChange(updateUV);
        guisd.add(gui, "repeatY", -10.0, 10.0).onChange(updateUV);
        guisd.add(gui, "rotation", -10.0, 10.0).onChange(updateUV);
        guisd.add(gui, "centerX", 0.0, 1.0).onChange(updateUV);
        guisd.add(gui, "centerY", 0.0, 1.0).onChange(updateUV);
        guisd.add(gui, "RepeatWrapping").onChange(function (e) {
          if (e) {
            material.map.wrapS = material.map.wrapT = THREE.RepeatWrapping; //设置为可循环
          } else {
            material.map.wrapS = material.map.wrapT = THREE.ClampToEdgeWrapping; //设置会默认的最后一像素伸展
          }

          material.map.needsUpdate = true;
        });

        scene.add(map);
        map.rotation.x = -0.82;
        this.getSpiritAndGltf();
      });
    },
    getLin(arr) {
      let groups = new THREE.Group();

      if (arr.length != 0) {
        // 初始化MeshLine
        const line = new MeshLine();
        // 传入顶点坐标数据
        line.setPoints(arr);
        // 生成线材质
        let material = new MeshLineMaterial({
          useMap: 0,
          color: "#0fb9ee",
          opacity: 1,
          resolution: new THREE.Vector2(window.innerWidth, window.innerHeight),
          sizeAttenuation: 2,
          lineWidth: 0.6,
          transparent: true,
          wireframe: false,
        });
        const mesh = new THREE.Mesh(line.geometry, material);

        mesh.layers.enable(1);
        groups.add(mesh);
        map.add(groups);
      }
    },

    // 启动
    animate() {
      this.AnimationFrameId = requestAnimationFrame(this.animate.bind(this));
      this.render();
    },
    render() {
      scene.traverse((obj) => {
        if (obj instanceof THREE.Scene) {
          this.materials = obj.background;
          obj.background = null;
        }
        if (bloomLayer.test(obj.layers) === false) {
          materials[obj.uuid] = obj.material;
          obj.material = darkMaterial;
        }
      });

      bloomComposer.render();
      scene.traverse((obj) => {
        if (materials[obj.uuid]) {
          obj.material = materials[obj.uuid];
          delete materials[obj.uuid];
        }
        if (obj instanceof THREE.Scene) {
          obj.background = this.materials;
          delete this.materials;
        }
      });
      finalComposer.render();
    },
  },
};
</script>

<style scoped>
.content_2d01 {
  width: 50px;
  height: 50px;
  background-color: aliceblue;
}
</style>
