<template>
    <span @mouseenter="promptHover" @mouseleave="promptBlur" class="prompt" :style="{top: `${top}px`, left: `${left}px`}">
        <span class="prompt-inner"></span>
    </span>
    <svg v-show="showLine" class="linesvg" ref="linesvg">
        <line :x2="left + 8" :y2="top + 8" stroke="#B4B4B4" stroke-width="2">
            <animate ref="lineAnimX" fill="freeze" attributeName="x1" :to="contentAnimX" :from="left + 8" dur="0.3s" repeatCount="1" begin="0s"/>
            <animate ref="lineAnimY" fill="freeze" attributeName="y1" :to="contentAnimY" :from="top + 8" dur="0.3s" repeatCount="1" begin="0s"/>
            <animate ref="lineAnimBackX" fill="freeze" attributeName="x1" :from="contentAnimX" :to="left + 8" dur="0.3s" repeatCount="1" begin="0s"/>
            <animate ref="lineAnimBackY" fill="freeze" attributeName="y1" :from="contentAnimY" :to="top + 8" dur="0.3s" repeatCount="1" begin="0s"/>
        </line>
    </svg>
    <div ref="promptanim" class="prompt-animation-container">
        <div ref="content" class="prompt-content">
            <div class="prompt-image" ref="canvascontainer" v-show="showLine">
                <model-viewer style="width: auto; height: 100%;" v-pre src="./assets/gltf/grass.glb" loading="lazy" auto-rotate auto-rotate-delay="0" rotation-per-second="50deg"></model-viewer>
            </div>
            <div class="prompt-description">
                <h3>Placeholder</h3>
                <p>Insane placeholder</p>
                <ul>
                    <li>Turbo GIGA</li>
                    <li>Mega super</li>
                    <li>Extra big</li>
                </ul>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        topInit: 0,
        leftInit: 0
    },
    data(){
        return {
            top: 0,
            left: 0,
            contentShiftX: 70,
            contentShiftY: 80,
            contentAnimX: 0,
            contentAnimY: 0,
            hover: false,
            showLine: false,
            showContent: false
        }
    },
    created(){
        this.top = this.topInit;
        this.left = this.leftInit;
    },  
    mounted() {
        let mouseX = -1;
        let mouseY = -1;

        window.onmousemove = (e) => {
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

            this.updatePosition(this.left - deltaX / 60, this.top - deltaY / 50 - deltaX / 200);
        }
    },
    methods: {
        updatePosition(left, top) {
            this.left = left;
            this.top = top;
        },
        promptHover(){
            // Todo: keep within window bounds
            this.hover = true;
            this.showLine = true;
            this.$refs.linesvg.style.opacity = 1;

            // Position prompt popup
            let contentY = Math.max(this.top - this.$refs.content.clientHeight - this.contentShiftY, 0);
            let contentX = Math.max(this.left - this.$refs.content.clientWidth - this.contentShiftX, 0);

            this.contentAnimX = contentX + this.$refs.content.clientWidth;
            this.contentAnimY = contentY + this.$refs.content.clientHeight;

            this.$refs.promptanim.style.top =  `${contentY + this.$refs.content.clientHeight}px`;
            this.$refs.promptanim.style.left = `${contentX + this.$refs.content.clientWidth}px`;

            this.$refs.promptanim.style.width = 0;
            this.$refs.promptanim.style.height = 0;

            // Start line animation
            this.$refs.lineAnimX.beginElement();
            this.$refs.lineAnimY.beginElement();

            setTimeout(() => {
                if(!this.hover) return;
                this.$refs.promptanim.style.height = `${this.$refs.content.clientHeight}px`;
                this.$refs.promptanim.style.width = `${this.$refs.content.clientWidth}px`;

                this.$refs.promptanim.style.top =  `${contentY}px`;
                this.$refs.promptanim.style.left = `${contentX}px`;
                this.showContent = true;
            }, 300);
        },
        promptBlur(){
            this.hover = false;

            if(!this.showContent){
                this.$refs.linesvg.style.opacity = 0;
            }

            // Position prompt popup
            let contentY = Math.max(this.top - this.$refs.content.clientHeight - this.contentShiftY, 0);
            let contentX = Math.max(this.left - this.$refs.content.clientWidth - this.contentShiftX, 0);

            this.contentAnimX = contentX + this.$refs.content.clientWidth;
            this.contentAnimY = contentY + this.$refs.content.clientHeight;

            this.$refs.promptanim.style.top =  `${contentY + this.$refs.content.clientHeight}px`;
            this.$refs.promptanim.style.left = `${contentX + this.$refs.content.clientWidth}px`;

            this.$refs.promptanim.style.width = 0;
            this.$refs.promptanim.style.height = 0;
            
            this.showContent = false;

            setTimeout(() => {
                // Start line animation
                if(this.hover) return;
                this.$refs.lineAnimBackX.beginElement();
                this.$refs.lineAnimBackY.beginElement();
                setTimeout(() => {
                    if(!this.hover) this.showLine = false;
                }, 200)
            }, 450);
        }
    }
}
</script>

<style lang="scss" scoped>
.prompt{
    display: flex;
    position: absolute;
    width: 20px;
    height: 20px;
    z-index: 1;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    &::after{
        position: absolute;
        content: "";
        display: block;
        border: 4px solid var(--tertiary);
        width: 20px;
        height: 20px;
        border-radius: 50%;
        animation: prompt 3000ms ease infinite;
    }
    .prompt-inner{
        width: 10px;
        height: 10px;
        border: 3px solid var(--tertiary);
        border-radius: 50%;
        opacity: 0.7;
        transition: all 300ms ease;
    }
    &:hover{
        &::after{
            opacity: 0;
            animation: none;
            transform: scale(1.5);
        }
        .prompt-inner{
            transform: scale(1.5);
            opacity: 1;
        }
    }
}

.linesvg{
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
}

svg{
    transition: opacity 200ms ease;
    pointer-events: none;
}

.prompt-content{
    position: absolute;
    display: flex;
    align-items: stretch;
    color: #fff;
    width: 300px;
    background: linear-gradient(102.98deg, var(--primary) 8%, var(--secondary) 23%, var(--tertiary) 68%);
    z-index: 3;
    font-size: 0.875rem;
    .prompt-image{
        width: 80px;
    }
    .prompt-description{
        padding: 6px 10px;
        flex-grow: 1;
        h3{
            font-size: 1.25em;
        }
        p{
            margin: 0;
        }
        ul{
            padding-left: 18px;
        }
    }
}

.prompt-animation-container{
    position: absolute;
    transition: all 500ms ease;
    overflow: hidden;
}

@keyframes prompt{
    0%{
        transform: scale(0.5);
        opacity: 0;
    }
    70%{
        opacity: 0.4;
    }
    100%{
        transform: scale(1.35);
        opacity: 0;
    }
}
</style>