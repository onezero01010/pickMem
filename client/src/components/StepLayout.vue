<template>
    <div class="step-layout">
        <h1 class="page-title">살레네컷</h1>
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
    background: linear-gradient(135deg, #f9f4ff 0%, #f1e6ff 100%);
    padding: 40px 20px;
    overflow-y: auto;
    overflow-x: hidden;
}

.page-title {
    font-size: 48px;
    font-weight: bold;
    color: #ff6b9d;
    margin: 0 auto 30px;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
    text-align: center;
    max-width: 1500px;
}

.content-container {
    background: white;
    border-radius: 20px;
    padding: 40px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
    max-width: 1500px;
    margin: 0 auto;
    margin-top: 0;
}

.title-container {
    text-align: center;
    margin-bottom: 30px;
}

.section-title {
    font-size: 28px;
    font-weight: 600;
    color: #8b5cf6;
    margin: 0 0 10px 0;
}

.instruction-text {
    font-size: 16px;
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

