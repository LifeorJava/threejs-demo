<template>
  <div style="width: 100%; height: 100%;">
    <div ref="container" class="panel-container"><canvas ref="canvas"></canvas></div>
  </div>

</template>

<script>
import * as THREE from 'three';

export default {
  data() {
    return {
      scene: null,
      camera: null,
      renderer: null,
      panels: [],
      selectedPanel: null,
      rotationSpeed: 0.0009, //轮播速度
      isRotating: true,
      numPanels: 20,// 面板数量
      radius: 12, // 圆的半径
      selectedPanelIndex: null
    };
  },
  mounted() {
    this.initScene();
    this.addPanels();
    this.animate();
  },
  beforeDestroy() {
    if (this.renderer) {
      this.renderer.dispose();
    }
    if (this.scene) {
      while (this.scene.children.length > 0) {
        const child = this.scene.children[0];
        this.scene.remove(child);
        if (child.geometry) child.geometry.dispose();
        if (child.material) {
          if (Array.isArray(child.material)) {
            child.material.forEach(material => material.dispose());
          } else {
            child.material.dispose();
          }
        }
      }
    }
  },
  methods: {
    initScene() {
      // 创建一个新的 Three.js 场景
      this.scene = new THREE.Scene();

      // 设置相机
      this.camera = new THREE.PerspectiveCamera(
        75, // 视野角度
        this.$refs.container.clientWidth / this.$refs.container.clientHeight, // 长宽比
        1, // 近裁剪平面
        50 // 远裁剪平面
      );
      this.camera.position.z = 35; // 将相机向后移动

      // 设置 WebGL 渲染器
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(this.$refs.container.clientWidth, this.$refs.container.clientHeight);
      this.$refs.canvas.appendChild(this.renderer.domElement);

      // 添加环境光
      const ambientLight = new THREE.AmbientLight(0x404040); // 柔和的白色光线
      this.scene.add(ambientLight);

      // 添加方向光
      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
      directionalLight.position.set(1, 1, 1).normalize();
      this.scene.add(directionalLight);

      // 添加窗口大小调整事件监听器
      window.addEventListener('resize', this.onWindowResize, false);
    },

    onWindowResize() {
      // 在窗口大小调整时更新相机的长宽比和渲染器的大小
      this.camera.aspect = this.$refs.container.clientWidth / this.$refs.container.clientHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(this.$refs.container.clientWidth, this.$refs.container.clientHeight);
    },

    addPanels() {
      const angleStep = (Math.PI * 2) / this.numPanels; // 每个面板之间的角度
      for (let i = 0; i < this.numPanels; i++) {
        const angle = i * angleStep;
        const x = Math.cos(angle) * this.radius;
        const y = Math.sin(angle) * this.radius;

        // 创建一个 div 元素作为面板
        const panelDiv = document.createElement('div');
        panelDiv.id=i;
        panelDiv.className = 'panel';
        panelDiv.innerHTML = `
          <p >姓名 ${i + 1}</p>
          <p >性别 ${i % 2 ==0?'女':'男'}</p>
          <p >特长：12312阿萨大时代额2额12额 2额 asasdasd12e12dascvbtewqdsasvbsasssadbnytrewrwdafmerweawAzbnhjyhdstsd2额</p>
          <button class='mini'  @click="showDetails(${i})">自定义按钮</button>
        `;
        const button = panelDiv.querySelector('button');
        button.addEventListener('click', () => this.showDetails(i));
        //给每个DIV增加鼠标事件
        panelDiv.addEventListener('mouseenter', () => this.stopRotation(i));
        panelDiv.addEventListener('mouseleave', () => this.startRotation());
        this.$refs.container.appendChild(panelDiv);

        // 获取实际尺寸
        const rect = panelDiv.getBoundingClientRect();
        const width = rect.width;
        const height = rect.height;

        // 创建一个网格对象在 3D 场景中表示面板
        const element = panelDiv;
        const material = new THREE.MeshBasicMaterial({ transparent: true, opacity: 0 });
        const geometry = new THREE.PlaneGeometry(width, height);
        const mesh = new THREE.Mesh(geometry, material);
        mesh.userData.element = element;
        mesh.position.set(x, 1, y); // 所有卡片都在 Y=1 的平面上
        this.panels.push(mesh);
        this.scene.add(mesh);
      }
      // 确保第一个面板在屏幕中心
      this.alignFirstPanelToCenter();
    },

    alignFirstPanelToCenter() {
      if (this.panels.length === 0) {
        return;
      }
      const firstPanel = this.panels[0];
      const targetPosition = new THREE.Vector3(0, 0, 0);
      // 计算需要旋转的角度以使第一个面板对齐到屏幕中心
      const currentAngle = Math.atan2(firstPanel.position.z, firstPanel.position.x);
      const desiredAngle = 0; // 第一个面板应该在正前方（0度）
      const angleDifference = desiredAngle - currentAngle;
      const rotationAxis = new THREE.Vector3(0, 1, 0);
      // 旋转所有面板
      this.panels.forEach((mesh) => {
        mesh.applyMatrix4(new THREE.Matrix4().makeRotationFromQuaternion(
          new THREE.Quaternion().setFromAxisAngle(rotationAxis, angleDifference)
        ));
      });
      // 更新相机位置以保持视角一致
      this.camera.position.setZ(15);
      this.camera.lookAt(targetPosition);
    },

    showDetails(index) {
      alert(index);
    },

      // 关闭详细信息
    closeDetails() {
      this.selectedPanelIndex = null;
      // 启动旋转
      this.startRotation();
      // 反转回原来的状态
      this.unflipAllCards();
    },

    unflipAllCards() {
      this.panels.forEach((mesh) => {

        const panelDiv = mesh.userData.element;
        panelDiv.classList.remove('big');
      });
    },

    animate() {
      requestAnimationFrame(this.animate.bind(this),2); // 请求下一帧
      this.render(); // 渲染场景
    },

    render() {
      // 更新每个面板的位置
      this.panels.forEach((mesh) => {
        if (mesh.userData.element) {
          const pos = mesh.getWorldPosition(new THREE.Vector3());
          const vector = pos.project(this.camera);
          const ele = mesh.userData.element;
          if(this.$refs.container){
            ele.style.left = `${(vector.x * this.$refs.container.clientWidth) / 2 + this.$refs.container.clientWidth / 2}px`;
            ele.style.top = `${-(vector.y * this.$refs.container.clientHeight) / 2 + this.$refs.container.clientHeight / 2}px`;
          }
       
          // 根据面板在视口中的可见性显示或隐藏面板
          if (vector.z > -1 && vector.z < 1) {
            ele.style.display = 'block';
          } else {
            ele.style.display = 'none';
          }

          // 符合近大远小规律
          const scale = 1 - Math.abs(vector.z);
          ele.style.transform = `scale(${scale})`;
        }
      });
      
      // 如果正在旋转，则使面板围绕 Y 轴旋转
      if (this.isRotating) {
        this.scene.rotation.y += this.rotationSpeed;
      }

      // 使用更新后的相机位置渲染场景
      this.renderer.render(this.scene, this.camera);
    },

    stopRotation(index) {
      this.isRotating = false;
      let div = document.getElementById(index);      
      div.classList.add(`big`)
    },

    startRotation() {
      this.panels.forEach((mesh) => {
        const panelDiv = mesh.userData.element;
        panelDiv.classList.remove('big');
      });
      this.isRotating = true;
    }

  }
};
</script>

<style scoped>
/* 组件特定的样式 */
</style>