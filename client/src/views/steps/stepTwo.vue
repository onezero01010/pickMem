<template>
    <StepLayout title="사진 촬영" :show-title="false">
        <div v-if="isLoading" class="loading-container">
                <div class="text-center">
                    <div><i class="mdi mdi-loading mdi-spin" style="font-size: 40px;"></i></div>
                    <small>카메라 설정 중</small>
                </div>
            </div>
            <div v-else-if="!isLoading && canPhoto" class="camera-container">
                <div class="title-container">
                    <h2 class="section-title">사진 촬영</h2>
                </div>
                <div class="phone-frame">
                    <div class="phone-notch"></div>
                    <div class="camera-content">
                        <video v-show="!isPhotoTaken" ref="camera" :class="{'camera-mirror': isMirrored, 'camera-22-frame': is22Frame}" autoplay></video>
                        <canvas v-show="isPhotoTaken" ref="canvas" :class="{'camera-22-frame': is22Frame}"></canvas>
                        <img 
                            v-if="overlayImageSrc && !isPhotoTaken" 
                            :src="overlayImageSrc" 
                            class="character-overlay"
                            alt="Character overlay"
                        />
                        <div v-if="isCountdown && !isPaused" class="countdown-overlay">
                            <div class="countdown-number">{{countdown}}</div>
                            <button class="btn btn-instant" @click="instantCapture">즉시 촬영</button>
                        </div>
                    </div>
                </div>
                <div class="control-buttons">
                    <button class="btn btn-photo" @click="handlePhotoClick">
                        사진촬영
                    </button>
                    <button class="btn btn-mirror" @click="toggleMirror">
                        좌우반전
                    </button>
                    <button class="btn btn-switch" @click="switchCamera">
                        카메라 전환
                    </button>
                    <button class="btn btn-restart" @click="togglePause">
                        {{ isPaused ? '다시시작' : '일시정지' }}
                    </button>
                </div>
                <div class="photo-counter">
                    사진 {{getImageLen}}/8
                </div>
                <div v-if="getImageLen > 0" class="photo-gallery">
                    <div 
                        v-for="(img, idx) in Object.values(images)" 
                        :key="idx" 
                        class="photo-thumbnail"
                    >
                        <img :src="img" alt="촬영된 사진">
                    </div>
                </div>
            </div>
            <div v-else class="camera-error">
                <div class="text-center">
                    <i class="mdi mdi-camera-off" style="font-size: 60px; color: #999;"></i>
                    <p style="font-size: 18px; color: #666; margin-top: 20px;">카메라에 접근할 수 없습니다.</p>
                    <p style="font-size: 14px; color: #999;">브라우저에서 카메라 권한을 허용해주세요.</p>
                </div>
            </div>
        <modal v-if="isOpen" @on-close="isOpen=false" @on-submit="isOpen=false;initImage()" :title="'사진 삭제'" :msg="'찍은 사진들을 모두 초기화 하시겠어요?'"></modal>
    </StepLayout>
</template>

<script>
import Modal from '../../components/modal.vue'
import StepLayout from '@/components/StepLayout.vue'
import jesusOverlay1 from '@/assets/design/2-2_jesus_1.png'
import jesusOverlay2 from '@/assets/design/2-2_jesus_2.png'
import boscoOverlay1 from '@/assets/design/2-2_bosco_1.png'
import boscoOverlay2 from '@/assets/design/2-2_bosco_2.png'
import cameraSound from '@/assets/sound/camera.mp3'

