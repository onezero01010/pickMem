    <template>
    <div class="step-four">
        <h1 class="page-title">살레네컷</h1>
        <div class="content-container">
            <div class="title-container">
                <h2 class="section-title">장식하기</h2>
                <p class="instruction-text">배경색, 패턴, 스티커로 사진을 꾸며보세요</p>
                        </div>
            
            <div class="main-content">
                <div class="canvas-section">
                    <div class="canvas-wrapper" ref="canvasWrapper">
                    <div ref="pic" :class="`outter-frame-${parseInt(frame.split('x')[0])}-${parseInt(frame.split('x')[1])}`" style="position: absolute; padding: 20px; padding-right: 0px;">
                        <div :class="`row p-0 m-0`" v-for="(row, rowIdx) of rowCnt" :key="rowIdx">
                            <div :class="`pl-0 pr-0 inner-frame inner-frame-${columns}-${rows}`" v-for="(col, colIdx) of colCnt" :key="colIdx">
                                <div :class="`inner-frame-${columns}-${rows}`">
                                    <img :src="images[rowIdx*colCnt.length + col]" :id="`canvas-${rowIdx*colCnt.length + col}`" draggable="false">
                                </div>
                            </div>
                        </div>
                    </div>
                        <div ref="deco" style="position: absolute;">
                        <canvas v-if="frame" :class="`outter-frame outter-frame-${parseInt(frame.split('x')[0])}-${parseInt(frame.split('x')[1])}`" ref="canvas"></canvas>
                    </div>
                </div>
                </div>
                
                <div class="sidebar-section">
                    <div class="message-section">
                        <h3 class="message-title">메시지 추가</h3>
                        <input 
                            type="text" 
                            v-model="messageText" 
                            class="message-input" 
                            placeholder="우린 살레시안"
                        >
                        <button class="apply-btn" @click="addMessage">적용</button>
            </div>
                    <div class="sidebar-content">
                        <div v-for="(value, theme) in bg" :key="theme">
                            <div class="color-grid">
                                <div 
                                    :class="{ 'selected': targetColor == color }" 
                                    class="color-item" 
                                    v-for="(color, idx) of value" 
                                    :key="idx"
                                    @click="selectBg(color)"
                                >
                                    <div class="bg" :style="{'background-color': color}"></div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="save-section">
                        <button class="save-btn" @click="saveImage">저장하기</button>
                        <button class="home-btn" @click="goHome">처음으로</button>
                    </div>
                    </div>
                </div>    
            </div>
    </div>
</template>

<script>
import { fabric } from 'fabric'
import html2canvas from 'html2canvas'

