<template>
    <div class="step-two">
        <h1 class="page-title">살레네컷</h1>
        <div class="content-container">
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
                        <video v-show="!isPhotoTaken" ref="camera" :class="{'camera-mirror': isMirrored}" autoplay></video>
                        <canvas v-show="isPhotoTaken" ref="canvas"></canvas>
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
        </div>
        <modal v-if="isOpen" @on-close="isOpen=false" @on-submit="isOpen=false;initImage()" :title="'사진 삭제'" :msg="'찍은 사진들을 모두 초기화 하시겠어요?'"></modal>
    </div>
</template>

<script>
import Modal from '../../components/modal.vue'

export default {
    name: 'StepOne',
    components: {
        Modal
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
            countdown: 5,
            countdownInterval: null,
            isPaused: false,
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
                    width: { ideal: 1800, min: 1200 },
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
            this.countdown = 5;
            
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
            this.countdown = 5;
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

        takePhoto() {
            if (!this.isPhotoTaken) {
                this.isPhotoTaken = true;
            } else return;

            const context = this.$refs.canvas.getContext('2d');
            const canvas = this.$refs.canvas;
            const video = this.$refs.camera;
            
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            
            // 촬영된 사진도 반전하여 저장
            context.save();
            context.scale(-1, 1);
            context.drawImage(video, -canvas.width, 0, canvas.width, canvas.height);
            context.restore();
            
            // 사진 저장 후 연속 촬영 처리
            this.$nextTick(() => {
                this.saveImage();
                this.isPhotoTaken = false;
                
                // 이미지 저장 후 길이 확인
                this.$nextTick(() => {
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

    },
    computed: {
        getImageLen() {
            return Object.keys(this.images).length;
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
.step-two {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    min-height: 100vh;
    background: linear-gradient(135deg, #e8d5ff 0%, #d4b3ff 100%);
    padding: 40px 20px;
    overflow-y: auto;
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

.title-container {
    text-align: center;
    margin-bottom: 30px;
}

.main-title {
    font-size: 48px;
    font-weight: bold;
    background: linear-gradient(135deg, #ff6b9d 0%, #8b5cf6 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    margin: 0;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
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

.loading-container {
    text-align: center;
    padding: 100px 20px;
    color: #666;
}

.title-container {
    text-align: center;
    margin-bottom: 30px;
}

.section-title {
    font-size: 28px;
    font-weight: 600;
    color: #8b5cf6;
    margin: 0;
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
    background: #ff6b9d;
    color: white;
    padding: 15px 30px;
    font-size: 18px;
    border-radius: 30px;
    border: none;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 15px rgba(255, 107, 157, 0.4);
    
    &:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 20px rgba(255, 107, 157, 0.6);
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
    background: #ff6b9d;
    color: white;
}

.btn-mirror,
.btn-switch,
.btn-restart {
    background: #8b5cf6;
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
        border-color: #8b5cf6;
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
    .page-title {
        font-size: 36px;
    }
    
    .main-title {
        font-size: 36px;
    }
    
    .content-container {
        padding: 20px;
    }
    
    .control-buttons {
        flex-direction: column;
        
        .btn {
            width: 100%;
        }
    }
    
}
</style>