export default {
    name: 'StepTwo',
    components: {
        Modal,
        StepLayout
    },
    data() {
        return {
            images: {},
            rows: 2,
            columns: 1,
            canPhoto: false,
            isOpen: false,
            isCameraOpen: false,
            isPhotoTaken: false,
            isShotPhoto: false,
            isLoading: false,
            isTarget: null,
            isMirrored: true,
            facingMode: 'user',
            stream: null,
            isCountdown: false,
            countdown: 3,
            countdownInterval: null,
            isPaused: false,
            currentPhotoIndex: 0, // 현재 촬영 중인 사진 번호 (1부터 시작)
            overlayImageSrc: null,
            jesusOverlay1,
            jesusOverlay2,
            boscoOverlay1,
            boscoOverlay2,
        }
    },
    created() {
        
    },
    mounted() {
        let table = this.$store.getters.getFrame
        if (table) {
            this.rows = table.split('x')[0];
            this.columns = table.split('x')[1];
        }
        this.images = this.$store.getters.getImages;
        
        // 현재 촬영할 사진 번호 설정 (이미 촬영된 사진 개수 + 1)
        this.currentPhotoIndex = this.getImageLen + 1;
        this.updateOverlayImage();

        if (this.getImageLen >= 8) this.$store.commit('setNext', true);
        else if (this.getImageLen >= 6) this.$store.commit('setNext', true);
        else this.$store.commit('setNext', false);

        this.createCameraElement();
    },
    beforeDestroy() {
        // this.showingImage();
        // this.stopCameraStream();
    },
    methods: {
        createCameraElement() {
            if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                console.error('카메라 API를 사용할 수 없습니다.');
                this.isLoading = false;
                this.canPhoto = false;
                return;
            }

            this.isLoading = true;
            this.canPhoto = false;
            
            const constraints = {
                audio: false,
                video: {
                    facingMode: this.facingMode,
                    width: { ideal: 1920, min: 1280 },
                    height: { ideal: 1080, min: 720 },
                },
            };

            navigator.mediaDevices.getUserMedia(constraints)
                .then(stream => {
                    this.stream = stream;
                    this.canPhoto = true;
                    this.isLoading = false;
                    
                    this.$nextTick(() => {
                        if (this.$refs.camera) {
                            this.$refs.camera.srcObject = stream;
                        }
                    });
                })
                .catch(error => {
                    console.error('카메라 접근 오류:', error);
                    this.isLoading = false;
                    this.canPhoto = false;
            });
        },
        
        switchCamera() {
            if (this.stream) {
                this.stream.getTracks().forEach(track => track.stop());
            }
            this.facingMode = this.facingMode === 'user' ? 'environment' : 'user';
            this.createCameraElement();
        },
        
        toggleMirror() {
            this.isMirrored = !this.isMirrored;
        },
        
        handlePhotoClick() {
            if (this.isPhotoTaken) {
                this.saveImage();
                this.isPhotoTaken = false;
            } else {
                this.startCountdown();
            }
        },
        
        startCountdown() {
            if (this.isCountdown && !this.isPaused) return;
            if (this.isPaused) {
                this.isPaused = false;
            }
            
            this.isCountdown = true;
            this.countdown = 3;
            
            this.countdownInterval = setInterval(() => {
                if (this.isPaused) {
                    clearInterval(this.countdownInterval);
                    this.countdownInterval = null;
                    return;
                }
                
                this.countdown--;
                
                if (this.countdown <= 0) {
                    this.stopCountdown();
                    this.takePhoto();
                }
            }, 1000);
        },
        
        stopCountdown() {
            if (this.countdownInterval) {
                clearInterval(this.countdownInterval);
                this.countdownInterval = null;
            }
            this.isCountdown = false;
            this.countdown = 3;
        },
        
        instantCapture() {
            this.stopCountdown();
            this.takePhoto();
        },
        
        togglePause() {
            if (this.isPaused) {
                // 다시시작: 카운트다운 재개
                this.isPaused = false;
                if (this.isCountdown) {
                    // 카운트다운이 이미 시작되어 있었다면 계속 진행
                    // 카운트다운이 없었다면 새로 시작
                    if (!this.countdownInterval) {
                        this.startCountdown();
                    }
                }
            } else {
                // 일시정지: 카운트다운 중지
                this.isPaused = true;
                if (this.countdownInterval) {
                    clearInterval(this.countdownInterval);
                    this.countdownInterval = null;
                }
            }
        },

        stopCameraStream() {
            if (this.stream) {
                this.stream.getTracks().forEach(track => {
                    track.stop();
                });
            }
        },

        async takePhoto() {
            if (!this.isPhotoTaken) {
                this.isPhotoTaken = true;
            } else return;

            // 사진 촬영 소리 재생
            const audio = new Audio(cameraSound);
            audio.play().catch(error => {
                console.error('사운드 재생 실패:', error);
            });

            const context = this.$refs.canvas.getContext('2d');
            const canvas = this.$refs.canvas;
            const video = this.$refs.camera;
            
            // 2-2 프레임인 경우 230:272 비율로 crop
            if (this.is22Frame) {
                const videoAspect = video.videoWidth / video.videoHeight; // 16:9 ≈ 1.778
                const targetAspect = 272 / 230; // 약 1.183
                
                // video가 더 넓으므로 양옆을 잘라야 함
                const cropWidth = video.videoHeight * targetAspect;
                const cropX = (video.videoWidth - cropWidth) / 2;
                
                canvas.width = cropWidth;
                canvas.height = video.videoHeight;
                
                // 촬영된 사진도 반전하여 저장 (양옆 crop)
                context.save();
                context.scale(-1, 1);
                context.drawImage(video, cropX, 0, cropWidth, video.videoHeight, -canvas.width, 0, canvas.width, canvas.height);
                context.restore();
            } else {
                canvas.width = video.videoWidth;
                canvas.height = video.videoHeight;
                
                // 촬영된 사진도 반전하여 저장
                context.save();
                context.scale(-1, 1);
                context.drawImage(video, -canvas.width, 0, canvas.width, canvas.height);
                context.restore();
            }
            
            // 오버레이 이미지가 있으면 그 위에 그리기
            if (this.overlayImageSrc) {
                await new Promise((resolve, reject) => {
                    const overlayImg = new Image();
                    overlayImg.crossOrigin = 'anonymous';
                    
                    overlayImg.onload = () => {
                        try {
                            // canvas 전체에 오버레이 이미지 그리기
                            context.drawImage(overlayImg, 0, 0, canvas.width, canvas.height);
                        } catch (error) {
                            console.error('오버레이 이미지 그리기 오류:', error);
                        }
                        resolve();
                    };
                    
                    overlayImg.onerror = (error) => {
                        console.error('오버레이 이미지 로드 실패:', error);
                        resolve(); // 실패해도 계속 진행
                    };
                    
                    overlayImg.src = this.overlayImageSrc;
                });
            }
            
            // 사진 저장 후 연속 촬영 처리
            this.$nextTick(() => {
                this.saveImage();
                this.isPhotoTaken = false;
                
                // 이미지 저장 후 길이 확인
                this.$nextTick(() => {
                    // 다음 사진 번호 업데이트
                    this.currentPhotoIndex = this.getImageLen + 1;
                    this.updateOverlayImage();
                    
                    // 8장 미만이면 자동으로 다음 카운트다운 시작
                    if (this.getImageLen < 8) {
                        setTimeout(() => {
                            this.startCountdown();
                        }, 500); // 0.5초 후 다음 카운트다운 시작
                    } else {
                        // 8장이 모두 촬영되면 다음 페이지로 이동
                        setTimeout(() => {
                            this.$emit('photos-complete');
                        }, 1000); // 1초 후 다음 페이지로 이동
                    }
                });
            });
        },

        saveImage() {
            const image = this.$refs.canvas.toDataURL("image/jpeg").replace("image/jpeg", "image/octet-stream");
            let id = (new Date).getTime();

            this.$set(this.images, id, image);
        },

        showingImage() {
            this.$store.commit('setImages', this.images);
        },
        
        initImage() {
            this.$store.commit('setNext', false);
            this.$store.commit('setImages', {});
            this.$store.commit('setTargets', {});
            this.$store.commit('setImgCanvas', {});
            this.$store.commit('setTmpTargets', {});
            this.$store.commit('setUpdateQueue', []);
            
            this.images = {};
        },

        removeImg(target) {
            this.images[target] = null;
            this.$delete(this.images, target);
            this.$store.commit('setImages', this.images);
        },
        
        updateOverlayImage() {
            // 2-2 프레임이고 인물이 선택된 경우에만 오버레이 표시
            if (!this.is22Frame) {
                this.overlayImageSrc = null;
                return;
            }
            
            const selectedCharacter = this.$store.getters.getSelectedCharacter;
            // selectedCharacter가 null이거나 없으면 오버레이 표시 안 함
            if (!selectedCharacter || selectedCharacter === 'none') {
                this.overlayImageSrc = null;
                return;
            }
            
            // 1, 5번째 사진: _1.png
            // 3, 7번째 사진: _2.png
            if (this.currentPhotoIndex === 1 || this.currentPhotoIndex === 5) {
                this.overlayImageSrc = selectedCharacter === 'jesus' ? this.jesusOverlay1 : this.boscoOverlay1;
            } else if (this.currentPhotoIndex === 3 || this.currentPhotoIndex === 7) {
                this.overlayImageSrc = selectedCharacter === 'jesus' ? this.jesusOverlay2 : this.boscoOverlay2;
            } else {
                this.overlayImageSrc = null;
            }
        },

    },
    computed: {
        getImageLen() {
            return Object.keys(this.images).length;
        },
        is22Frame() {
            return parseInt(this.rows) === 2 && parseInt(this.columns) === 2;
        },
    },
    watch: {
        images: {
            deep: true,
            handler() {
                if (this.getImageLen >= 8) this.$store.commit('setNext', true);
                else if (this.getImageLen >= 6) this.$store.commit('setNext', true);
                else this.$store.commit('setNext', false);
                this.showingImage();
            }
        }
    },
    beforeDestroy() {
        this.stopCameraStream();
        this.stopCountdown();
    }
}
</script>

