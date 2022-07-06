<template>
    <canvas ref="canvas"></canvas>
    <component :ref="prompt.name" v-for="prompt in promptObjects" :is="prompt.elem" :leftInit="prompt.left" :topInit="prompt.top" />
</template>

<script>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import Prompt from "./Prompt.vue";
import { shallowRef } from '@vue/reactivity';

// Global GLTF loader
const loader = new GLTFLoader();

export default {
    data(){
        return {
            promptObjects: [],
        }
    },
    components: {
        Prompt
    },
    mounted(){
        // Create scene
        const scene = new THREE.Scene();
        const camera = this.createCamera();
        const renderer = this.createRenderer(scene, camera);

        this.loadRoom(scene, camera);

        // Animation loop
        const animate = () => {
            requestAnimationFrame(animate);
            renderer.render(scene, camera);
        }
        animate();

        // Resize renderer when window size changes 
        window.onresize = () => {
            // Update renderer
            this.resizeRenderer(renderer);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();

            this.updatePrompts(camera);
        }
          
        function rotateScene(deltaX, deltaY) {
            scene.rotation.y += deltaX / 10000;
            scene.rotation.x += deltaY / 10000;
            scene.rotation.z -= deltaY / 10000;
        }

        let mouseX = -1;
        let mouseY = -1;

        this.$refs.canvas.addEventListener("mousemove", (e) => {
            e.preventDefault();

            // Handle mouse input to make rotating effect
            if(mouseX === -1 && mouseY === -1){
                mouseX = e.clientX;
                mouseY = e.clientY;
            }
            let deltaX = e.clientX - mouseX;
            let deltaY = e.clientY - mouseY;
            mouseX = e.clientX;
            mouseY = e.clientY;
            rotateScene(deltaX, deltaY);
        });

        this.$refs.canvas.addEventListener("touchmove", (e) => {
            e.preventDefault();

            // Handle mouse input to make rotating effect
            if(mouseX === -1 && mouseY === -1){
                mouseX = e.touches[0].clientX;
                mouseY = e.touches[0].clientY;
            }
            let deltaX = e.touches[0].clientX - mouseX;
            let deltaY = e.touches[0].clientY - mouseY;
            mouseX = e.touches[0].clientX;
            mouseY = e.touches[0].clientY;
            rotateScene(deltaX, deltaY);
        });
    },
    methods: {
        loadRoom: function(scene, camera) {
            loader.load("./assets/gltf/room.glb", (gltf) => {
                this.room = gltf.scene;
                this.createPrompts(camera);
                scene.add(gltf.scene);
            }, undefined, function (error) {
                console.error(error);
            } );
        },
        createPrompts: function(camera) {
            for(let child of this.room.children){
                // Get object position and create prompt
                if(child.name === "Keyboard") { // Replace with array of objects or some shit
                    let promptVector = child.position.clone();
                    promptVector.project(camera);
                    promptVector.x = ( promptVector.x + 1) * window.innerWidth / 2;
                    promptVector.y = - ( promptVector.y - 1) * window.innerHeight / 2;
                    promptVector.z = 0;

                    this.promptObjects.push(
                        {
                            name: child.name,
                            elem: shallowRef(Prompt),
                            top: promptVector.y - 10,
                            left: promptVector.x - 10
                        }
                    );
                }
            }
        },
        updatePrompts: function(camera) {
            for(let child of this.room.children){
                // Get object position and update prompt
                if(child.name === "Keyboard") { // Replace with array of objects or some shit
                    let promptVector = child.position.clone();
                    promptVector.project(camera);
                    promptVector.x = ( promptVector.x + 1) * window.innerWidth / 2;
                    promptVector.y = - ( promptVector.y - 1) * window.innerHeight / 2;
                    promptVector.z = 0;

                    this.$refs[child.name][0].updatePosition(promptVector.x - 10, promptVector.y - 10);
                }
            }
        },
        createCamera: function () { // Create and cofigure camera and return it 
            const camera = new THREE.PerspectiveCamera(47, window.innerWidth / window.innerHeight, 1, 15);
            camera.position.set(4.4, 2, 4.3);
            camera.rotation.x = THREE.MathUtils.degToRad(-32);
            camera.rotation.y = THREE.MathUtils.degToRad(42);
            camera.rotation.z = THREE.MathUtils.degToRad(22);

            return camera;
        },
        createRenderer: function (scene, camera) { // Create and configure renderer and return it 
            const renderer = new THREE.WebGLRenderer({
                powerPreference: "high-performance",
                antialias: true,
                canvas: this.$refs.canvas,
                alpha: true
            });

            renderer.setClearColor( 0x000000, 0 );

            this.resizeRenderer(renderer);

            renderer.render(scene, camera);
            renderer.outputEncoding = THREE.sRGBEncoding;
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;

            return renderer;
        },
        resizeRenderer: function (renderer) { // Set's the renderers size to current window size
            renderer.setPixelRatio(window.devicePixelRatio);
            renderer.setSize(window.innerWidth, window.innerHeight);
        }
    }
}
</script>

<style scoped>
canvas{
    width: 100vw;
    height: 100vh;
}
</style>