export default {
    name: 'stepFour',
    components: {
    },
    data() {
        return {
            rows: 2,
            columns: 1,
            rowCnt: [],
            colCnt: [],
            images: {},
            targetColor: '#00ff0000',
            targetPattern: null,
            frame: null,
            canvasHeight: null,
            canvasWidth: null,
            canvas: null,
            contentHeight: 500,
            messageText: '',
            fontSize: 24,
            pattern: {
                'basic': [],
            },
            bg: {
                'simple': ['#F2F2F3', '#A6A6A6', '#595959', '#262626', '#0D0D0D'],
                'astro': ['#F25E7A', '#4A2B8C', '#5155A6', '#05F2DB', '#F2E963'],
                'cartoon': ['#636AF2', '#41A0F2', '#A2DCF2', '#04D98B', '#F2E205'],
            }
        }
    },
    created() {
        this.frame = this.$store.getters.getFrame;
    },
    mounted() {
        this.init();
    },
    beforeDestroy() {
        this.saveWork();
    },
    methods: {
        async init() {
            let table = this.$store.getters.getTable;

            this.images = this.$store.getters.getTargets;
            this.rows = table.rows;
            this.columns = table.columns;
            this.rowCnt = Array.from({length: table.rows}, (v, i) => i + 1);
            this.colCnt = Array.from({length: table.columns}, (v, i) => i + 1);
            this.pattern.basic = Array.from({length: 9}, (v, i) => i + 1);

            this.canvasHeight = this.$refs.canvas.clientHeight;
            this.canvasWidth = this.$refs.canvas.clientWidth;
            this.canvas = new fabric.Canvas(this.$refs.canvas);
            this.canvas.setHeight(this.canvasHeight);
            this.canvas.setWidth(this.canvasWidth);

            if (this.$store.getters.getCanvas) {
                await this.loadCanvasToJSON();
                this.targetColor = this.canvas.backgroundColor;
                this.canvas.renderAll();
            }

            this.setMouseEvent();
            this.setWorkMode();
        },
        loadCanvasToJSON() {
            return new Promise((resolve, reject) => {
                this.canvas.loadFromJSON(this.$store.getters.getCanvas, this.canvas.renderAll.bind(this.canvas));
                resolve();
            })
        },
        setMouseEvent() {
            this.canvas.on('selection:created', (e) => {
                if (e.selected.length > 1) this.canvas.discardActiveObject().renderAll();
            }).on('selection:cleared', (e) => {
                this.saveWork();
                this.canvas.discardActiveObject().renderAll();
            })
        },
        saveWork() {
            this.canvas.discardActiveObject().renderAll();

            this.setWorkMode();
            this.$store.commit('setCanvas', JSON.stringify(this.canvas.toObject(['id', 'borderColor', 'cornerColor', 'cornerSize', 'transparentCorners'])));
            this.$store.commit('setFrameImg', this.canvas.toDataURL({ format: 'image/png' }));
        },
        setWorkMode() {
            this.canvas.discardActiveObject().renderAll();
            // 배경 모드: 사진이 위에 보이도록 (사진이 보이고, canvas 배경은 뒤에서 보임)
                this.$refs.pic.style['z-index'] = 2;
                this.$refs.deco.style['z-index'] = 1;
                this.$refs.deco.style['opacity'] = 1;
        },
        selectBg(color) {
            if (this.targetColor == color) {
                this.targetColor = '#FFF';
                this.canvas.backgroundColor = '#FFF';
            } else {
                this.targetColor = color;
                this.targetPattern = null;
                this.canvas.backgroundColor = color;
            }
            
            this.canvas.renderAll();
            this.setWorkMode(); // 배경이 바로 보이도록 z-index 업데이트
            this.saveWork();
        },
        selectPattern(pattern) {
            if (this.targetPattern == pattern) {
                this.canvas.backgroundColor = '#FFF';
                this.targetColor = '#FFF';
                this.targetPattern = null;
                this.setWorkMode(); // 배경이 바로 보이도록 z-index 업데이트
                this.saveWork();
            } else {
                this.targetColor = '#FFF'
                this.targetPattern = pattern;

                this.canvas.setBackgroundColor({ source: this.$refs[this.targetPattern][0].src, repeat: 'repeat' }, () => {
                    this.canvas.renderAll();
                    this.setWorkMode(); // 배경이 바로 보이도록 z-index 업데이트
                    this.saveWork();
                });
            }

            this.canvas.renderAll();
        },
        addMessage() {
            if (!this.messageText.trim()) return;

            // 기존 메시지 텍스트 박스가 있으면 제거
            const existingTexts = this.canvas.getObjects().filter(obj => obj.id && obj.id.includes('message'));
            existingTexts.forEach(text => this.canvas.remove(text));

            // 배경색에 따른 반대색 계산
            const bgColor = this.canvas.backgroundColor;
            const textColor = this.getContrastColor(bgColor);

            // 프레임 아래 부분에 텍스트 추가 (canvas 높이의 90% 위치)
            const textBox = new fabric.Textbox(this.messageText, {
                left: this.canvas.width / 2,
                top: this.canvas.height * 0.95,
                width: this.canvas.width - 40,
                fontSize: this.fontSize,
                textAlign: 'center',
                originX: 'center',
                originY: 'center',
                fill: textColor,
                id: `message_${Date.now()}`
            });

            this.canvas.add(textBox);
            this.canvas.renderAll();
            this.saveWork();
        },
        getContrastColor(bgColor) {
            // 배경색이 없거나 흰색이면 검정색 반환
            if (!bgColor || bgColor === '#FFF' || bgColor === '#FFFFFF' || bgColor === 'white' || bgColor === 'transparent') {
                return '#000000';
            }

            // 패턴 이미지인 경우 (객체로 들어옴)
            if (typeof bgColor === 'object' && bgColor.source) {
                // 패턴은 보통 밝은 배경이므로 검정색 반환
                return '#000000';
            }

            // HEX 색상인 경우
            if (typeof bgColor === 'string' && bgColor.startsWith('#')) {
                // # 제거
                const hex = bgColor.replace('#', '');
                
                // RGB 값 추출
                let r, g, b;
                if (hex.length === 3) {
                    r = parseInt(hex[0] + hex[0], 16);
                    g = parseInt(hex[1] + hex[1], 16);
                    b = parseInt(hex[2] + hex[2], 16);
                } else if (hex.length === 6) {
                    r = parseInt(hex.substring(0, 2), 16);
                    g = parseInt(hex.substring(2, 4), 16);
                    b = parseInt(hex.substring(4, 6), 16);
                } else {
                    return '#000000';
                }

                // 밝기 계산 (상대적 밝기)
                const luminance = (0.299 * r + 0.587 * g + 0.114 * b) / 255;

                // 밝기가 0.5보다 크면 어두운 텍스트, 작으면 밝은 텍스트
                return luminance > 0.5 ? '#000000' : '#FFFFFF';
            }

            // 기본값: 검정색
            return '#000000';
        },
        async saveImage() {
            // canvas와 사진을 합쳐서 이미지로 저장
            this.saveWork();
            
            try {
                // 새로운 고해상도 canvas 생성
                const scale = 2;
                const exportCanvas = document.createElement('canvas');
                exportCanvas.width = this.canvas.width * scale;
                exportCanvas.height = this.canvas.height * scale;
                const ctx = exportCanvas.getContext('2d');
                
                // 배경색/패턴 그리기
                if (this.canvas.backgroundColor) {
                    if (typeof this.canvas.backgroundColor === 'string') {
                        ctx.fillStyle = this.canvas.backgroundColor;
                        ctx.fillRect(0, 0, exportCanvas.width, exportCanvas.height);
                    } else if (this.canvas.backgroundColor && this.canvas.backgroundColor.source) {
                        // 패턴인 경우
                        const patternImg = new Image();
                        patternImg.crossOrigin = 'anonymous';
                        await new Promise((resolve, reject) => {
                            patternImg.onload = () => {
                                const pattern = ctx.createPattern(patternImg, 'repeat');
                                ctx.fillStyle = pattern;
                                ctx.fillRect(0, 0, exportCanvas.width, exportCanvas.height);
                                resolve();
                            };
                            patternImg.onerror = reject;
                            patternImg.src = this.canvas.backgroundColor.source;
                        });
                    } else {
                        ctx.fillStyle = '#FFFFFF';
                        ctx.fillRect(0, 0, exportCanvas.width, exportCanvas.height);
                    }
                } else {
                    ctx.fillStyle = '#FFFFFF';
                    ctx.fillRect(0, 0, exportCanvas.width, exportCanvas.height);
                }
                
                // 사진들 그리기 - DOM에서 직접 가져오기
                const picElement = this.$refs.pic;
                if (picElement) {
                    const innerFrames = picElement.querySelectorAll('.inner-frame');
                    console.log('찾은 inner-frame 개수:', innerFrames.length);
                    
                    for (let i = 0; i < innerFrames.length; i++) {
                        const frame = innerFrames[i];
                        const img = frame.querySelector('img');
                        
                        if (img && img.src) {
                            console.log(`사진 ${i + 1} 로딩 시작:`, img.src);
                            
                            const imgElement = new Image();
                            imgElement.crossOrigin = 'anonymous';
                            
                            await new Promise((resolve, reject) => {
                                const timeout = setTimeout(() => {
                                    console.error('이미지 로드 타임아웃:', img.src);
                                    resolve(); // 타임아웃해도 계속 진행
                                }, 5000);
                                
                                imgElement.onload = () => {
                                    clearTimeout(timeout);
                                    const frameRect = frame.getBoundingClientRect();
                                    const canvasRect = this.$refs.canvas.getBoundingClientRect();
                                    
                                    // canvas 기준으로 상대 위치 계산
                                    const x = (frameRect.left - canvasRect.left) * scale;
                                    const y = (frameRect.top - canvasRect.top) * scale;
                                    const width = frameRect.width * scale;
                                    const height = frameRect.height * scale;
                                    
                                    console.log(`사진 ${i + 1} 그리기:`, { x, y, width, height });
                                    ctx.drawImage(imgElement, x, y, width, height);
                                    resolve();
                                };
                                
                                imgElement.onerror = (error) => {
                                    clearTimeout(timeout);
                                    console.error('이미지 로드 실패:', img.src, error);
                                    resolve(); // 실패해도 계속 진행
                                };
                                
                                imgElement.src = img.src;
                            });
                        } else {
                            console.warn(`사진 ${i + 1}을 찾을 수 없습니다.`);
                        }
                    }
                }
                
                // canvas의 텍스트와 다른 객체들만 그리기 (배경 제외)
                const originalBackground = this.canvas.backgroundColor;
                this.canvas.backgroundColor = 'transparent';
                this.canvas.renderAll();
                
                const canvasDataURL = this.canvas.toDataURL({ 
                    format: 'png',
                    multiplier: scale,
                    backgroundColor: 'transparent'
                });
                
                // 원래 배경 복원
                this.canvas.backgroundColor = originalBackground;
                this.canvas.renderAll();
                
                const canvasImg = new Image();
                await new Promise((resolve, reject) => {
                    canvasImg.onload = () => {
                        ctx.drawImage(canvasImg, 0, 0);
                        resolve();
                    };
                    canvasImg.onerror = reject;
                    canvasImg.src = canvasDataURL;
                });
                
                // blob으로 변환하여 다운로드
                exportCanvas.toBlob((blob) => {
                    if (blob) {
                        const url = URL.createObjectURL(blob);
                        const link = document.createElement('a');
                        link.download = `pickMem_${Date.now()}.png`;
                        link.href = url;
                        document.body.appendChild(link);
                        link.click();
                        document.body.removeChild(link);
                        URL.revokeObjectURL(url);
                    } else {
                        alert('이미지 저장에 실패했습니다.');
                    }
                }, 'image/png');
            } catch (error) {
                console.error('이미지 저장 오류:', error);
                alert('이미지 저장 중 오류가 발생했습니다.');
            }
        },
        goHome() {
            this.$router.push('/');
        },
    },
    computed: {
        getContentHeight() {
            return window.screen.availHeight;
        }
    },
}
</script>