<style lang="scss" scoped>
.loading-container {
    text-align: center;
    padding: 100px 20px;
    color: #666;
}

.phone-frame {
    position: relative;
    width: 100%;
    max-width: 1000px;
    margin: 0 auto 30px;
    background: #000;
    border-radius: 25px;
    padding: 10px;
    box-shadow: 0 5px 20px rgba(0, 0, 0, 0.3);
}

.phone-notch {
    position: absolute;
    top: 0;
    left: 50%;
    transform: translateX(-50%);
    width: 480px;
    height: 25px;
    background: #000;
    border-radius: 0 0 15px 15px;
    z-index: 10;
}

.camera-content {
    position: relative;
    width: 100%;
    border-radius: 15px;
    overflow: hidden;
    background: #000;
}

.camera-content video,
.camera-content canvas {
    width: 100%;
    height: auto;
    display: block;
    object-fit: contain;
    border-radius: 15px;
    min-height: 400px;
    
    // 2-2 프레임일 때 230:272 비율로 양옆 crop
    &.camera-22-frame {
        object-fit: cover;
        aspect-ratio: 272 / 230; // 가로:세로 비율
        width: 100%;
        height: 100%;
    }
}

.character-overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    pointer-events: none;
    z-index: 10;
    border-radius: 15px;
}

