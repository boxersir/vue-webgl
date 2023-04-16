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
        const Boid = function () {

            let goal;

            let vector = new THREE.Vector3();

            let neighbourhoodRadius = 50;

            let width = 200;

            let height = 200;

            let depth = 100;

            let maxSpeed = 10;

            let maxSteerForce = 0.75;

            let acceleration = new THREE.Vector3();

            let avoidWalls = true;

            this.position = new THREE.Vector3();

            this.velocity = new THREE.Vector3();

            this.setGoal = (target) => {

                goal = target;

            }

            this.setAvoidWalls = (value) => {

                avoidWalls = value;

            }

            this.setWorldSize = (width, height, depth) => {

                width = width;

                depth = depth;

                height = height;

            }

            this.alignment = (boids) => {

                let boid;

                let velSum = new THREE.Vector3();

                let count = 0;

                for (let i = 0; i < boids.length; i += 1) {

                    if (Math.random() > 0.6) continue;

                    boid = boids[i];

                    let dist = boid.position.distanceTo(this.position);

                    if (dist > 0 && dist < neighbourhoodRadius) {

                        velSum.add(boid.velocity);

                        count += 1;

                    }

                }

                if (count > 0) {

                    velSum.divideScalar(count);

                    let l = velSum.length();

                    if (l > maxSteerForce) {

                        velSum.divideScalar(l / maxSteerForce)

                    }

                }

                return velSum;

            }

            this.cohesion = (boids) => {

                let boid;

                let dist;

                let posSum = new THREE.Vector3();

                let steer = new THREE.Vector3();

                let count = 0;

                for (let i = 0, il = boids.length; i < il; i += 1) {

                    if (Math.random() > 0.6) continue;

                    boid = boids[i];

                    dist = boid.position.distanceTo(this.position);

                    if (dist > 0 && dist < neighbourhoodRadius) {

                        posSum.add(boid.position);

                        count += 1;

                    }

                }

                if (count > 0) {

                    posSum.divideScalar(count);

                }

                steer.subVectors(posSum, this.position);

                let l = steer.length();

                if (l > maxSteerForce) {

                    steer.divideScalar(l / maxSteerForce);

                }

                return steer;

            }

            this.separation = (boids) => {

                let boid;

                let dist;

                let posSum = new THREE.Vector3();

                let repulse = new THREE.Vector3();

                for (let i = 0, il = boids.length; i < il; i += 1) {

                    if (Math.random() > 0.6) continue;

                    boid = boids[i];

                    dist = boid.position.distanceTo(this.position);

                    if (dist > 0 && dist < neighbourhoodRadius) {

                        repulse.subVectors(this.position, boid.position);

                        repulse.normalize();

                        repulse.divideScalar(dist);

                        posSum.add(repulse);

                    }

                }

                return posSum;

            }

            this.move = () => {

                this.velocity.add(acceleration);

                let l = this.velocity.length();

                if (l > maxSpeed) {

                    this.velocity.divideScalar(l / maxSpeed);

                }

                this.position.add(this.velocity);

                acceleration.set(0, 0, 0);

            }

            this.reach = (target, amount) => {

                let steer = new THREE.Vector3();

                steer.subVectors(target, this.position);

                steer.multiplyScalar(amount);

                return steer;

            }

            this.flock = (boids) => {

                if (goal) {

                    acceleration.add(this.reach(goal, 0.1));

                }

                acceleration.add(this.alignment(boids));

                acceleration.add(this.cohesion(boids));

                acceleration.add(this.separation(boids));

            }

            this.avoid = (target) => {

                let steer = new THREE.Vector3();

                steer.copy(this.position);

                steer.sub(target);

                steer.multiplyScalar(1 / this.position.distanceToSquared(target))

                return steer;

            }

            this.repulse = () => {

                let dist = this.position.distanceTo(target);

                if (dist < 150) {

                    let steer = new THREE.Vector3();

                    steer.subVectors(this.position, target);

                    steer.multiplyScalar(0.5 / dist);

                    acceleration.add(steer);

                }

            }

            this.run = (boids) => {

                if (avoidWalls) {

                    vector.set(-width, this.position.y, this.position.z);

                    vector = this.avoid(vector);

                    vector.multiplyScalar(5);

                    acceleration.add(vector);

                    vector.set(width, this.position.y, this.position.z);

                    vector = this.avoid(vector);

                    vector.multiplyScalar(5);

                    acceleration.add(vector);

                    vector.set(this.position.x, - height, this.position.z);

                    vector = this.avoid(vector);

                    vector.multiplyScalar(5);

                    acceleration.add(vector);

                    vector.set(this.position.x, height, this.position.z);

                    vector = this.avoid(vector);

                    vector.multiplyScalar(5);

                    acceleration.add(vector);

                    vector.set(this.position.x, this.position.y, -depth);

                    vector = this.avoid(vector);

                    vector.multiplyScalar(5);

                    acceleration.add(vector);

                    vector.set(this.position.x, this.position.y, depth);

                    vector = this.avoid(vector);

                    vector.multiplyScalar(5);

                    acceleration.add(vector);

                }

                if (Math.random() > 0.5) {

                    this.flock(boids);

                }

                this.move();

            }

        }

        class WordFish {

            constructor(props) {

                this.texture = props.texture;

                this.speed = 0.9;

                this.angle = Math.random() * 360;

                this.offsets = [

                    Math.random() * 0.6,

                    Math.random() * 0.35,

                    Math.random() * 0.7

                ];

                this.mesh = null;

            }

            init(scene) {

                const geometry = new THREE.PlaneGeometry(10, 5, 10, 1);

                const material = new THREE.MeshBasicMaterial({

                    map: this.texture,

                    transparent: true,

                    side: THREE.DoubleSide

                });

                this.mesh = new THREE.Mesh(geometry, material);

                this.mesh.position.z = -100;

                scene.add(this.mesh);

                return this;

            }

            update(target) {

                this.texture.needsUpdate = true;

                this.mesh.geometry.verticesNeedUpdate = true;

                this.mesh.geometry.vertices.forEach((v, i) => {

                    if (i % 11 === 0) {

                        if (i === 11) {

                            v.x += (target.x - v.x) * (this.speed * 2);

                            v.y += (target.y - v.y) * (this.speed * 2);

                            v.z += (target.z - v.z) * (this.speed * 2);

                        } else {

                            let midv = this.mesh.geometry.vertices[11];

                            v.x += midv.x - v.x + Math.sin(this.angle / 10) * 100;

                            v.y += midv.y - v.y + Math.cos(this.angle / 2) * 100;

                            v.z += midv.z - v.z + Math.sin(this.angle / 10) * 20;

                        }

                    } else {

                        let prevVert = this.mesh.geometry.vertices[i - 1];

                        let dx = prevVert.x - v.x;

                        let dy = prevVert.y - v.y;

                        let dz = prevVert.z - v.z;

                        v.x += dx * (this.speed * 0.45);

                        v.y += dy * (this.speed * 0.45);

                        v.z += dz * (this.speed * 0.45);

                    }

                });

                this.angle += 0.095;

            }

        }

        const makeTexture = (function () {

            let canvas = document.createElement('canvas');

            let ctx = canvas.getContext('2d');

            let width = canvas.width = 128;

            let height = canvas.height = 32;

            return (word, a) => {

                //this.ctx.fillStyle = '#111';

                ctx.clearRect(0, 0, width, height);

                ctx.fillStyle = '#111';

                ctx.font = '24px Arial';

                ctx.fillText(word, width / 2 - ctx.measureText(word).width / 2, height / 2 + 5);

                let texture = new THREE.Texture(canvas);

                texture.anisotropy = maxAnisotropy

                return texture;

            }

        }());

        //

        let stats = new Stats();

        stats.showPanel(0); // 0: fps, 1: ms, 2: mb, 3+: custom

        stats.domElement.style.position = 'fixed';

        stats.domElement.style.top = '10px';

        stats.domElement.style.left = '10px';

        document.body.appendChild(stats.domElement);

        let width = window.innerWidth;

        let height = window.innerHeight;

        const scene = new THREE.Scene();

        const camera = new THREE.PerspectiveCamera(45, width / height, 0.1, 10000);

        const renderer = new THREE.WebGLRenderer();

        const boids = [];

        const fishes = [];

        let cameraOffset = 800;

        camera.position.set(cameraOffset, cameraOffset, cameraOffset);

        camera.lookAt(new THREE.Vector3(0, 0, 0));

        renderer.setSize(width, height);

        renderer.setClearColor(0xefefef);

        renderer.setPixelRatio(window.devicePixelRatio);

        document.body.appendChild(renderer.domElement);

        let maxAnisotropy = renderer.getMaxAnisotropy();

        const a = new THREE.GridHelper(600, 5);

        scene.add(a);

        for (let i = 0; i < 80; i += 1) {

            let boid = new Boid();

            boid.position.x = Math.random() * 400 - 200;

            boid.position.y = Math.random() * 400 - 200;

            boid.position.z = Math.random() * 400 - 200;

            boid.velocity.x = Math.random() * 2 - 1;

            boid.velocity.y = Math.random() * 2 - 1;

            boid.velocity.z = Math.random() * 2 - 1;

            boid.setWorldSize(250, 250, 250);

            boid.setAvoidWalls(true);

            boids.push(boid);

            fishes.push(new WordFish({

                texture: makeTexture('techbrood', maxAnisotropy)

            }).init(scene))

        }

        const renderFrame = (ts) => {

            stats.begin();

            renderer.render(scene, camera);

            for (let i = 0; i < boids.length; i += 1) {

                let boid = boids[i];

                let fish = fishes[i];

                boid.run(boids);

                fish.update(boid.position);

            }

            stats.end();

            requestAnimationFrame(renderFrame);

        }

        renderFrame();
    }
})