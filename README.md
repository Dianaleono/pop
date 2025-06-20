<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AR Поворот Квадрата - MindAR</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.2/dist/mindar-image.prod.js"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            background: #000;
            font-family: 'Arial', sans-serif;
            overflow: hidden;
        }
        
        #ar-container {
            position: relative;
            width: 100vw;
            height: 100vh;
        }
        
        .ui-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 1000;
        }
        
        .controls {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 15px;
            border-radius: 10px;
            pointer-events: all;
            backdrop-filter: blur(10px);
        }
        
        .controls h3 {
            margin: 0 0 10px 0;
            color: #4CAF50;
        }
        
        .control-group {
            margin: 10px 0;
        }
        
        .control-group label {
            display: block;
            margin-bottom: 5px;
            font-size: 14px;
        }
        
        .control-group input, .control-group button {
            width: 100%;
            padding: 8px;
            border: none;
            border-radius: 5px;
            margin-bottom: 5px;
        }
        
        .control-group input {
            background: rgba(255, 255, 255, 0.1);
            color: white;
            border: 1px solid rgba(255, 255, 255, 0.3);
        }
        
        .control-group button {
            background: #4CAF50;
            color: white;
            cursor: pointer;
            transition: background 0.3s;
        }
        
        .control-group button:hover {
            background: #45a049;
        }
        
        .info-panel {
            position: absolute;
            bottom: 20px;
            right: 20px;
            background: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 15px;
            border-radius: 10px;
            pointer-events: all;
            backdrop-filter: blur(10px);
            max-width: 300px;
        }
        
        .loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            font-size: 18px;
            text-align: center;
        }
        
        .marker-instructions {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.9);
            color: white;
            padding: 20px;
            border-radius: 15px;
            text-align: center;
            pointer-events: all;
            backdrop-filter: blur(10px);
        }
        
        .marker-instructions button {
            background: #4CAF50;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 15px;
        }
        
        .angle-display {
            font-size: 24px;
            font-weight: bold;
            color: #4CAF50;
            text-align: center;
            margin: 10px 0;
        }
    </style>