.camera-mirror {
    transform: scaleX(-1);
}

.countdown-overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: transparent;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    border-radius: 15px;
    z-index: 20;
}

.countdown-number {
    font-size: 120px;
    font-weight: bold;
    color: white;
    text-shadow: 0 0 20px rgba(255, 255, 255, 0.5);
    margin-bottom: 30px;
    animation: pulse 0.5s ease-in-out;
}

@keyframes pulse {
    0%, 100% {
        transform: scale(1);
    }
    50% {
        transform: scale(1.2);
    }
}

.btn-instant {
    background: #1e3a8a;
    color: white;
    padding: 15px 30px;
    font-size: 18px;
    border-radius: 30px;
    border: none;
    cursor: pointer;
    transition: all 0.3s ease;
        box-shadow: 0 4px 15px rgba(30, 58, 138, 0.4);
    
    &:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 20px rgba(30, 58, 138, 0.6);
    }
}

.control-buttons {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-bottom: 20px;
    flex-wrap: wrap;
}

.btn {
    padding: 12px 24px;
    border: none;
    border-radius: 25px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    
    &:hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
    }
}

.btn-photo {
    background: #1e3a8a;
    color: white;
}

.btn-mirror,
.btn-switch,
.btn-restart {
    background: #3b82f6;
    color: white;
}

.photo-counter {
    text-align: center;
    font-size: 18px;
    font-weight: 600;
    color: #666;
    margin-bottom: 20px;
}

.photo-gallery {
    display: flex;
    justify-content: center;
    gap: 15px;
    flex-wrap: wrap;
    margin-top: 30px;
    padding: 20px 0;
}

.photo-thumbnail {
    width: 120px;
    height: 120px;
    border-radius: 10px;
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
    border: 2px solid #e0e0e0;
    cursor: pointer;
    transition: all 0.3s ease;
    
    &:hover {
        transform: translateY(-3px);
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
        border-color: #3b82f6;
    }
    
    img {
        width: 100%;
        height: 100%;
        object-fit: cover;
    }
}

.camera-error {
    padding: 60px 20px;
    text-align: center;
}

@media (max-width: 768px) {
    .control-buttons {
        flex-direction: column;
        
        .btn {
            width: 100%;
        }
    }
    
}
</style>