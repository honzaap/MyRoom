<template>
    <svg v-show="showLine" class="linesvg" ref="linesvg">
        <line :x2="left" :y2="top" stroke="#B4B4B4" stroke-width="2">
            <animate ref="lineAnimX" fill="freeze" attributeName="x1" :to="contentAnimX" :from="left" dur="0.3s" repeatCount="1" begin="indefinite"/>
            <animate ref="lineAnimY" fill="freeze" attributeName="y1" :to="contentAnimY" :from="top" dur="0.3s" repeatCount="1" begin="indefinite"/>
            <animate ref="lineAnimBackX" fill="freeze" attributeName="x1" :from="contentAnimX" :to="left" dur="0.3s" repeatCount="1" begin="indefinite"/>
            <animate ref="lineAnimBackY" fill="freeze" attributeName="y1" :from="contentAnimY" :to="top" dur="0.3s" repeatCount="1" begin="indefinite"/>
        </line>
    </svg>
    <div ref="promptanim" class="prompt-animation-container">
        <div ref="content" class="prompt-content">
            <div class="prompt-image" ref="canvascontainer" v-show="showLine">
                <model-viewer ref="viewer" style="width: auto; height: 100%;" v-pre loading="lazy" auto-rotate auto-rotate-delay="0" rotation-per-second="50deg"></model-viewer>
            </div>
            <div class="prompt-description">
                <h3>{{ info.title }}</h3>
                <div v-html="info.description"></div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        topInit: 0,
        leftInit: 0,
        infoInit: null
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
            showContent: false,
            info: null
        }
    },
    created(){
        this.top = this.topInit;
        this.left = this.leftInit;
        this.info = this.infoInit;
    },  
    mounted() {
        this.updateParams(this.topInit, this.leftInit, this.infoInit);
        this.promptHover();
    },
    methods: {
        updateParams(top, left, info){
            this.top = top;
            this.left = left;
            this.info = info;
        },
        promptHover() {
            this.$refs.viewer.src = `./assets/gltf/${this.info.model}`;
            this.hover = true;
            this.showLine = true;
            if(this.$refs.linesvg) this.$refs.linesvg.style.opacity = 1;

            // Position prompt popup
            let contentY = Math.max(this.top - 125 - this.contentShiftY, 0);
            let contentX = Math.max(this.left - 350 - this.contentShiftX, 0);

            this.contentAnimX = contentX + 350;
            this.contentAnimY = contentY + 125;

            this.$refs.promptanim.style.top =  `${contentY + 125}px`;
            this.$refs.promptanim.style.left = `${contentX + 350}px`;

            this.$refs.promptanim.style.width = 0;
            this.$refs.promptanim.style.height = 0;

            // Start line animation
            this.$refs.lineAnimX.beginElement();
            this.$refs.lineAnimY.beginElement();

            setTimeout(() => {
                if(!this.hover) return;
                this.$refs.promptanim.style.height = `125px`;
                this.$refs.promptanim.style.width = `350px`;

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
            let contentY = Math.max(this.top - 125 - this.contentShiftY, 0);
            let contentX = Math.max(this.left - 350 - this.contentShiftX, 0);

            this.contentAnimX = contentX + 350;
            this.contentAnimY = contentY + 125;

            this.$refs.promptanim.style.top =  `${contentY + 125}px`;
            this.$refs.promptanim.style.left = `${contentX + 350}px`;

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

<style lang="scss">
.prompt{
    display: flex;
    position: absolute;
    width: 20px;
    height: 20px;
    z-index: 1;
    justify-content: center;
    align-items: center;
    pointer-events: all;
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
    min-width: 350px;
    min-height: 125px;
    background: linear-gradient(102.98deg, var(--primary) 8%, var(--secondary) 23%, var(--tertiary) 68%);
    z-index: 3;
    font-size: 0.875rem;
    .prompt-image{
        min-width: 100px;
        max-width: 100px;
    }
    .prompt-description{
        padding: 6px 10px;
        flex-grow: 1;
        h3{
            font-size: 1.15em;
        }
        p{
            margin: 0;
        }
        ul{
            padding-left: 12px;
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