<!--
 * @Author: caixin caixin185@163.com
 * @Date: 2023-04-08 10:54:16
 * @LastEditors: caixin
 * @LastEditTime: 2023-04-08 18:27:26
 * @Description: file content
-->
<template>
    <div ref="threeRef"></div>
</template>
<script lang="ts">
import { defineComponent,onMounted,ref } from 'vue'
import {Scene,PerspectiveCamera,WebGLRenderer,Color,AxesHelper,PlaneGeometry,MeshBasicMaterial,Mesh,BoxGeometry, SphereGeometry} from 'three'


export default defineComponent({
    setup() {
        const threeRef = ref()
        function init() {
            const scene = new Scene()
            const axes = new AxesHelper(20)
            scene.add(axes)

            const renderer = new WebGLRenderer({ antialias: true })
            renderer.setClearColor(new Color(0xEEEEEE))
            renderer.setSize(window.innerWidth, window.innerHeight+60)

            const planeGeometry = new PlaneGeometry(60, 20)
          const planeBasicMaterial = new MeshBasicMaterial({ color: 0xcccccc })
          const plane = new Mesh(planeGeometry, planeBasicMaterial)
          plane.rotation.x = -0.5 * Math.PI
          plane.rotation.x = 30
          plane.position.y = 0
          plane.position.z = 10
          scene.add(plane)

          const cubeGeometry = new BoxGeometry(4, 4, 14) //盒子
          const cubeMaterial = new MeshBasicMaterial({ color: 0xff0000, wireframe: true })
          const cube = new Mesh(cubeGeometry, cubeMaterial)
          cube.position.x = 10
          cube.position.y = 3
          cube.position.z = 10
          scene.add(cube)

          const sphereGeometry = new SphereGeometry(4, 20, 20) //球形
          const sphereMaterial = new MeshBasicMaterial({ color: 0x77777ff, wireframe: true })
          const sphere = new Mesh(sphereGeometry, sphereMaterial)
          sphere.position.x = 40
          sphere.position.y = 4
          sphere.position.z = 10
          scene.add(sphere)

          const camera = new PerspectiveCamera(80, window.innerWidth / window.innerHeight)
          camera.position.x = -30
          camera.position.y = 40
          camera.position.z = 30
          camera.lookAt(scene.position)
          threeRef.value.appendChild(renderer.domElement)
          renderer.render(scene,camera)
        }
    onMounted(() => {
          init()
    })
    return {
          threeRef
        }
    },
})
</script>