<style lang="scss" scoped>
.step-four {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100vh;
    background: linear-gradient(135deg, #e8d5ff 0%, #d4b3ff 100%);
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

.main-content {
    display: flex;
    gap: 30px;
    align-items: flex-start;
}

.canvas-section {
    flex: 1;
    display: flex;
    margin-top: 40px;
    flex-direction: column;
    align-items: center;
    min-width: 0;
    max-width: 100%;
}

.canvas-wrapper {
    position: relative;
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
}

.help-text {
    text-align: center;
    color: #666;
    font-size: 14px;
    margin-top: 10px;

    p {
        margin: 0;
    }
}

.sidebar-section {
    width: 350px;
    flex: 0 0 auto;
    background: #f9fafb;
    border-radius: 15px;
    padding: 20px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.message-section {
    margin-bottom: 30px;
    padding-bottom: 20px;
    border-bottom: 2px solid #e5e7eb;
}

.message-title {
    font-size: 18px;
    font-weight: 600;
    color: #374151;
    text-align: center;
    margin: 0 0 15px 0;
}

.message-input {
    width: 100%;
    padding: 12px 16px;
    border: 1px solid #d1d5db;
    border-radius: 10px;
    font-size: 16px;
    color: #374151;
    margin-bottom: 15px;
    box-sizing: border-box;
    transition: border-color 0.3s ease;

    &:focus {
        outline: none;
        border-color: #8b5cf6;
    }

    &::placeholder {
        color: #9ca3af;
    }
}

.apply-btn {
    width: 100%;
    padding: 12px 24px;
    background: linear-gradient(135deg, #8b5cf6 0%, #7c3aed 100%);
    color: white;
    border: none;
    border-radius: 10px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 12px rgba(139, 92, 246, 0.3);

    &:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 16px rgba(139, 92, 246, 0.4);
    }

    &:active {
        transform: translateY(0);
    }
}

.sidebar-content {
    max-height: 600px;
    overflow-y: auto;
    padding-right: 5px;
    margin-bottom: 20px;

    &::-webkit-scrollbar {
        width: 6px;
    }

    &::-webkit-scrollbar-track {
        background: #f1f1f1;
        border-radius: 10px;
    }

    &::-webkit-scrollbar-thumb {
        background: #8b5cf6;
        border-radius: 10px;
    }
}

.save-section {
    margin-top: 20px;
    padding-top: 20px;
    border-top: 2px solid #e5e7eb;
    display: flex;
    gap: 10px;
}

.save-btn {
    flex: 1;
    padding: 14px 24px;
    background: linear-gradient(135deg, #10b981 0%, #059669 100%);
    color: white;
    border: none;
    border-radius: 10px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 12px rgba(16, 185, 129, 0.3);

    &:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 16px rgba(16, 185, 129, 0.4);
    }

    &:active {
        transform: translateY(0);
    }
}

.home-btn {
    flex: 1;
    padding: 14px 24px;
    background: linear-gradient(135deg, #6b7280 0%, #4b5563 100%);
    color: white;
    border: none;
    border-radius: 10px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 12px rgba(107, 114, 128, 0.3);

    &:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 16px rgba(107, 114, 128, 0.4);
    }

    &:active {
        transform: translateY(0);
    }
}

.theme-title {
    font-size: 18px;
    font-weight: 600;
    color: #8b5cf6;
    margin-bottom: 15px;
    margin-top: 20px;

    &:first-child {
        margin-top: 0;
    }
}

.color-grid {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 10px;
    margin-bottom: 20px;
}

.color-item {
    aspect-ratio: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 10px;
    cursor: pointer;
    transition: all 0.3s ease;
    padding: 5px;

    &:hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
    }

    &.selected {
        border: 3px solid #8b5cf6;
        box-shadow: 0 0 0 2px rgba(139, 92, 246, 0.3);
    }
}

.bg {
    height: 100%;
    width: 100%;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.pattern-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
    margin-bottom: 30px;
}

.pattern-item {
    aspect-ratio: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 10px;
    cursor: pointer;
    transition: all 0.3s ease;
    padding: 5px;
    background: white;
    border: 2px solid transparent;

    &:hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
        border-color: #8b5cf6;
    }

    &.selected {
        border-color: #8b5cf6;
        box-shadow: 0 0 0 2px rgba(139, 92, 246, 0.3);
    }
}

.pattern {
    width: 100%;
    height: 100%;
    object-fit: cover;
    border-radius: 6px;
}

// Canvas frame styles
img {
    object-position: center center;
    object-fit: cover;
    height: 100%;
    width: 100%;
}

.outter-frame {
    padding-right: 0px;
    padding-bottom: 0px;

    &-1-1 {
        height: 350px;
        width: 400px;
    }
    &-1-2 {
        height: 520px;
        width: 320px;
    }
    &-1-3 {
        height: 525px;
        width: 220px;
    }
    &-1-4 {
        height: 480px;
        width: 180px;
    }
    &-2-1 {
        height: 320px;
        width: 520px;
    }
    &-2-2 {
        height: 440px;
        width: 620px; 
    }
    &-2-3 {
        height: 525px;
        width: 420px;
    }
    &-3-1 {
        height: 220px;
        width: 525px;
    }
    &-3-2 {
        height: 420px;
        width: 525px;
    }
    &-4-1 {
        height: 180px;
        width: 480px;
    }
}

.inner-frame {
    position: relative;
    margin-bottom: 20px;
    margin-right: 20px;
    background: #FFF;

    &-1-1 {
        height: 270px;
        width: 360px;
    }
    &-1-2 {
        height: 210px;
        width: 280px;
    }
    &-1-3 {
        height: 135px;
        width: 180px;
    }
    &-1-4 {
        height: 84px;
        width: 140px;
    }
    &-2-1 {
        height: 280px;
        width: 210px;
    }
    &-2-2 {
        height: 168px;
        width: 280px;
    }
    &-2-3 {
        height: 135px;
        width: 180px;
    }
    &-3-1 {
        height: 180px;
        width: 135px;
    }
    &-3-2 {
        height: 180px;
        width: 135px;
    }
    &-4-1 {
        height: 140px;
        width: 84px;
    }
}

@media (max-width: 1200px) {
    .main-content {
        flex-direction: column;
        align-items: center;
        gap: 20px;
    }

    .canvas-section {
        margin-top: 20px;
        width: 100%;
        max-width: 100%;
        flex: none;
    }

    .sidebar-section {
        width: 100%;
        max-width: 100%;
        margin-right: 0;
        margin-top: 500px;
        flex: none;
    }
}

@media (max-width: 768px) {
    .content-container {
        padding: 20px;
    }

    .main-content {
        gap: 15px;
    }

    .canvas-section {
        margin-top: 10px;
        width: 100%;
        max-width: 100%;
        flex: none;
    }

    .sidebar-section {
        margin-top: 500px;
        padding: 15px;
        width: 100%;
        max-width: 100%;
        flex: none;
    }

    .color-grid {
        grid-template-columns: repeat(4, 1fr);
    }

    .pattern-grid {
        grid-template-columns: repeat(4, 1fr);
    }   
}

/* 세로로 긴 화면 (높이가 넓은 화면보다 긴 경우) */
@media (max-height: 900px) and (orientation: portrait) {
    .main-content {
        flex-direction: column;
        align-items: center;
        gap: 15px;
    }

    .canvas-section {
        margin-top: 10px;
        width: 100%;
        max-width: 100%;
        flex: none;
    }

    .sidebar-section {
        width: 100%;
        max-width: 100%;
        margin-right: 0;
        margin-top: 500px;
        flex: none;
    }
}

/* 모바일 세로 모드 */
@media (max-width: 480px) {
    .page-title {
        font-size: 36px;
        margin-bottom: 20px;
    }

    .content-container {
        padding: 15px;
    }

    .section-title {
        font-size: 24px;
    }

    .main-content {
        gap: 10px;
    }

    .canvas-section {
        margin-top: 0;
        width: 100%;
        max-width: 100%;
        flex: none;
    }

    .sidebar-section {
        padding: 12px;
        width: 100%;
        max-width: 100%;
        flex: none;
        margin-top: 500px;
    }

    .color-grid {
        grid-template-columns: repeat(3, 1fr);
        gap: 8px;
    }

    .message-input {
        font-size: 14px;
        padding: 10px 12px;
    }

    .apply-btn, .save-btn, .home-btn {
        font-size: 14px;
        padding: 12px 20px;
    }
}
</style>