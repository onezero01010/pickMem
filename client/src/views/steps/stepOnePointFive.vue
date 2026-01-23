<template>
    <StepLayout title="인물 선택">
        <div class="character-options-wrapper">
            <div 
                class="character-option" 
                :class="{ 'selected': selectedCharacters.includes('bosco') }" 
                @click="toggleCharacter('bosco')"
            >
                <img class="character-preview" src="@/assets/design/bosco.png" alt="보스코" draggable="false">
                <span class="character-label">보스코</span>
            </div>
            <div class="or-divider">OR</div>
            <div 
                class="character-option" 
                :class="{ 'selected': selectedCharacters.includes('none') }" 
                @click="toggleCharacter('none')"
            >
                <div class="character-preview no-character">
                    <span class="no-character-text">인물 없음</span>
                </div>
                <span class="character-label">인물 없음</span>
            </div>
        </div>
        <div class="selected-info" :class="{ 'complete': selectedCharacters.length === 1 }">
        </div>
    </StepLayout>
</template>

<script>
import StepLayout from '@/components/StepLayout.vue'

export default {
    name: 'StepOnePointFive',
    components: {
        StepLayout
    },
    data() {
        return {
            selectedCharacters: [],
        }
    },
    mounted() {
        // 저장된 인물 선택 정보가 있으면 복원
        const saved = this.$store.getters.getSelectedCharacter;
        if (saved) {
            this.selectedCharacters = [saved];
        }
    },
    watch: {
        selectedCharacters: {
            handler(newVal) {
                if (newVal.length === 1) {
                    // 1개 선택되면 자동으로 다음 단계로 이동
                    this.$nextTick(() => {
                        this.$emit('characters-selected');
                    });
                }
            },
            immediate: false
        }
    },
    methods: {
        toggleCharacter(character) {
            // 하나만 선택 가능하도록 변경
            if (this.selectedCharacters.includes(character)) {
                // 이미 선택된 경우 제거
                this.selectedCharacters = [];
                this.$store.commit('setSelectedCharacter', null);
            } else {
                // 새로운 인물 선택 (기존 선택 해제)
                this.selectedCharacters = [character];
                // store에 저장 (단일 값으로 저장, 'none'은 null로 저장)
                const valueToStore = character === 'none' ? null : character;
                this.$store.commit('setSelectedCharacter', valueToStore);
            }
        },
    }
}
</script>

<style lang="scss" scoped>
.character-options-wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 15px;
}

.character-option {
    flex: 1;
    cursor: pointer;
    border-radius: 12px;
    padding: 12px;
    display: flex;
    flex-direction: column;
    align-items: center;
    transition: all 0.3s ease;
    
    &:hover {
        transform: translateY(-3px);
    }
    
    &.selected {
        background: #e0f2fe;
        box-shadow: 0 0 0 3px #3b82f6;
    }
}

.character-preview {
    width: 100%;
    max-width: 140px;
    height: auto;
    object-fit: contain;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    transition: box-shadow 0.3s ease, transform 0.3s ease;
    
    &:hover {
        box-shadow: 0 8px 20px rgba(0, 0, 0, 0.25);
        transform: translateY(-3px);
    }
    
    &.no-character {
        height: 140px;
        display: flex;
        align-items: center;
        justify-content: center;
        background: #f9fafb;
        border: 2px dashed #d1d5db;
        
        .no-character-text {
            font-size: 14px;
            font-weight: 600;
            color: #6b7280;
        }
    }
}

.character-label {
    font-size: 13px;
    font-weight: 600;
    color: #333;
    margin-top: 10px;
}

.or-divider {
    font-size: 18px;
    font-weight: bold;
    color: #333;
    padding: 0 12px;
}

.selected-info {
    margin-top: 30px;
    text-align: center;
    padding: 15px;
    border-radius: 10px;
    font-weight: 600;
    
    p {
        margin: 0;
        color: #666;
    }
    
    &.complete {
        background: #e8f5e9;
        
        p {
            color: #2e7d32;
        }
    }
}

@media (max-width: 768px) {
    .character-options-wrapper {
        flex-direction: column;
        gap: 20px;
    }
    
    .or-divider {
        padding: 10px 0;
    }
}
</style>
