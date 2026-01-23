<template>
    <div class="step-layout">
        <h1 class="page-title">VIDES네컷</h1>
        <div class="content-container">
            <div v-if="showTitle" class="title-container">
                <h2 class="section-title">{{ title }}</h2>
                <p v-if="instruction" class="instruction-text">{{ instruction }}</p>
            </div>
            <slot></slot>
        </div>
    </div>
</template>

<script>
export default {
    name: 'StepLayout',
    props: {
        title: {
            type: String,
            default: ''
        },
        instruction: {
            type: String,
            default: ''
        },
        showTitle: {
            type: Boolean,
            default: true
        }
    }
}
</script>

<style lang="scss" scoped>
.step-layout {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100vh;
    background: linear-gradient(135deg, #f0f9ff 0%, #e0f7fa 100%);
    padding: 20px 15px;
    overflow-y: auto;
    overflow-x: hidden;
}

.page-title {
    font-size: 36px;
    font-weight: bold;
    color: #1e3a8a;
    margin: 0 auto 20px;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
    text-align: center;
    max-width: min(calc(100vw - 80px), 1400px);
}

.content-container {
    background: #F5F5F5;
    border-radius: 20px;
    padding: 20px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
    max-width: min(calc(100vw - 80px), 1400px);
    margin: 0 auto;
    margin-top: 0;
    box-sizing: border-box;
}

.title-container {
    text-align: center;
    margin-bottom: 20px;
}

.section-title {
    font-size: 24px;
    font-weight: 600;
    color: #3b82f6;
    margin: 0 0 8px 0;
}

.instruction-text {
    font-size: 14px;
    color: #666;
    margin: 0;
}

@media (max-width: 768px) {
    .page-title {
        font-size: 36px;
    }
    
    .content-container {
        padding: 20px;
    }
    
    .section-title {
        font-size: 22px;
    }
}
</style>

