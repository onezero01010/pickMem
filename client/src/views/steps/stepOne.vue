<template>
    <StepLayout title="사진 촬영 프레임 선택">
        <div class="frame-options-wrapper">
            <div class="frame-option" :class="{ 'selected': targetFrame == '1x4' }" @click="selectFrame('1x4')">
                <img class="frame-preview" src="@/assets/frame/1_4.png" alt="세로 4컷 프레임" draggable="false">
            </div>
            <div class="or-divider">OR</div>
            <div class="frame-option" :class="{ 'selected': targetFrame == '2x2' }" @click="selectFrame('2x2')">
                <img class="frame-preview frame-preview-large" src="@/assets/frame/2_2.png" alt="2x2 그리드 프레임" draggable="false">
            </div>
        </div>
    </StepLayout>
</template>

<script>
import StepLayout from '@/components/StepLayout.vue'

export default {
    name: 'StepOne',
    components: {
        StepLayout
    },
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
    .frame-options-wrapper {
        flex-direction: column;
        gap: 20px;
    }
    
    .or-divider {
        padding: 10px 0;
    }
}
</style>