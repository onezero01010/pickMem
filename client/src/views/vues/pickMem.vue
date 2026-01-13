<template>
    <div class="pick-mem h-100">
        <div class="content" style="overflow-y: auto;">
            <nav-bar v-if="step > 3" class="nav" :msg="msg[step]" @on-next="nextStep" @on-previous="previousStep" :step="step"></nav-bar>
            <div class="body container">
                <step-one class="step-0" v-if="step==0" @frame-selected="nextStep" @character-selection-needed="goToCharacterSelection"></step-one>
                <step-one-point-five class="step-0-5" v-else-if="step==0.5" @characters-selected="goToPhotoStep"></step-one-point-five>
                <step-two class="step-1" v-else-if="step==1" @photos-complete="nextStep"></step-two>
                <step-three class="step-2" v-else-if="step==2" @photos-selected="nextStep"></step-three>
                <step-four class="step-3" v-else-if="step==3" @on-previous="previousStep"></step-four>
                <step-five class="step-4" v-else-if="step==4"></step-five>
                <step-result class="step-5" v-else @on-previous="previousStep"></step-result>
            </div>
        </div>
        <footer-bar v-if="step > 3" class="footer"></footer-bar>
    </div>
    
</template>

<script>
import navBar from '../../components/nav.vue'
import footerBar from '../../components/footer.vue'
import stepOne from '../steps/stepOne.vue';
import stepOnePointFive from '../steps/stepOnePointFive.vue';
import stepTwo from '../steps/stepTwo.vue';
import stepThree from '../steps/stepThree.vue';
import stepFour from '../steps/stepFour.vue';
import stepFive from '../steps/stepFive.vue';
import stepResult from '../steps/stepResult.vue';

export default {
    name: 'PickMem',
    components: {
        navBar,
        footerBar,
        stepOne,
        stepOnePointFive,
        stepTwo,
        stepThree,
        stepFour,
        stepFive,
        stepResult,
    },
    data() {
        return {
            step: 0,
            msg: [
                '액자를 골라보세요!',
                '가장 멋있고 예쁘게 찍어보세요',
                '어렵겠지만 몇 개만 골라주세요',
                '마음가는 대로 액자를 꾸며봐요!',
                '이번에는 사진을 꾸며볼까요',
                '마음에 드시나요?'
            ]
        }
    },
    methods: {
        nextStep() {
            let canNext = this.$store.getters.getNext;

            if (this.step == 0 && !canNext) {
                this.$Utils.toast('액자를 골라주세요.')
                return;
            } else if (this.step == 1 && !canNext) {
                this.$Utils.toast('사진이 부족합니다.')
                return;
            } else if (this.step == 2 && !canNext) {
                this.$Utils.toast('사진을 모두 골라주세요.')
                return;
            }

            if (this.step == 5 || !canNext) return;

            if (this.step == 1 && Object.keys(this.$store.getters.getImages).length < 6) return;

            
            this.step += 1;
        },

        previousStep() {
            if (this.step == 0) this.$router.push('/');
            if (this.step == 0.5) {
                this.step = 0;
                return;
            }

            this.step -= 1;
        },
        goToCharacterSelection() {
            this.step = 0.5;
        },
        goToPhotoStep() {
            this.step = 1;
        },
    },
}
</script>

<style lang="scss">

.body {
    margin-top: 0;
}

.step-0,
.step-0-5 {
    margin-top: 0 !important;
}

.step {
    padding-bottom: 50px;
}

.nav {
    position: fixed;
    width: 100%;
    z-index: 1000;
    background-color: #FFF;
}

.content {
    height: 100%;
}

.footer {
    height: 50px;
    position : relative;
    background-color: #FFF;
    z-index: 1000;
    transform : translateY(-100%);
}

.custom-toast {
    background-color: red;
}
</style>