<template>
    <div id="container">
        <canvas ref="canvas"></canvas>
        <div ref="loading" class="loading">
            <p>Loading</p>
        </div>
    </div>
    <Prompt v-if="currentPrompt != null" ref="prompt" :infoInit="currentPrompt.info" :leftInit="currentPrompt.left" :topInit="currentPrompt.top"/>
</template>

<script>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { CSS2DRenderer, CSS2DObject } from 'three/examples/jsm/renderers/CSS2DRenderer.js';
import Prompt from "./Prompt.vue";
import { PROMPT_OBJECTS } from "../constants";
import { shallowRef } from '@vue/reactivity';

// Global GLTF loader
const loader = new GLTFLoader();

export default {
    data(){
        return {
            promptObjects: [],
            currentPrompt: null
        }
    },
    components: {
        Prompt
    },
    mounted(){
        this.$refs.loading.style.display = "flex";
        // Create scene
        const scene = new THREE.Scene();
        const camera = this.createCamera();
        const renderer = this.createRenderer(scene, camera);

        this.loadRoom(scene, camera);

        let labelRenderer = new CSS2DRenderer();
        labelRenderer.setSize( window.innerWidth, window.innerHeight );
        labelRenderer.domElement.style.position = 'absolute';
        labelRenderer.domElement.style.top = '0px';
        labelRenderer.domElement.style.pointerEvents = 'none';
        document.getElementById( 'container' ).appendChild( labelRenderer.domElement );

        // Animation loop
        const animate = () => {
            requestAnimationFrame(animate);
            renderer.render(scene, camera);
            labelRenderer.render( scene, camera );
        }
        animate();

        // Resize renderer when window size changes 
        window.onresize = () => {
            // Update renderer
            this.resizeRenderer(renderer, labelRenderer);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
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
                this.createPrompts(camera, scene);
                scene.add(gltf.scene);
            }, undefined, function (error) {
                console.error(error);
            } );
        },
        createPrompts: function(camera, scene) {

            for(let child of this.room.children){
                // Get object position and create prompt
                let prompt_info = PROMPT_OBJECTS.find(p => p.name === child.name);
                if(prompt_info) { 
                    const prompt = document.createElement('div');
                    const promptInner = document.createElement('div');
                    prompt.className = 'prompt';
                    promptInner.className = 'prompt-inner';
                    prompt.appendChild(promptInner);

                    const label = new CSS2DObject(prompt);
                    const hover = () => {
                        let coords = label.element.style.transform.slice(32, -1).split(",").map(c => parseFloat(c.slice(0, -2)));
                        this.currentPrompt = {
                            info: prompt_info, 
                            left: coords[0],
                            top: coords[1]
                        };

                        if(this.$refs.prompt) {
                            this.$refs.prompt.updateParams(coords[1], coords[0], prompt_info);
                            this.$refs.prompt.promptHover();
                        }
                    }
                    const leave = () => {
                        if(this.currentPrompt == null) return;
                        this.$refs.prompt.promptBlur();
                        this.currentPrompt.delete = true;
                        setTimeout(() => {
                            if(!this.currentPrompt.delete) return;
                            this.currentPrompt = null;
                        }, 750);
                    }
                    label.element.onmouseover = hover;
                    label.element.onmouseleave = leave;
                    label.element.onfocus = hover;
                    label.element.onblur = leave;
                    
                    label.position.copy(child.position);
                    scene.add(label);
                }
            }

            this.$refs.loading.style.display = "none";
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

            renderer.setPixelRatio(window.devicePixelRatio);
            renderer.setSize(window.innerWidth, window.innerHeight);

            renderer.render(scene, camera);
            renderer.outputEncoding = THREE.sRGBEncoding;
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;

            return renderer;
        },
        resizeRenderer: function (renderer, labelRenderer) { // Set's the renderers size to current window size
            renderer.setPixelRatio(window.devicePixelRatio);
            renderer.setSize(window.innerWidth, window.innerHeight);
            labelRenderer.setSize(window.innerWidth, window.innerHeight);
        }
    }
}
</script>

<style scoped lang="scss">
canvas{
    width: 100vw;
    height: 100vh;
}
.loading{
    display: none;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    align-items: center;
    justify-content: center;
    font-size: 2em;
    color: #fff;
    background: var(--secondary);
    p{
        display: flex;
        &::after{
            content: "";
            max-width: 0;
            display: block;
            animation: loading 1500ms linear infinite;
        }
    }
 
}

@keyframes loading {
    0%{
        content: "";
    }
    25%{
        content: ".";
    }
    50%{
        content: "..";
    }
    75%{
        content: "..."
    }
}
</style>