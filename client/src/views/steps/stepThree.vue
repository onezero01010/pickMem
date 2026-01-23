<template>
    <StepLayout 
        title="사진 선택" 
        :instruction="`순서대로 ${rows * columns}장의 사진을 선택하세요`"
    >
        <div class="photo-grid">
                <div 
                    v-for="(id, idx) of Object.keys(images)" 
                    :key="idx" 
                    class="photo-item"
                    :class="{ 'selected': Object.values(getSelectList).includes(id) }"
                    @click="selectToClick(getSelectTargetLen, id, images[id])"
                >
                    <img :src="images[id]" alt="사진" draggable="false">
                    <div v-if="Object.values(getSelectList).includes(id)" class="selection-badge">
                        {{ getSelectionNumber(id) }}
                    </div>
                </div>
            </div>
    </StepLayout>
</template>

<script>
import basicFrame from './frames/basicFrame.vue'
import StepLayout from '@/components/StepLayout.vue'

export default {
    name: 'StepThree',
    components: {
        basicFrame,
        StepLayout
    },

    data() {
        return {
            rows: 2,
            columns: 1,
            width: 500,
            images: {},
            selectTarget: {},
            selectList: {},
            frame: null,
            queue: [],
        }
    },

    created() {   
    },

    mounted() {
        let table = this.$store.getters.getFrame

        this.rows = table.split('x')[0];
        this.columns = table.split('x')[1];

        this.selectTarget = this.$store.getters.getTargets;
        this.selectList = this.$store.getters.getTargetList;
        this.images = this.$store.getters.getImages;
        this.frame = this.$store.getters.getFrame;

        this.queue = Array.from({length: this.rows * this.columns}, (v, i) => { if (!this.selectTarget[i + 1]) return i + 1 });
        this.queue = this.queue.filter(item => item);
        this.$store.commit('setUpdateQueue', this.queue);

        if (Object.keys(this.$store.getters.getTargets).length == this.rows * this.columns) this.$store.commit('setNext', true);
        else this.$store.commit('setNext', false);
        
    },

    destroyed() {
        
    },

    methods: {
        selectToClick(idx, id, src) {
            this.selectTarget = this.$store.getters.getTargets;
            this.selectList = this.$store.getters.getTargetList;
            
            // 이미 선택된 사진이면 선택 해제
            if (Object.values(this.getSelectList).includes(id)) {
                // 선택 해제 처리
                for (let key in this.selectList) {
                    if (this.selectList[key] === id) {
                        const position = parseInt(key);
                        // 해당 위치의 타겟 제거
                        this.$delete(this.selectTarget, position);
                        this.$delete(this.selectList, key);
                        // 큐에 위치 추가 (정렬)
                        this.queue = this.$store.getters.getRemoveQueues;
                        this.queue.push(position);
                        this.queue.sort((a, b) => a - b);
                        this.$store.commit('setUpdateQueue', this.queue);
                        this.$store.commit('setTargets', this.selectTarget);
                        this.$store.commit('setTmpTargets', this.selectTarget);
                        this.$store.commit('setTargetList', this.selectList);
                        break;
                    }
                }
                this.selectTarget = this.$store.getters.getTargets;
                this.$store.commit('setNext', false);
                return;
            }

            let table = this.frame.split('x');
            const maxCount = parseInt(table[0]) * parseInt(table[1]);

            if (Object.keys(this.selectTarget).length >= maxCount) {
                this.$Utils.toast('이미 모두 골랐어요.')
                return;
            } else {
                this.queue = this.$store.getters.getRemoveQueues;
                this.$set(this.selectList, idx + 1, id);

                let recoverKey = this.queue.shift();

                this.$store.commit('setUpdateQueue', this.queue);
                this.$store.commit('setTarget', [recoverKey, src]);
                this.$store.commit('setTmpTarget', [recoverKey, src]);
                this.$store.commit('setTargetList', this.selectList);

                this.selectTarget = this.$store.getters.getTargets;
            }

            if (Object.keys(this.$store.getters.getTargets).length == this.rows * this.columns) {
                this.$store.commit('setNext', true);
                // 4개 사진 다 선택되면 자동으로 다음 페이지로 이동
                setTimeout(() => {
                    this.$emit('photos-selected');
                }, 500);
            } else {
                this.$store.commit('setNext', false);
            }
        },
        getSelectionNumber(id) {
            const list = this.getSelectList;
            for (let key in list) {
                if (list[key] === id) {
                    return parseInt(key);
                }
            }
            return null;
        },
    },
    computed: {
        getSelectList() {
            return this.selectList;
        },
        getSelectTargetLen() {
            return Object.keys(this.selectTarget).length;
        },
    },
}
</script>

<style lang="scss" scoped>
.photo-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 15px;
    margin-top: 20px;
}

.photo-item {
    position: relative;
    aspect-ratio: 1;
    border-radius: 10px;
    overflow: hidden;
    cursor: pointer;
    border: 3px solid transparent;
    transition: all 0.3s ease;
    
    &:hover {
        transform: translateY(-3px);
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
    }
    
    &.selected {
        border-color: #1e3a8a;
        box-shadow: 0 0 0 2px #1e3a8a;
    }
    
    img {
        width: 100%;
        height: 100%;
        object-fit: cover;
    }
}

.selection-badge {
    position: absolute;
    top: 10px;
    right: 10px;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    background: #1e3a8a;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 20px;
    font-weight: bold;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

@media (max-width: 768px) {
    .photo-grid {
        grid-template-columns: repeat(2, 1fr);
        gap: 15px;
    }
}
</style>