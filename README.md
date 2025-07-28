<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>반려동물 용품 POP 자동 생성기</title>
    
    <!-- CDN 라이브러리 (GitHub Pages에서 안정적) -->
    <script src="https://cdn.jsdelivr.net/npm/qrious@4.0.2/dist/qrious.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>
    
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Malgun Gothic', Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 10px;
        }

        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            overflow: hidden;
        }

        .header {
            background: linear-gradient(45deg, #ff6b6b, #ffd93d);
            color: white;
            text-align: center;
            padding: 20px;
        }

        .header h1 {
            font-size: 1.8rem;
            margin-bottom: 8px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }

        .header p {
            font-size: 1rem;
            opacity: 0.9;
        }

        .main-content {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            padding: 20px;
        }

        .form-section {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 15px;
            border: 2px solid #e9ecef;
            height: fit-content;
        }

        .form-group {
            margin-bottom: 15px;
        }

        .form-group label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
            color: #495057;
            font-size: 0.9rem;
        }

        .form-group input, .form-group textarea, .form-group select {
            width: 100%;
            padding: 10px;
            border: 2px solid #dee2e6;
            border-radius: 8px;
            font-size: 0.9rem;
            transition: border-color 0.3s ease;
        }

        .form-group input:focus, .form-group textarea:focus, .form-group select:focus {
            border-color: #667eea;
            outline: none;
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
        }

        .size-options {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(80px, 1fr));
            gap: 6px;
            margin-bottom: 15px;
        }

        .size-option {
            padding: 10px 5px;
            border: 2px solid #dee2e6;
            border-radius: 8px;
            text-align: center;
            cursor: pointer;
            transition: all 0.3s ease;
            background: white;
            font-size: 0.8rem;
        }

        .size-option:hover {
            border-color: #667eea;
            transform: translateY(-1px);
        }

        .size-option.active {
            border-color: #667eea;
            background: #667eea;
            color: white;
        }

        .custom-size {
            display: none;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 10px;
        }

        .custom-size.show {
            display: grid;
        }

        .template-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(70px, 1fr));
            gap: 6px;
            margin-bottom: 15px;
        }

        .template-option {
            aspect-ratio: 1;
            border: 2px solid #dee2e6;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
            font-size: 0.7rem;
        }

        .template-option:hover {
            transform: scale(1.05);
            border-color: #667eea;
        }

        .template-option.active {
            border-color: #667eea;
            box-shadow: 0 0 10px rgba(102, 126, 234, 0.3);
        }

        .keywords-input {
            display: grid;
            grid-template-columns: 1fr 1fr 1fr;
            gap: 5px;
        }

        .preview-section {
            position: relative;
        }

        .preview-container {
            border: 3px solid #dee2e6;
            border-radius: 15px;
            padding: 15px;
            background: #f8f9fa;
            min-height: 400px;
            display: flex;
            align-items: center;
            justify-content: center;
            position: relative;
        }

        .pop-canvas {
            border-radius: 10px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            max-width: 100%;
            max-height: 100%;
            object-fit: contain;
        }

        .download-section {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 15px;
        }

        .download-btn {
            padding: 12px 15px;
            border: none;
            border-radius: 8px;
            font-size: 0.9rem;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s ease;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .btn-pdf {
            background: linear-gradient(45deg, #ff6b6b, #ee5a52);
            color: white;
        }

        .btn-png {
            background: linear-gradient(45deg, #4ecdc4, #44a08d);
            color: white;
        }

        .download-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }

        .download-btn:disabled {
            opacity: 0.6;
            cursor: not-allowed;
            transform: none;
        }

        .size-info {
            text-align: center;
            margin-top: 8px;
            font-size: 0.8rem;
            color: #6c757d;
        }

        .loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #667eea;
            font-size: 1.1rem;
        }

        /* 템플릿 미리보기 스타일 */
        .template-basic { background: linear-gradient(135deg, #667eea, #764ba2); }
        .template-cute { background: linear-gradient(135deg, #ffeaa7, #fab1a0); }
        .template-premium { background: linear-gradient(135deg, #2d3436, #636e72); }
        .template-nature { background: linear-gradient(135deg, #00b894, #00cec9); }
        .template-playful { background: linear-gradient(135deg, #fd79a8, #fdcb6e); }
        .template-simple { background: #ffffff; border: 2px solid #dee2e6; color: #333; }
        .template-minimal { background: #f8f9fa; border: 2px solid #e9ecef; color: #495057; }
        .template-clean { background: #ffffff; border: 2px solid #667eea; color: #667eea; }

        /* 섹션 제목 */
        .section-title {
            font-size: 1.1rem;
            margin-bottom: 15px;
            color: #495057;
            border-bottom: 2px solid #667eea;
            padding-bottom: 5px;
        }

                        @media (max-width: 1024px) {
            .main-content {
                grid-template-columns: 1fr;
                gap: 15px;
                padding: 15px;
            }
            
            .header h1 {
                font-size: 1.5rem;
            }
            
            .template-grid {
                grid-template-columns: repeat(4, 1fr);
            }
            
            .size-options {
                grid-template-columns: repeat(3, 1fr);
            }
            
            .keywords-input {
                grid-template-columns: 1fr;
                gap: 8px;
            }
        }

        @media (max-width: 768px) {
            body {
                padding: 5px;
            }
            
            .header {
                padding: 15px;
            }
            
            .form-section, .preview-section {
                padding: 15px;
            }
            
            .template-grid {
                grid-template-columns: repeat(3, 1fr);
            }
            
            .size-options {
                grid-template-columns: repeat(2, 1fr);
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🐾 반려동물 용품 POP 생성기</h1>
            <p>간편하게 전문적인 진열 POP를 만들어보세요!</p>
        </div>

        <div class="main-content">
            <div class="form-section">
                <h3 class="section-title">📝 제품 정보 입력</h3>
                
                <div class="form-group">
                    <label for="productName">제품명</label>
                    <input type="text" id="productName" placeholder="예: 프리미엄 강아지 사료">
                </div>

                <div class="form-group">
                    <label for="productPrice">가격</label>
                    <input type="text" id="productPrice" placeholder="예: 29,900원">
                </div>

                <div class="form-group">
                    <label for="productDescription">한 줄 설명</label>
                    <textarea id="productDescription" rows="2" placeholder="예: 건강한 성장을 위한 영양가득 사료"></textarea>
                </div>

                <div class="form-group">
                    <label>특징 키워드 (최대 3개)</label>
                    <div class="keywords-input">
                        <input type="text" id="keyword1" placeholder="저알러지">
                        <input type="text" id="keyword2" placeholder="노령견">
                        <input type="text" id="keyword3" placeholder="소화잘됨">
                    </div>
                </div>

                <div class="form-group">
                    <label for="qrUrl">QR 코드 URL (선택사항)</label>
                    <input type="url" id="qrUrl" placeholder="https://example.com">
                </div>

                <h3 class="section-title" style="margin-top: 20px;">📐 POP 사이즈</h3>
                
                <div class="size-options">
                    <div class="size-option active" data-size="a6">
                        <strong>A6</strong><br>
                        <small>105×148mm</small>
                    </div>
                    <div class="size-option" data-size="a5">
                        <strong>A5</strong><br>
                        <small>148×210mm</small>
                    </div>
                    <div class="size-option" data-size="a4">
                        <strong>A4</strong><br>
                        <small>210×297mm</small>
                    </div>
                    <div class="size-option" data-size="landscape_a6">
                        <strong>A6 가로</strong><br>
                        <small>148×105mm</small>
                    </div>
                    <div class="size-option" data-size="landscape_a5">
                        <strong>A5 가로</strong><br>
                        <small>210×148mm</small>
                    </div>
                    <div class="size-option" data-size="landscape_a4">
                        <strong>A4 가로</strong><br>
                        <small>297×210mm</small>
                    </div>
                    <div class="size-option" data-size="square_small">
                        <strong>정사각형 S</strong><br>
                        <small>100×100mm</small>
                    </div>
                    <div class="size-option" data-size="square_large">
                        <strong>정사각형 L</strong><br>
                        <small>150×150mm</small>
                    </div>
                    <div class="size-option" data-size="custom">
                        <strong>사용자정의</strong><br>
                        <small>직접입력</small>
                    </div>
                </div>

                <div class="custom-size" id="customSizeInputs">
                    <div>
                        <label>폭 (mm)</label>
                        <input type="number" id="customWidth" placeholder="105" min="50" max="500">
                    </div>
                    <div>
                        <label>높이 (mm)</label>
                        <input type="number" id="customHeight" placeholder="148" min="50" max="500">
                    </div>
                </div>

                <h3 class="section-title" style="margin-top: 20px;">🎨 디자인 템플릿</h3>
                
                <div class="template-grid">
                    <div class="template-option template-basic active" data-template="basic">
                        <div style="color: white; padding: 3px; font-size: 0.6rem; text-align: center; height: 100%; display: flex; align-items: center; justify-content: center;">기본형</div>
                    </div>
                    <div class="template-option template-cute" data-template="cute">
                        <div style="color: white; padding: 3px; font-size: 0.6rem; text-align: center; height: 100%; display: flex; align-items: center; justify-content: center;">귀여운형</div>
                    </div>
                    <div class="template-option template-premium" data-template="premium">
                        <div style="color: white; padding: 3px; font-size: 0.6rem; text-align: center; height: 100%; display: flex; align-items: center; justify-content: center;">고급형</div>
                    </div>
                    <div class="template-option template-nature" data-template="nature">
                        <div style="color: white; padding: 3px; font-size: 0.6rem; text-align: center; height: 100%; display: flex; align-items: center; justify-content: center;">자연형</div>
                    </div>
                    <div class="template-option template-playful" data-template="playful">
                        <div style="color: white; padding: 3px; font-size: 0.6rem; text-align: center; height: 100%; display: flex; align-items: center; justify-content: center;">활발형</div>
                    </div>
                    <div class="template-option template-simple" data-template="simple">
                        <div style="color: #333; padding: 3px; font-size: 0.6rem; text-align: center; height: 100%; display: flex; align-items: center; justify-content: center;">심플</div>
                    </div>
                    <div class="template-option template-minimal" data-template="minimal">
                        <div style="color: #495057; padding: 3px; font-size: 0.6rem; text-align: center; height: 100%; display: flex; align-items: center; justify-content: center;">미니멀</div>
                    </div>
                    <div class="template-option template-clean" data-template="clean">
                        <div style="color: #667eea; padding: 3px; font-size: 0.6rem; text-align: center; height: 100%; display: flex; align-items: center; justify-content: center;">클린</div>
                    </div>
                </div>
            </div>

            <div class="preview-section">
                <h3 class="section-title">👀 미리보기</h3>
                <div class="preview-container">
                    <div class="loading" id="loadingText">🎨 POP를 생성하는 중...</div>
                    <canvas id="popCanvas" class="pop-canvas" style="display: none;"></canvas>
                </div>
                <div class="size-info" id="sizeInfo">A6 (105×148mm) - 300DPI</div>
                
                <div class="download-section">
                    <button class="download-btn btn-pdf" onclick="downloadPDF()">
                        📄 PDF 다운로드
                    </button>
                    <button class="download-btn btn-png" onclick="downloadPNG()">
                        🖼️ PNG 다운로드
                    </button>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 템플릿 설정
        const templates = {
            basic: {
                background: 'linear-gradient(135deg, #667eea, #764ba2)',
                titleColor: '#ffffff',
                priceColor: '#ffd93d',
                descColor: '#ffffff',
                keywordBg: 'rgba(255,255,255,0.2)',
                keywordColor: '#ffffff',
                borderColor: '#ffffff',
                isGradient: true
            },
            cute: {
                background: 'linear-gradient(135deg, #ffeaa7, #fab1a0)',
                titleColor: '#2d3436',
                priceColor: '#e84393',
                descColor: '#2d3436',
                keywordBg: 'rgba(255,255,255,0.8)',
                keywordColor: '#2d3436',
                borderColor: '#ffffff',
                isGradient: true
            },
            premium: {
                background: 'linear-gradient(135deg, #2d3436, #636e72)',
                titleColor: '#ffffff',
                priceColor: '#fdcb6e',
                descColor: '#ddd',
                keywordBg: 'rgba(253,203,110,0.2)',
                keywordColor: '#fdcb6e',
                borderColor: '#fdcb6e',
                isGradient: true
            },
            nature: {
                background: 'linear-gradient(135deg, #00b894, #00cec9)',
                titleColor: '#ffffff',
                priceColor: '#fdcb6e',
                descColor: '#ffffff',
                keywordBg: 'rgba(255,255,255,0.2)',
                keywordColor: '#ffffff',
                borderColor: '#ffffff',
                isGradient: true
            },
            playful: {
                background: 'linear-gradient(135deg, #fd79a8, #fdcb6e)',
                titleColor: '#ffffff',
                priceColor: '#ffffff',
                descColor: '#2d3436',
                keywordBg: 'rgba(255,255,255,0.3)',
                keywordColor: '#2d3436',
                borderColor: '#ffffff',
                isGradient: true
            },
            simple: {
                background: '#ffffff',
                titleColor: '#2d3436',
                priceColor: '#e74c3c',
                descColor: '#495057',
                keywordBg: '#f8f9fa',
                keywordColor: '#495057',
                borderColor: '#dee2e6',
                isGradient: false
            },
            minimal: {
                background: '#f8f9fa',
                titleColor: '#2d3436',
                priceColor: '#667eea',
                descColor: '#6c757d',
                keywordBg: '#ffffff',
                keywordColor: '#495057',
                borderColor: '#e9ecef',
                isGradient: false
            },
            clean: {
                background: '#ffffff',
                titleColor: '#667eea',
                priceColor: '#28a745',
                descColor: '#495057',
                keywordBg: 'rgba(102,126,234,0.1)',
                keywordColor: '#667eea',
                borderColor: '#667eea',
                isGradient: false
            }
        };

        // 사이즈 설정 (mm to px, 300DPI 기준)
        const sizes = {
            a6: { width: 105, height: 148 },
            a5: { width: 148, height: 210 },
            a4: { width: 210, height: 297 },
            landscape_a6: { width: 148, height: 105 },
            landscape_a5: { width: 210, height: 148 },
            landscape_a4: { width: 297, height: 210 },
            square_small: { width: 100, height: 100 },
            square_large: { width: 150, height: 150 }
        };

        // 현재 설정
        let currentTemplate = 'basic';
        let currentSize = 'a6';
        let customSize = { width: 105, height: 148 };

        // 키워드 아이콘 매핑
        const keywordIcons = {
            '강아지': '🐶', '개': '🐶', '도그': '🐶', 'dog': '🐶',
            '고양이': '🐱', '캣': '🐱', '냥이': '🐱', 'cat': '🐱',
            '저알러지': '🌿', '알러지': '🌿', '무알러지': '🌿', '알레르기': '🌿',
            '노령견': '👴', '시니어': '👴', '성견': '👴', '나이': '👴',
            '입냄새': '💨', '구취': '💨', '냄새': '💨', '입구냄새': '💨',
            '영양': '💪', '건강': '💪', '비타민': '💪', '단백질': '💪',
            '소화': '❤️', '위장': '❤️', '장건강': '❤️', '소화잘됨': '❤️',
            '치아': '🦷', '덴탈': '🦷', '양치': '🦷', '구강': '🦷',
            '털': '✨', '모질': '✨', '윤기': '✨', '털빠짐': '✨',
            '관절': '🦴', '뼈': '🦴', '칼슘': '🦴', '관절건강': '🦴',
            '면역': '🛡️', '면역력': '🛡️', '항산화': '🛡️', '면역강화': '🛡️',
            '체중': '⚖️', '다이어트': '⚖️', '살빼기': '⚖️', '체중관리': '⚖️',
            '프리미엄': '👑', '고급': '👑', '특급': '👑', '최고급': '👑',
            '유기농': '🌱', '천연': '🌱', '자연': '🌱', '오가닉': '🌱'
        };

        // 라이브러리 로딩 확인
        let librariesLoaded = false;

        function checkLibraries() {
            if (typeof QRious !== 'undefined' && typeof html2canvas !== 'undefined' && typeof jsPDF !== 'undefined') {
                librariesLoaded = true;
                initializeApp();
            } else {
                setTimeout(checkLibraries, 100);
            }
        }

        // 초기화
        function initializeApp() {
            initializeEventListeners();
            setTimeout(() => {
                updatePreview();
                document.getElementById('loadingText').style.display = 'none';
                document.getElementById('popCanvas').style.display = 'block';
            }, 500);
        }

        document.addEventListener('DOMContentLoaded', function() {
            checkLibraries();
        });

        function initializeEventListeners() {
            // 폼 입력 이벤트
            ['productName', 'productPrice', 'productDescription', 'keyword1', 'keyword2', 'keyword3', 'qrUrl'].forEach(id => {
                const element = document.getElementById(id);
                if (element) {
                    element.addEventListener('input', debounce(updatePreview, 300));
                }
            });

            // 사이즈 선택 이벤트
            document.querySelectorAll('.size-option').forEach(option => {
                option.addEventListener('click', function() {
                    document.querySelectorAll('.size-option').forEach(o => o.classList.remove('active'));
                    this.classList.add('active');
                    
                    const size = this.dataset.size;
                    currentSize = size;
                    
                    if (size === 'custom') {
                        document.getElementById('customSizeInputs').classList.add('show');
                    } else {
                        document.getElementById('customSizeInputs').classList.remove('show');
                    }
                    
                    updateSizeInfo();
                    updatePreview();
                });
            });

            // 사용자 정의 사이즈 입력
            ['customWidth', 'customHeight'].forEach(id => {
                const element = document.getElementById(id);
                if (element) {
                    element.addEventListener('input', function() {
                        if (currentSize === 'custom') {
                            customSize.width = parseInt(document.getElementById('customWidth').value) || 105;
                            customSize.height = parseInt(document.getElementById('customHeight').value) || 148;
                            updateSizeInfo();
                            updatePreview();
                        }
                    });
                }
            });

            // 템플릿 선택 이벤트
            document.querySelectorAll('.template-option').forEach(option => {
                option.addEventListener('click', function() {
                    document.querySelectorAll('.template-option').forEach(o => o.classList.remove('active'));
                    this.classList.add('active');
                    currentTemplate = this.dataset.template;
                    updatePreview();
                });
            });
        }

        // 디바운스 함수
        function debounce(func, wait) {
            let timeout;
            return function executedFunction(...args) {
                const later = () => {
                    clearTimeout(timeout);
                    func(...args);
                };
                clearTimeout(timeout);
                timeout = setTimeout(later, wait);
            };
        }

        function mmToPx(mm, dpi = 300) {
            return Math.round((mm / 25.4) * dpi);
        }

        function getCurrentSize() {
            if (currentSize === 'custom') {
                return customSize;
            }
            return sizes[currentSize];
        }

        function updateSizeInfo() {
            const size = getCurrentSize();
            const info = currentSize === 'custom' 
                ? `사용자 정의 (${size.width}×${size.height}mm) - 300DPI`
                : `${currentSize.toUpperCase()} (${size.width}×${size.height}mm) - 300DPI`;
            const sizeInfoElement = document.getElementById('sizeInfo');
            if (sizeInfoElement) {
                sizeInfoElement.textContent = info;
            }
        }

        function getKeywordIcon(keyword) {
            const cleanKeyword = keyword.toLowerCase().trim();
            for (const [key, icon] of Object.entries(keywordIcons)) {
                if (cleanKeyword.includes(key.toLowerCase())) {
                    return icon;
                }
            }
            return '⭐';
        }

        async function updatePreview() {
            if (!librariesLoaded) return;

            const canvas = document.getElementById('popCanvas');
            if (!canvas) return;

            const ctx = canvas.getContext('2d');
            
            const size = getCurrentSize();
            const canvasWidth = mmToPx(size.width);
            const canvasHeight = mmToPx(size.height);
            
            // 고해상도 설정
            canvas.width = canvasWidth;
            canvas.height = canvasHeight;
            
            // 화면 표시 크기 조정
            const maxDisplayWidth = 400;
            const maxDisplayHeight = 500;
            const aspectRatio = canvasWidth / canvasHeight;
            
            let displayWidth, displayHeight;
            if (aspectRatio > 1) {
                displayWidth = Math.min(maxDisplayWidth, canvasWidth / 3);
                displayHeight = displayWidth / aspectRatio;
            } else {
                displayHeight = Math.min(maxDisplayHeight, canvasHeight / 3);
                displayWidth = displayHeight * aspectRatio;
            }
            
            canvas.style.width = displayWidth + 'px';
            canvas.style.height = displayHeight + 'px';
            
            // Canvas 2D Context의 roundRect 폴리필 (구형 브라우저 호환성)
            if (!ctx.roundRect) {
                ctx.roundRect = function(x, y, width, height, radius) {
                    this.beginPath();
                    this.moveTo(x + radius, y);
                    this.lineTo(x + width - radius, y);
                    this.quadraticCurveTo(x + width, y, x + width, y + radius);
                    this.lineTo(x + width, y + height - radius);
                    this.quadraticCurveTo(x + width, y + height, x + width - radius, y + height);
                    this.lineTo(x + radius, y + height);
                    this.quadraticCurveTo(x, y + height, x, y + height - radius);
                    this.lineTo(x, y + radius);
                    this.quadraticCurveTo(x, y, x + radius, y);
                    this.closePath();
                };
            }
            const template = templates[currentTemplate];
            
            if (template.isGradient) {
                // 그라디언트 배경
                const gradient = ctx.createLinearGradient(0, 0, canvasWidth, canvasHeight);
                const gradientMatch = template.background.match(/#[a-fA-F0-9]{6}/g);
                if (gradientMatch && gradientMatch.length >= 2) {
                    gradient.addColorStop(0, gradientMatch[0]);
                    gradient.addColorStop(1, gradientMatch[1]);
                } else {
                    gradient.addColorStop(0, '#667eea');
                    gradient.addColorStop(1, '#764ba2');
                }
                ctx.fillStyle = gradient;
            } else {
                // 단색 배경
                ctx.fillStyle = template.background;
            }
            
            ctx.fillRect(0, 0, canvasWidth, canvasHeight);
            
            // 테두리
            ctx.strokeStyle = template.borderColor;
            ctx.lineWidth = template.isGradient ? 8 : 4;
            const borderOffset = template.isGradient ? 4 : 2;
            ctx.strokeRect(borderOffset, borderOffset, canvasWidth - (borderOffset * 2), canvasHeight - (borderOffset * 2));
            
            // 폰트 설정
            const baseFontSize = Math.max(24, canvasWidth / 15);
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            
            // 제품명
            const productNameElement = document.getElementById('productName');
            const productName = productNameElement ? productNameElement.value || '제품명을 입력하세요' : '제품명을 입력하세요';
            ctx.font = `bold ${baseFontSize * 1.2}px "Malgun Gothic", Arial, sans-serif`;
            ctx.fillStyle = template.titleColor;
            ctx.fillText(productName, canvasWidth / 2, canvasHeight * 0.2);
            
            // 가격
            const productPriceElement = document.getElementById('productPrice');
            const productPrice = productPriceElement ? productPriceElement.value || '가격을 입력하세요' : '가격을 입력하세요';
            ctx.font = `bold ${baseFontSize * 1.5}px "Malgun Gothic", Arial, sans-serif`;
            ctx.fillStyle = template.priceColor;
            ctx.fillText(productPrice, canvasWidth / 2, canvasHeight * 0.35);
            
            // 설명
            const descriptionElement = document.getElementById('productDescription');
            const description = descriptionElement ? descriptionElement.value || '설명을 입력하세요' : '설명을 입력하세요';
            ctx.font = `${baseFontSize * 0.8}px "Malgun Gothic", Arial, sans-serif`;
            ctx.fillStyle = template.descColor;
            
            // 긴 텍스트 줄바꿈 처리
            const maxWidth = canvasWidth * 0.8;
            const lines = wrapText(ctx, description, maxWidth);
            lines.forEach((line, index) => {
                ctx.fillText(line, canvasWidth / 2, canvasHeight * 0.5 + (index * baseFontSize * 0.9));
            });
            
            // 키워드 배치
            const keywords = [];
            for (let i = 1; i <= 3; i++) {
                const keywordElement = document.getElementById(`keyword${i}`);
                if (keywordElement && keywordElement.value.trim()) {
                    keywords.push(keywordElement.value.trim());
                }
            }
            
            if (keywords.length > 0) {
                const keywordY = canvasHeight * 0.72;
                const keywordSpacing = canvasWidth / (keywords.length + 1);
                
                keywords.forEach((keyword, index) => {
                    const x = keywordSpacing * (index + 1);
                    const icon = getKeywordIcon(keyword);
                    
                    // 키워드 배경
                    ctx.fillStyle = template.keywordBg;
                    const bgWidth = Math.min(120, canvasWidth * 0.2);
                    const bgHeight = 50;
                    
                    if (template.isGradient) {
                        ctx.fillRect(x - bgWidth/2, keywordY - bgHeight/2, bgWidth, bgHeight);
                    } else {
                        // 심플 템플릿용 둥근 모서리 배경
                        ctx.beginPath();
                        ctx.roundRect(x - bgWidth/2, keywordY - bgHeight/2, bgWidth, bgHeight, 8);
                        ctx.fill();
                        
                        // 테두리 추가 (심플 템플릿용)
                        ctx.strokeStyle = template.borderColor;
                        ctx.lineWidth = 1;
                        ctx.stroke();
                    }
                    
                    // 아이콘
                    ctx.font = `${baseFontSize * 0.9}px "Malgun Gothic", Arial, sans-serif`;
                    ctx.fillStyle = template.keywordColor;
                    ctx.fillText(icon, x, keywordY - 8);
                    
                    // 키워드 텍스트
                    ctx.font = `${baseFontSize * 0.55}px "Malgun Gothic", Arial, sans-serif`;
                    ctx.fillStyle = template.keywordColor;
                    ctx.fillText(keyword, x, keywordY + 12);
                });
            }
            
            // QR 코드
            const qrUrlElement = document.getElementById('qrUrl');
            const qrUrl = qrUrlElement ? qrUrlElement.value : '';
            if (qrUrl && typeof QRious !== 'undefined') {
                try {
                    const qrSize = Math.min(canvasWidth * 0.15, 120);
                    const qrCanvas = document.createElement('canvas');
                    const qr = new QRious({
                        element: qrCanvas,
                        value: qrUrl,
                        size: qrSize,
                        background: 'white',
                        foreground: 'black'
                    });
                    
                    // QR 코드 배경 (심플 템플릿용)
                    if (!template.isGradient) {
                        ctx.fillStyle = 'white';
                        ctx.fillRect(canvasWidth - qrSize - 25, canvasHeight - qrSize - 25, qrSize + 10, qrSize + 10);
                        ctx.strokeStyle = template.borderColor;
                        ctx.lineWidth = 2;
                        ctx.strokeRect(canvasWidth - qrSize - 25, canvasHeight - qrSize - 25, qrSize + 10, qrSize + 10);
                    }
                    
                    ctx.drawImage(qrCanvas, canvasWidth - qrSize - 20, canvasHeight - qrSize - 20, qrSize, qrSize);
                } catch (error) {
                    console.log('QR 코드 생성 오류:', error);
                }
            }
        }

        function wrapText(ctx, text, maxWidth) {
            const words = text.split(' ');
            const lines = [];
            let currentLine = words[0] || '';

            for (let i = 1; i < words.length; i++) {
                const word = words[i];
                const width = ctx.measureText(currentLine + " " + word).width;
                if (width < maxWidth) {
                    currentLine += " " + word;
                } else {
                    lines.push(currentLine);
                    currentLine = word;
                }
            }
            lines.push(currentLine);
            return lines;
        }

        async function downloadPDF() {
            if (!librariesLoaded || typeof jsPDF === 'undefined') {
                alert('PDF 라이브러리가 로딩되지 않았습니다. 잠시 후 다시 시도해주세요.');
                return;
            }

            const canvas = document.getElementById('popCanvas');
            if (!canvas) {
                alert('캔버스를 찾을 수 없습니다.');
                return;
            }
            
            const size = getCurrentSize();
            
            try {
                // jsPDF 객체 생성
                const { jsPDF } = window.jspdf;
                const pdf = new jsPDF({
                    orientation: size.width > size.height ? 'landscape' : 'portrait',
                    unit: 'mm',
                    format: [size.width, size.height]
                });
                
                const imgData = canvas.toDataURL('image/jpeg', 1.0);
                pdf.addImage(imgData, 'JPEG', 0, 0, size.width, size.height);
                
                const filename = `반려동물_POP_${size.width}x${size.height}mm.pdf`;
                pdf.save(filename);
            } catch (error) {
                console.error('PDF 생성 오류:', error);
                alert('PDF 생성 중 오류가 발생했습니다. 브라우저를 새로고침 후 다시 시도해주세요.');
            }
        }

        async function downloadPNG() {
            const canvas = document.getElementById('popCanvas');
            if (!canvas) {
                alert('캔버스를 찾을 수 없습니다.');
                return;
            }
            
            try {
                const size = getCurrentSize();
                const link = document.createElement('a');
                link.download = `반려동물_POP_${size.width}x${size.height}mm.png`;
                link.href = canvas.toDataURL('image/png');
                link.click();
            } catch (error) {
                console.error('PNG 생성 오류:', error);
                alert('PNG 생성 중 오류가 발생했습니다. 다시 시도해주세요.');
            }
        }

        // 페이지 로드 완료 후 초기 미리보기 업데이트
        window.addEventListener('load', function() {
            setTimeout(() => {
                if (librariesLoaded) {
                    updatePreview();
                }
            }, 1000);
        });
    </script>
</body>
</html>
