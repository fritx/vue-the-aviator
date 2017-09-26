<template>
  <renderer :obj="renderer" :size="renderSize">
    <scene :obj="scene">
      <camera :obj="camera"
        :position="ui.camera.position"></camera>
      <object3d :obj="sea"
        :position="ui.sea.position"
        :rotation="ui.sea.rotation"></object3d>
      <object3d :obj="sky"
        :position="ui.sky.position"
        :rotation="ui.sky.rotation"></object3d>
      <animation :fn="loop"></animation>
    </scene>
  </renderer>
</template>

<script>
/* eslint-disable semi, comma-spacing, key-spacing */
/* eslint-disable comma-dangle, space-infix-ops, no-floating-decimal */
/* eslint-disable space-before-blocks, space-in-parens, quotes */
/* eslint-disable keyword-spacing */
/* eslint-disable space-before-function-paren */
import * as THREE from 'three'
import AirPlane from './AirPlane'
import { Colors } from './common'

function normalize(v, vmin, vmax, tmin, tmax){
  var nv = Math.max(Math.min(v,vmax), vmin);
  var dv = vmax-vmin;
  var pc = (nv-vmin)/dv;
  var dt = tmax-tmin;
  var tv = tmin + (pc*dt);
  return tv;
}

export default {
  name: 'Game',

  data () {
    return {
      ui: {
        mouse: { x: 0, y: 0 },
        airplane: {
          position: { y: 100 }
        },
        sea: {
          position: { y: -600 },
          rotation: { z: 0 }
        },
        sky: {
          position: { y: -600 },
          rotation: { z: 0 }
        },
        camera: {
          position: { x: 0, y: 100, z: 200 }
        }
      },
      WIDTH: window.innerWidth,
      HEIGHT: window.innerHeight
    }
  },

  computed: {
    renderSize () {
      return {
        w: this.WIDTH,
        h: this.HEIGHT
      }
    }
  },

  created () {
    this.renderer = this.createRenderer()
    this.scene = this.createScene()
    this.camera = this.createCamera()

    this.sea = this.createSea()
    this.sky = this.createSky()
    this.airplane = this.createAirPlane()
  },

  methods: {
    loop () {
      this.ui.sea.rotation.z += .005
      this.ui.sky.rotation.z += .01
      // update plane
      var targetY = normalize(this.ui.mouse.y,-.75,.75,25, 175);
      var targetX = normalize(this.ui.mouse.x,-.75,.75,-100, 100);
      this.ui.airplane.position.y = targetY;
      this.ui.airplane.position.x = targetX;
      if (this.airplane && this.airplane.propeller) this.airplane.propeller.rotation.x += 0.3;
    },

    createAirPlane () {
      let airplane = new AirPlane();
      airplane.mesh.scale.set(.25,.25,.25);
      return airplane
    },

    createSky () {
      let mesh = new THREE.Object3D()
      let nClouds = 20
      let clouds = []
      var stepAngle = Math.PI*2 / nClouds
      for (var i=0; i<nClouds; i++) {
        var c = this.createCloud()
        clouds.push(c)
        var a = stepAngle*i
        var h = 750 + Math.random()*200
        c.position.y = Math.sin(a)*h
        c.position.x = Math.cos(a)*h
        c.position.z = -400-Math.random()*400
        c.rotation.z = a + Math.PI/2
        var s = 1+Math.random()*2
        c.scale.set(s,s,s)
        mesh.add(c)
      }
      return mesh
    },
    createCloud () {
      let mesh = new THREE.Object3D()
      mesh.name = 'cloud'
      var geom = new THREE.CubeGeometry(20,20,20)
      var mat = new THREE.MeshPhongMaterial({
        color: Colors.white,
      })

      var nBlocs = 3+Math.floor(Math.random()*3)
      for (var i=0; i<nBlocs; i++ ) {
        var m = new THREE.Mesh(geom.clone(), mat)
        m.position.x = i*15
        m.position.y = Math.random()*10
        m.position.z = Math.random()*10
        m.rotation.z = Math.random()*Math.PI*2
        m.rotation.y = Math.random()*Math.PI*2
        var s = .1 + Math.random()*.9
        m.scale.set(s,s,s)
        m.castShadow = true
        m.receiveShadow = true
        mesh.add(m)
      }
      return mesh
    },

    createSea () {
      var geom = new THREE.CylinderGeometry(600,600,800,40,10)
      geom.applyMatrix(new THREE.Matrix4().makeRotationX(-Math.PI/2))
      var mat = new THREE.MeshPhongMaterial({
        color:Colors.blue,
        transparent:true,
        opacity:.6,
        shading:THREE.FlatShading,
      })
      var mesh = new THREE.Mesh(geom, mat)
      mesh.receiveShadow = true
      return mesh
    },

    createCamera () {
      let aspectRatio = this.WIDTH / this.HEIGHT
      let fieldOfView = 60
      let nearPlane = 1
      let farPlane = 10000
      let camera = new THREE.PerspectiveCamera(
        fieldOfView,
        aspectRatio,
        nearPlane,
        farPlane
        )
      return camera
    },
    createScene () {
      let scene = new THREE.Scene()
      scene.fog = new THREE.Fog(0xf7d9aa, 100,950)
      return scene
    },
    createRenderer () {
      let renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true })
      renderer.shadowMap.enabled = true
      return renderer
    }
  }
}
</script>

<style>
body { margin: 0; overflow: hidden; }
.dg.main { user-select: none; }
</style>

<style scoped>
.panel { position: absolute; right: 0 }
</style>