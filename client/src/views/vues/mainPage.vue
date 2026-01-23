<template>
    <div class="main-page">
        <div class="content-container">
            <div class="title-container">
                <h1 class="main-title">VIDES네컷</h1>
                <p class="subtitle">무제한으로 원하는 모습을 찍고 꾸미세요!</p>
            </div>
            <div class="start-button-container">
                <button class="start-btn" @click="init();$router.push('/pickMem')">
                    시작하기
                </button>
            </div>
        </div>
    </div>    
</template>

<script>
export default {
    name: 'MainPage',
    mounted() {
        this.init();
    },
    methods: {
        init() {
            this.$store.commit('resetState');
        }
    }
}
</script>

<style lang="scss" scoped>
.main-page {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    min-height: 100vh;
    background: linear-gradient(135deg, #e0f2fe 0%, #bae6fd 100%);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 15px 10px;
}

.content-container {
    background: white;
    border-radius: 15px;
    padding: 30px 25px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
    max-width: min(calc(100vw - 30px), 600px);
    width: 100%;
    text-align: center;
}

.title-container {
    margin-bottom: 25px;
}

.main-title {
    font-size: 32px;
    font-weight: bold;
    color: #1e3a8a;
    margin: 0 0 12px 0;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

.subtitle {
    font-size: 18px;
    font-weight: 600;
    color: #3b82f6;
    margin: 0;
}

.start-button-container {
    margin-top: 20px;
}

.start-btn {
    padding: 12px 35px;
    background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
    color: white;
    border: none;
    border-radius: 10px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
        box-shadow: 0 4px 12px rgba(59, 130, 246, 0.3);

    &:hover {
        transform: translateY(-3px);
        box-shadow: 0 6px 20px rgba(59, 130, 246, 0.4);
    }

    &:active {
        transform: translateY(-1px);
    }
}

@media (max-width: 768px) {
    .content-container {
        padding: 40px 30px;
    }

    .main-title {
        font-size: 36px;
    }

    .subtitle {
        font-size: 20px;
    }

    .start-btn {
        padding: 14px 40px;
        font-size: 18px;
    }
}
</style>