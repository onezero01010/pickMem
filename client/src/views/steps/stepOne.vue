<template>
    <div class="step-one">
        <h1 class="page-title">살레네컷</h1>
        <div class="frame-selection-container">
            <div class="title-container">
                <h2 class="subtitle">사진 촬영 프레임 선택</h2>
            </div>
            <div class="frame-options-wrapper">
                <div class="frame-option" :class="{ 'selected': targetFrame == '1x4' }" @click="selectFrame('1x4')">
                    <img class="frame-preview" src="@/assets/frame/1_4.png" alt="세로 4컷 프레임" draggable="false">
                </div>
                <div class="or-divider">OR</div>
                <div class="frame-option" :class="{ 'selected': targetFrame == '2x2' }" @click="selectFrame('2x2')">
                    <img class="frame-preview frame-preview-large" src="@/assets/frame/2_2.png" alt="2x2 그리드 프레임" draggable="false">
                </div>
            </div>
        </div>
    </div>
</template>

<script>

export default {
    name: 'StepOne',
    data() {
        return {
            targetFrame: null,
        }
    },
    mounted() {
        this.init();
        this.targetFrame = this.$store.getters.getFrame;
        if (!this.targetFrame) this.$store.commit('setNext', false);
        else this.$store.commit('setNext', true);
    },
    beforeDestroy() {
        this.nextStep();
    },
    methods: {
        selectFrame(id) {
            this.targetFrame = id;
            this.$store.commit('setNext', true);
            this.$store.commit('setTable', {
                'columns': parseInt(id.split('x')[0]),
                'rows': parseInt(id.split('x')[1]),
            });
            this.$store.commit('setFrame', id);
            
            // 프레임 선택 시 자동으로 다음 단계로 이동
            this.$nextTick(() => {
                this.$emit('frame-selected');
            });
        },
        nextStep() {
            if (!this.targetFrame) return;
            this.$store.commit('setFrame', this.targetFrame);
        },
        init() {
            this.$store.commit('resetState');
        },
    }
}
</script>

<style lang="scss" scoped>
.step-one {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    min-height: 100vh;
    background: linear-gradient(135deg, #e8d5ff 0%, #d4b3ff 100%);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    padding: 40px 20px;
    padding-top: 40px;
    overflow-y: auto;
}

.page-title {
    font-size: 48px;
    font-weight: bold;
    color: #ff6b9d;
    margin: 0 0 30px 0;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
    text-align: center;
}

.title-container {
    text-align: center;
    margin-bottom: 30px;
}

.main-title {
    font-size: 48px;
    font-weight: bold;
    color: #ff6b9d;
    margin: 0 0 15px 0;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

.subtitle {
    font-size: 28px;
    font-weight: 600;
    color: #8b5cf6;
    margin: 0;
}

.frame-selection-container {
    background: white;
    border-radius: 20px;
    padding: 40px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
    max-width: 1500px;
    width: 100%;
    margin-top: 0;
}

.frame-options-wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 30px;
}

.frame-option {
    flex: 1;
    cursor: pointer;
    border-radius: 15px;
    padding: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
    
    &.selected {
        background: #f3e8ff;
        box-shadow: 0 0 0 3px #8b5cf6;
    }
}

.frame-label {
    font-size: 16px;
    font-weight: 600;
    color: #333;
    margin-bottom: 15px;
}

.frame-preview {
    width: 100%;
    max-width: 200px;
    height: auto;
    object-fit: contain;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    transition: box-shadow 0.3s ease, transform 0.3s ease;
    
    &:hover {
        box-shadow: 0 8px 20px rgba(0, 0, 0, 0.25);
        transform: translateY(-3px);
    }
    
    &.frame-preview-large {
        max-width: 400px;
    }
}

.or-divider {
    font-size: 24px;
    font-weight: bold;
    color: #333;
    padding: 0 20px;
}

@media (max-width: 768px) {
    .page-title {
        font-size: 36px;
    }
    
    .frame-selection-container {
        flex-direction: column;
        gap: 20px;
    }
    
    .or-divider {
        padding: 10px 0;
    }
    
    .main-title {
        font-size: 36px;
    }
    
    .subtitle {
        font-size: 22px;
    }
}
</style>