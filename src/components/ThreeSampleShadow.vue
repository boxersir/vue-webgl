<template>
     <div ref="statsDivRef" class="stats"></div>
    <div ref="threeRef"></div>
</template>
<script lang="ts"> 
import { defineComponent, onMounted,ref} from 'vue'
import {Scene,PerspectiveCamera,WebGLRenderer,Color,AxesHelper,MeshLambertMaterial,PlaneGeometry,Mesh,BoxGeometry, SphereGeometry, SpotLight,AmbientLight} from 'three'
import Stats from 'stats.js'
import * as dat from 'dat.gui';

export default defineComponent({
    setup() {

        const  threeRef=ref<HTMLDivElement>()
        const  statsDivRef=ref<HTMLDivElement>()
        const statsRef=ref<Stats>()
        function  initStats(){
            statsRef.value=new Stats()
            statsRef.value?.showPanel(1)
            statsDivRef.value?.appendChild(statsRef.value.dom);
        }    

        function init(){
            const scene=new Scene()
            const axes=new AxesHelper(20)
            scene.add(axes)

            const renderer=new WebGLRenderer({antialias:true})
            renderer.setClearColor(new Color(0xEEEEEE))
            renderer.setSize(window.innerWidth,window.innerHeight)


            const planeGeometry=new PlaneGeometry(100,100);
            const planeMaterial=new MeshLambertMaterial({color:0xffffff});
            const plane=new Mesh(planeGeometry,planeMaterial)
            plane.rotation.x=-0.5*Math.PI
            plane.position.x=0
            plane.position.y=0;
            plane.position.z=0;
            //地面接受阴影
            plane.receiveShadow=true;
            scene.add(plane)

            //盒子几何体
            const cubeGeometry=new BoxGeometry(4,4,4)
            const cubeMaterial=new MeshLambertMaterial({color:0xff0000});
            const cube=new Mesh(cubeGeometry,cubeMaterial)
            //cube.rotation.x=-0.5*Math.PI
            //开启阴影
            cube.castShadow=true;
            cube.position.x=14
            cube.position.y=2;
            cube.position.z=2;
            scene.add(cube)

            //球形几何体
            const sphereGeometry=new SphereGeometry(4,20,20);
            const sphereMaterial=new MeshLambertMaterial({color:0x77777ff})
            const sphere=new Mesh(sphereGeometry,sphereMaterial);
            sphere.castShadow=true
            sphere.position.x=2
            sphere.position.y=2
            sphere.position.z=2
            scene.add(sphere)
            //光源效果
            let sptLight=new SpotLight(0xFFFFFF)
            sptLight.position.set(-40,60,-40)
            //sptLight.shadow.camera.near=40,
            //sptLight.shadow.camera.far=-40,
            //开启阴影
            sptLight.castShadow=true
            scene.add(sptLight)
            const ambientLight=new AmbientLight(0xcccccc)
            //scene.add(ambientLight)
  
            const camera=new PerspectiveCamera(45,window.innerWidth/window.innerHeight,0.1, 1000)
            camera.position.x=-30;
            camera.position.y=40;
            camera.position.z=30;
            camera.lookAt(scene.position)
            threeRef.value?.appendChild(renderer.domElement)
            //渲染器开启阴影
            renderer.shadowMap.enabled=true
            //renderer.render(scene,camera)

            let step=0;
            
            const controls={
                rotationSpeed:0.02,
                bouncingSpeed:0.03,
                //添加对象
                addCube:function(){
                      //盒子几何体
                    const cubeGeometry=new BoxGeometry(4,4,4)
                    const cubeMaterial=new MeshLambertMaterial({color:0xff0000});
                    const cube=new Mesh(cubeGeometry,cubeMaterial)
                    //cube.rotation.x=-0.5*Math.PI
                    //开启阴影
                    cube.castShadow=true;
                    cube.position.x=-30+Math.round(Math.random()*planeGeometry.parameters.width)
                    cube.position.y=Math.round(Math.random()*5);
                    cube.position.z=-20+Math.round(Math.random()*planeGeometry.parameters.height);
                    //设置Cube的名称
                    cube.name='cube-'+scene.children.length
                    scene.add(cube)
                    this.numberOfObjects=scene.children.length;
                },
                //删除对象
                removeCube:function(){
                
                    //过滤非Mesh 对象的，
                    const meshList=scene.children.filter((e)=>{
                        if(e instanceof Mesh&& e.name.startsWith('cube-')){
                            return true;
                        }
                        return false;
                    })
                    console.log(meshList)
                    scene.remove(meshList[meshList.length-1])
                    this.numberOfObjects=scene.children.length
                },
                //对象数据
                numberOfObjects:scene.children.length
            }
            const gui = new dat.GUI();
            gui.add(controls,'rotationSpeed',0,0.05)
            gui.add(controls,'bouncingSpeed',0,0.05)
            gui.add(controls,'addCube')
            gui.add(controls,'removeCube')
            //监听
            gui.add(controls,'numberOfObjects').listen()
            rendererScene()
            
            function rendererScene(){
                statsRef.value?.begin()
                scene.traverse((e)=>{
                    if(e instanceof Mesh&& e!=plane){
                        e.rotation.x+=controls.rotationSpeed
                        e.rotation.y+=controls.rotationSpeed
                        e.rotation.z+=controls.rotationSpeed
                    }
                })
                cube.rotation.x+=0.02;
                cube.rotation.y+=0.02;
                cube.rotation.z+=0.02;
                step+=0.02
                sphere.position.x=20+10*Math.cos(step);
                sphere.position.y=2+10*Math.abs(Math.cos(step));
                
                //
                requestAnimationFrame(rendererScene)
                renderer.render(scene,camera)
                statsRef.value?.end()
            }
          
        }

        function clear(){
            const dg=document.querySelector('.dg')
            if(dg){
                dg.innerHTML=''
            }
        }
        
        onMounted(()=>{
            clear()
            init()
            initStats()
        })
        return{
            statsDivRef,
            threeRef
        }
    },
})
</script>
<style scoped>
.stats{
    position:absolute;
    left: 0;
    top:0;
}
</style>