</head>
<body>
    <div id="ar-container">
        <div class="loading" id="loading">
            <div>🔄 Завантаження AR системи...</div>
            <div style="font-size: 14px; margin-top: 10px;">Підготовка камери та розпізнавання маркерів</div>
        </div>
        
        <div class="marker-instructions" id="marker-instructions" style="display: none;">
            <h3>📱 Інструкції для AR</h3>
            <p>1. Дозвольте доступ до камери</p>
            <p>2. Наведіть камеру на маркер</p>
            <p>3. Спостерігайте за поворотом квадрата!</p>
            <button onclick="hideInstructions()">Зрозуміло</button>
        </div>
        
        <div class="ui-overlay">
            <div class="controls">
                <h3>🔄 Керування поворотом</h3>
                
                <div class="control-group">
                    <label>Кут повороту (градуси):</label>
                    <input type="range" id="angleSlider" min="0" max="360" value="0" step="15">
                    <div class="angle-display" id="angleDisplay">0°</div>
                </div>
                
                <div class="control-group">
                    <label>Швидкість анімації:</label>
                    <input type="range" id="speedSlider" min="0.5" max="5" value="1" step="0.5">
                </div>
                
                <div class="control-group">
                    <button onclick="toggleAutoRotation()">▶️ Авто поворот</button>
                    <button onclick="resetRotation()">🔄 Скинути</button>
                    <button onclick="rotateBy90()">↻ +90°</button>
                </div>
            </div>
            
            <div class="info-panel">
                <h4>📐 Поворот квадрата</h4>
                <p><strong>Центр повороту:</strong> Центр квадрата</p>
                <p><strong>Поточний кут:</strong> <span id="currentAngle">0°</span></p>
                <p><strong>Тип перетворення:</strong> Ізометрія</p>
                <p><strong>Властивості:</strong> Зберігає форму та розмір</p>
                <div style="margin-top: 10px; font-size: 12px; opacity: 0.8;">
                    💡 Поворот - це перетворення, при якому кожна точка фігури повертається навколо фіксованої точки на заданий кут
                </div>
            </div>
        </div>
    </div>

    <script>
        class ARSquareRotation {
            constructor() {
                this.mindarThree = null;
                this.scene = null;
                this.camera = null;
                this.renderer = null;
                this.square = null;
                this.centerPoint = null;
                this.rotationAngle = 0;
                this.targetAngle = 0;
                this.animationSpeed = 1;
                this.autoRotate = false;
                this.isMarkerVisible = false;
                
                this.originalSquare = null;
                this.rotatedSquare = null;
                this.rotationLine = null;
                
                this.init();
            }
            
            async init() {
                try {
                    // Створюємо базовий маркер (в реальному проекті потрібен файл .mind)
                    const markerData = this.createDummyMarkerData();
                    
                    // Ініціалізація MindAR
                    this.mindarThree = new window.MINDAR.IMAGE.MindARThree({
                        container: document.getElementById('ar-container'),
                        imageTargetSrc: markerData, // В реальному проекті: './markers/square-marker.mind'
                    });
                    
                    const { renderer, scene, camera } = this.mindarThree;
                    this.renderer = renderer;
                    this.scene = scene;
                    this.camera = camera;
                    
                    // Створюємо 3D контент
                    this.create3DContent();
                    
                    // Налаштовуємо обробники подій
                    this.setupEventListeners();
                    
                    // Запуск AR
                    await this.mindarThree.start();
                    
                    // Ховаємо завантаження, показуємо інструкції
                    document.getElementById('loading').style.display = 'none';
                    document.getElementById('marker-instructions').style.display = 'block';
                    
                    // Запуск анімаційного циклу
                    this.animate();
                    
                } catch (error) {
                    console.error('Помилка ініціалізації AR:', error);
                    this.showError('Помилка завантаження AR системи. Перевірте підтримку браузера.');
                }
            }
            
            createDummyMarkerData() {
                // Замість реального .mind файлу створюємо заглушку
                // В реальному проекті тут буде посилання на підготовлений маркер
                return 'data:application/octet-stream;base64,UklGRgAAAABXQVZFZm10IBAAAAABAAEA';
            }
            
            create3DContent() {
                // Створюємо групу для всіх об'єктів
                const anchor = this.mindarThree.addAnchor(0);
                
                // Освітлення
                const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
                anchor.group.add(ambientLight);
                
                const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
                directionalLight.position.set(1, 1, 1);
                anchor.group.add(directionalLight);
                
                // Центральна точка (центр повороту)
                this.createCenterPoint(anchor.group);
                
                // Оригінальний квадрат (напівпрозорий)
                this.createOriginalSquare(anchor.group);
                
                // Квадрат, що повертається
                this.createRotatingSquare(anchor.group);
                
                // Лінія, що показує поворот
                this.createRotationIndicator(anchor.group);
                
                // Координатні осі для орієнтації
                this.createAxes(anchor.group);
                
                // Обробник видимості маркера
                anchor.onTargetFound = () => {
                    this.isMarkerVisible = true;
                    console.log('🎯 Маркер знайдено!');
                };
                
                anchor.onTargetLost = () => {
                    this.isMarkerVisible = false;
                    console.log('❌ Маркер втрачено');
                };
            }
            
            createCenterPoint(parent) {
                const geometry = new THREE.SphereGeometry(0.02, 16, 16);
                const material = new THREE.MeshBasicMaterial({ 
                    color: 0xff0000,
                    transparent: true,
                    opacity: 0.8
                });
                this.centerPoint = new THREE.Mesh(geometry, material);
                parent.add(this.centerPoint);
                
                // Текст-підпис
                const loader = new THREE.FontLoader();
                // Замість завантаження шрифту використовуємо 3D текст
                this.createTextLabel(parent, 'O', 0, -0.1, 0, 0xff0000);
            }
            
            createOriginalSquare(parent) {
                const size = 0.3;
                const geometry = new THREE.PlaneGeometry(size, size);
                const material = new THREE.MeshBasicMaterial({ 
                    color: 0x4CAF50,
                    transparent: true,
                    opacity: 0.3,
                    side: THREE.DoubleSide,
                    wireframe: true
                });
                
                this.originalSquare = new THREE.Mesh(geometry, material);
                parent.add(this.originalSquare);
            }
            
            createRotatingSquare(parent) {
                const size = 0.3;
                const geometry = new THREE.PlaneGeometry(size, size);
                const material = new THREE.MeshLambertMaterial({ 
                    color: 0x2196F3,
                    transparent: true,
                    opacity: 0.8,
                    side: THREE.DoubleSide
                });
                
                this.rotatedSquare = new THREE.Mesh(geometry, material);
                
                // Додаємо контур
                const edges = new THREE.EdgesGeometry(geometry);
                const lineMaterial = new THREE.LineBasicMaterial({ color: 0x000000, linewidth: 2 });
                const wireframe = new THREE.LineSegments(edges, lineMaterial);
                this.rotatedSquare.add(wireframe);
                
                parent.add(this.rotatedSquare);
                this.square = this.rotatedSquare;
                
                // Позначаємо вершини квадрата
                this.createSquareVertices(parent);
            }
            
            createSquareVertices(parent) {
                const size = 0.3;
                const positions = [
                    [-size/2, -size/2, 0.001], // A
                    [size/2, -size/2, 0.001],  // B
                    [size/2, size/2, 0.001],   // C
                    [-size/2, size/2, 0.001]   // D
                ];
                
                const labels = ['A', 'B', 'C', 'D'];
                const colors = [0xff9800, 0xe91e63, 0x9c27b0, 0x3f51b5];
                
                this.vertices = [];
                
                positions.forEach((pos, index) => {
                    // Створюємо точку
                    const geometry = new THREE.SphereGeometry(0.015, 8, 8);
                    const material = new THREE.MeshBasicMaterial({ color: colors[index] });
                    const vertex = new THREE.Mesh(geometry, material);
                    vertex.position.set(pos[0], pos[1], pos[2]);
                    
                    this.vertices.push(vertex);
                    parent.add(vertex);
                    
                    // Додаємо підпис
                    this.createTextLabel(parent, labels[index], pos[0], pos[1] - 0.05, pos[2], colors[index]);
                });
            }
            
            createRotationIndicator(parent) {
                // Коло, що показує траєкторію повороту
                const geometry = new THREE.RingGeometry(0.2, 0.22, 32);
                const material = new THREE.MeshBasicMaterial({ 
                    color: 0xffeb3b,
                    transparent: true,
                    opacity: 0.4,
                    side: THREE.DoubleSide
                });
                
                const circle = new THREE.Mesh(geometry, material);
                parent.add(circle);
                
                // Стрілка для показу напрямку
                this.createRotationArrow(parent);
            }
            
            createRotationArrow(parent) {
                const arrowGeometry = new THREE.ConeGeometry(0.02, 0.05, 8);
                const arrowMaterial = new THREE.MeshBasicMaterial({ color: 0xff5722 });
                const arrow = new THREE.Mesh(arrowGeometry, arrowMaterial);
                
                arrow.position.set(0.21, 0, 0.001);
                arrow.rotation.z = -Math.PI / 2;
                
                parent.add(arrow);
                this.rotationArrow = arrow;
            }
            
            createAxes(parent) {
                // X-ось (червона)
                this.createAxis(parent, new THREE.Vector3(0.5, 0, 0), 0xff0000, 'X');
                // Y-ось (зелена)
                this.createAxis(parent, new THREE.Vector3(0, 0.5, 0), 0x00ff00, 'Y');
            }
            
            createAxis(parent, direction, color, label) {
                const geometry = new THREE.BufferGeometry().setFromPoints([
                    new THREE.Vector3(0, 0, 0),
                    direction
                ]);
                const material = new THREE.LineBasicMaterial({ color: color });
                const line = new THREE.Line(geometry, material);
                parent.add(line);
                
                // Підпис осі
                this.createTextLabel(parent, label, direction.x, direction.y, direction.z, color);
            }
            
            createTextLabel(parent, text, x, y, z, color) {
                // Простий спосіб створення 3D тексту за допомогою плоскої геометрії
                const canvas = document.createElement('canvas');
                const context = canvas.getContext('2d');
                canvas.width = 64;
                canvas.height = 64;
                
                context.fillStyle = `#${color.toString(16).padStart(6, '0')}`;
                context.font = '32px Arial';
                context.textAlign = 'center';
                context.fillText(text, 32, 40);
                
                const texture = new THREE.CanvasTexture(canvas);
                const geometry = new THREE.PlaneGeometry(0.05, 0.05);
                const material = new THREE.MeshBasicMaterial({ 
                    map: texture, 
                    transparent: true 
                });
                
                const label = new THREE.Mesh(geometry, material);
                label.position.set(x, y, z);
                parent.add(label);
            }
            
            setupEventListeners() {
                // Слайдер кута
                const angleSlider = document.getElementById('angleSlider');
                angleSlider.addEventListener('input', (e) => {
                    this.targetAngle = parseFloat(e.target.value) * Math.PI / 180;
                    this.updateAngleDisplay(e.target.value);
                });
                
                // Слайдер швидкості
                const speedSlider = document.getElementById('speedSlider');
                speedSlider.addEventListener('input', (e) => {
                    this.animationSpeed = parseFloat(e.target.value);
                });
                
                // Оновлення дисплею кута
                this.updateAngleDisplay(0);
            }
            
            updateAngleDisplay(degrees) {
                document.getElementById('angleDisplay').textContent = `${degrees}°`;
                document.getElementById('currentAngle').textContent = `${degrees}°`;
            }
            
            toggleAutoRotation() {
                this.autoRotate = !this.autoRotate;
                const button = event.target;
                button.textContent = this.autoRotate ? '⏸️ Зупинити' : '▶️ Авто поворот';
            }
            
            resetRotation() {
                this.targetAngle = 0;
                this.rotationAngle = 0;
                document.getElementById('angleSlider').value = 0;
                this.updateAngleDisplay(0);
            }
            
            rotateBy90() {
                const currentDegrees = this.targetAngle * 180 / Math.PI;
                const newDegrees = ((currentDegrees + 90) % 360);
                this.targetAngle = newDegrees * Math.PI / 180;
                document.getElementById('angleSlider').value = newDegrees;
                this.updateAngleDisplay(newDegrees);
            }
            
            animate() {
                requestAnimationFrame(() => this.animate());
                
                if (this.square && this.isMarkerVisible) {
                    // Автоматичний поворот
                    if (this.autoRotate) {
                        this.targetAngle += 0.02 * this.animationSpeed;
                        const degrees = (this.targetAngle * 180 / Math.PI) % 360;
                        document.getElementById('angleSlider').value = degrees;
                        this.updateAngleDisplay(Math.round(degrees));
                    }
                    
                    // Плавна анімація до цільового кута
                    const angleDiff = this.targetAngle - this.rotationAngle;
                    this.rotationAngle += angleDiff * 0.1 * this.animationSpeed;
                    
                    // Застосовуємо поворот до квадрата
                    this.square.rotation.z = this.rotationAngle;
                    
                    // Повертаємо вершини
                    if (this.vertices) {
                        const size = 0.3;
                        const positions = [
                            [-size/2, -size/2], // A
                            [size/2, -size/2],  // B
                            [size/2, size/2],   // C
                            [-size/2, size/2]   // D
                        ];
                        
                        this.vertices.forEach((vertex, index) => {
                            const [x, y] = positions[index];
                            const cos = Math.cos(this.rotationAngle);
                            const sin = Math.sin(this.rotationAngle);
                            
                            const newX = x * cos - y * sin;
                            const newY = x * sin + y * cos;
                            
                            vertex.position.set(newX, newY, 0.001);
                        });
                    }
                    
                    // Оновлюємо стрілку напрямку
                    if (this.rotationArrow) {
                        this.rotationArrow.rotation.z = -Math.PI / 2 + this.rotationAngle;
                    }
                }
                
                this.renderer.render(this.scene, this.camera);
            }
            
            showError(message) {
                document.getElementById('loading').innerHTML = `
                    <div style="color: #ff5722;">❌ ${message}</div>
                    <div style="font-size: 14px; margin-top: 10px;">
                        Переконайтесь, що:
                        <br>• Браузер підтримує WebGL
                        <br>• Дозволено доступ до камери
                        <br>• Використовується HTTPS
                    </div>
                `;
            }
        }
        
        // Глобальні функції для керування
        function hideInstructions() {
            document.getElementById('marker-instructions').style.display = 'none';
        }
        
        function toggleAutoRotation() {
            if (window.arApp) {
                window.arApp.toggleAutoRotation();
            }
        }
        
        function resetRotation() {
            if (window.arApp) {
                window.arApp.resetRotation();
            }
        }
        
        function rotateBy90() {
            if (window.arApp) {
                window.arApp.rotateBy90();
            }
        }
        
        // Запуск додатку
        window.addEventListener('load', () => {
            // Перевірка підтримки
            if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                document.getElementById('loading').innerHTML = `
                    <div style="color: #ff5722;">❌ Браузер не підтримує доступ до камери</div>
                    <div style="font-size: 14px; margin-top: 10px;">
                        Спробуйте Chrome, Firefox або Safari
                    </div>
                `;
                return;
            }
            
            if (!window.THREE) {
                document.getElementById('loading').innerHTML = `
                    <div style="color: #ff5722;">❌ Помилка завантаження Three.js</div>
                `;
                return;
            }
            
            // Ініціалізація AR додатку
            window.arApp = new ARSquareRotation();
        });
        
        // Обробка помилок
        window.addEventListener('error', (e) => {
            console.error('Глобальна помилка:', e.error);
        });
        
        // Адаптивність для мобільних пристроїв
        window.addEventListener('orientationchange', () => {
            setTimeout(() => {
                if (window.arApp && window.arApp.renderer) {
                    window.arApp.renderer.setSize(window.innerWidth, window.innerHeight);
                }
            }, 500);
        });
    </script>
</body>
</html>
