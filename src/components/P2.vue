<template>
  <div>
    <div id="container"></div>
  </div>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

@Component
export default class ClassComponent extends Vue {
  scene: any;
  camera: any;
  mesh: any;
  rendererScene: any;
  controls: any;
  mouse = new THREE.Vector2(1, 1);
  raycaster = new THREE.Raycaster();
  white = new THREE.Color().setHex(0xffffff);
  color = new THREE.Color();

  init() {
    const container = document.getElementById('container');
    const amount = parseInt(window.location.search.slice(1)) || 10;
    const count = Math.pow(amount, 3);

    // color
    const color = new THREE.Color();

    // scene
    this.scene = new THREE.Scene();

    // camera
    this.camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 100);
    this.camera.position.set(amount, amount, amount);
    this.camera.lookAt(0, 0, 0);

    // light
    const light = new THREE.HemisphereLight(0xffffff, 0x888888);
    light.position.set(0, 1, 0);
    this.scene.add(light);

    // geometry
    const geometry = new THREE.IcosahedronGeometry(0.5, 3);
    const material = new THREE.MeshPhongMaterial();
    // визуализация большого количества объектов с одинаковой геометрией
    this.mesh = new THREE.InstancedMesh(geometry, material, count);

    let i = 0;
    const offset = (amount - 1) / 2;
    const matrix = new THREE.Matrix4();

    for (let x = 0; x < amount; x++) {
        for (let y = 0; y < amount; y++) {
            for (let z = 0; z < amount; z++) {
                matrix.setPosition(offset - x, offset - y, offset - z);

                this.mesh.setMatrixAt(i, matrix);
                this.mesh.setColorAt(i, color);

                i++
            }
        }
    }

    this.scene.add(this.mesh);

    // renderer
    this.rendererScene = new THREE.WebGLRenderer({antialias: true});
    this.rendererScene.setPixelRatio(window.devicePixelRatio);
    this.rendererScene.setSize(window.innerWidth, window.innerHeight);

    // orbitControls
    this.controls = new OrbitControls(this.camera, this.rendererScene.domElement);
    this.controls.enableDamping = true;

    container?.appendChild(this.rendererScene.domElement);

    window.addEventListener('resize', this.onWindowResize);
    document.addEventListener('mousemove', this.onMouseMove)
  }

  onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();

      this.rendererScene.setSize(window.innerWidth, window.innerHeight);
  }

  onMouseMove(event: any) {
      event.preventDefault();

      this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      this.mouse.y = - (event.clientY / window.innerHeight) * 2 + 1;
  }

  animate() {
      requestAnimationFrame(this.animate);
      this.controls.update();
      this.raycaster.setFromCamera(this.mouse, this.camera);

      const intersection = this.raycaster.intersectObject(this.mesh);

      if (intersection.length > 0) {
          const instanceId = intersection[0].instanceId;
          this.mesh.getColorAt(instanceId, this.color);

          if (this.color.equals(this.white)) {
              this.mesh.setColorAt(instanceId, this.color.setHex(Math.random() * 0xffffff));
              this.mesh.instanceColor.needsUpdate = true;
          }
      }
      this.renderScene();
  }

  renderScene() {
    this.rendererScene.render(this.scene, this.camera);
  }

  mounted() {
    this.init();
    this.animate();
    // this.renderScene();
  }
}
</script>
