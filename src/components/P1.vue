<template>
  <div>
    <div id="container"></div>
  </div>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
// @ts-ignore
import * as THREE from 'three';

@Component
export default class ClassComponent extends Vue {
  scene: any;
  camera: any;
  rendererScene: any;
  rollOverMaterial: any;
  rollOverMesh: any;
  raycaster: any;
  pointer: any;
  cubeGeo: any;
  cubeMaterial: any;
  plane: any;

  objects: [] = [];

  isShiftDown: boolean = false;

  init() {
    const container = document.getElementById('container');

    // scene
    this.scene = new THREE.Scene();
    this.scene.background = new THREE.Color(0xf0f0f0);

    // camera
    this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 10000);
    this.camera.position.set(1000, 1000, 1000);
    this.camera.lookAt(0, 0, 0);

    // grid
    const gridHelper = new THREE.GridHelper(1000, 20);
    this.scene.add(gridHelper);

    // roll-over helpers
    const rollOverGeo = new THREE.BoxGeometry(50, 50, 50);
    this.rollOverMaterial = new THREE.MeshBasicMaterial({
      color: 0xff0000,
      opacity: 0.5,
      transparent: true
    })
    this.rollOverMesh = new THREE.Mesh(rollOverGeo, this.rollOverMaterial);
    this.scene.add(this.rollOverMesh);

    // cursor
    this.raycaster = new THREE.Raycaster();
    this.pointer = new THREE.Vector2();

    // cube
    this.cubeGeo = new THREE.BoxGeometry(50, 50, 50);
    this.cubeMaterial = new THREE.MeshBasicMaterial({
      color: 0xdeb74c
    })

    // geometry
    const geometry = new THREE.PlaneGeometry(1000, 1000);
    geometry.rotateX(-Math.PI / 2);

    // plane
    this.plane = new THREE.Mesh(geometry, new THREE.MeshBasicMaterial({
      visible: false
    }));

    // @ts-ignore
    this.objects.push(this.plane)

    // lights
    const ambientLight = new THREE.AmbientLight(0x606060);
    this.scene.add(ambientLight);

    const directionalLight = new THREE.DirectionalLight(0xffffff);
    directionalLight.position.set(100, 10, 10).normalize();
    this.scene.add(directionalLight);

    // renderer
    this.rendererScene = new THREE.WebGLRenderer({
      antialias: true
    });
    this.rendererScene.setPixelRatio(window.devicePixelRatio);
    this.rendererScene.setSize(window.innerWidth, window.innerHeight);

    container?.appendChild(this.rendererScene.domElement);

    document.addEventListener('pointermove', this.onPointerMove);
    document.addEventListener('pointerdown', this.onPointerDown);
    document.addEventListener('keydown', this.onDocumentKeyDown);
    document.addEventListener('keyup', this.onDocumentKeyUp);

    window.addEventListener('resize', this.onWindowResize);
  }

  onWindowResize() {
    this.camera.aspect = window.innerWidth / window.innerHeight;
    // вызывается после любого изменения параметров
    this.camera.updateProjectionMatrix();
    this.rendererScene.setSize(window.innerWidth, window.innerHeight);

    this.renderScene();
  }

  onPointerMove(event: any) {
    this.pointer.set(
      (event.clientX / window.innerWidth) * 2 - 1,
      - (event.clientY / window.innerHeight) * 2 + 1
    );
    this.raycaster.setFromCamera(this.pointer, this.camera);

    const intersects = this.raycaster.intersectObjects(this.objects, false);

    if (intersects.length > 0) {
      const intersect = intersects[0];
      this.rollOverMesh.position.copy(intersect.point).add(intersect.face.normal);
      this.rollOverMesh.position.divideScalar(50).floor().multiplyScalar(50).addScalar(25);

      this.renderScene();
    }
  }

  onPointerDown(event: any) {
    this.pointer.set(
      (event.clientX / window.innerWidth) * 2 - 1,
      - (event.clientY / window.innerHeight) * 2 + 1
    );
    this.raycaster.setFromCamera(this.pointer, this.camera);

    const intersects = this.raycaster.intersectObjects(this.objects, false);

    if (intersects.length > 0) {
      const intersect = intersects[0];

       // delete
      if (this.isShiftDown) {
        if (intersect.object !== this.plane) {
          this.scene.remove(intersect.object);
          // @ts-ignore
          this.objects.splice(this.objects.indexOf(intersect.object), 1);
        }
      } else {
        // create
        const voxel = new THREE.Mesh(this.cubeGeo, this.cubeMaterial);
        voxel.position.copy(intersect.point).add(intersect.face.normal);
        voxel.position.divideScalar(50).floor().multiplyScalar(50).addScalar(25);

        this.scene.add(voxel);
        // @ts-ignore
        this.objects.push(voxel);
      }

      this.renderScene();
    }
  }

  onDocumentKeyDown(event: any) {
    switch (event.keyCode) {
      case 16: this.isShiftDown = true;
      break;
    }
  }

  onDocumentKeyUp(event: any) {
    switch (event.keyCode) {
      case 16: this.isShiftDown = false;
      break;
    }
  }

  renderScene() {
    this.rendererScene.render(this.scene, this.camera);
  }

  mounted() {
    this.init();
    this.renderScene();
  }
}
</script>
