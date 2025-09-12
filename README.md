<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Particle Simulator</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background: radial-gradient(ellipse at center, #1a1a2e 0%, #16213e 50%, #0f0f23 100%);
            overflow: hidden;
            font-family: 'Arial', sans-serif;
            color: white;
        }
        
        #container {
            position: relative;
            width: 100vw;
            height: 100vh;
        }
        
        #controls {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 100;
            background: rgba(15, 15, 35, 0.9);
            padding: 20px;
            border-radius: 15px;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
            min-width: 280px;
        }
        
        #controls h1 {
            margin: 0 0 15px 0;
            font-size: 22px;
            background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            animation: gradientShift 3s ease-in-out infinite alternate;
        }
        
        @keyframes gradientShift {
            0% { filter: hue-rotate(0deg); }
            100% { filter: hue-rotate(60deg); }
        }
        
        .control-group {
            margin: 15px 0;
        }
        
        label {
            display: block;
            margin-bottom: 8px;
            font-size: 14px;
            opacity: 0.9;
            font-weight: 500;
        }
        
        input[type="range"] {
            width: 100%;
            height: 6px;
            border-radius: 3px;
            background: rgba(255, 255, 255, 0.2);
            outline: none;
            -webkit-appearance: none;
            transition: all 0.3s ease;
        }
        
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 18px;
            height: 18px;
            border-radius: 50%;
            background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
            cursor: pointer;
            box-shadow: 0 0 10px rgba(78, 205, 196, 0.5);
            transition: all 0.3s ease;
        }
        
        input[type="range"]::-webkit-slider-thumb:hover {
            transform: scale(1.2);
            box-shadow: 0 0 20px rgba(78, 205, 196, 0.8);
        }
        
        input[type="range"]::-moz-range-thumb {
            width: 18px;
            height: 18px;
            border-radius: 50%;
            background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
            cursor: pointer;
            border: none;
            box-shadow: 0 0 10px rgba(78, 205, 196, 0.5);
        }
        
        .value-display {
            display: inline-block;
            background: rgba(255, 255, 255, 0.1);
            padding: 4px 8px;
            border-radius: 8px;
            font-size: 12px;
            margin-left: 10px;
            min-width: 40px;
            text-align: center;
        }
        
        button {
            background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
            border: none;
            padding: 10px 20px;
            border-radius: 10px;
            color: white;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s ease;
            margin-right: 10px;
            font-size: 14px;
        }
        
        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(255, 107, 107, 0.4);
        }
        
        #info {
            position: absolute;
            bottom: 20px;
            right: 20px;
            z-index: 100;
            background: rgba(15, 15, 35, 0.8);
            padding: 15px;
            border-radius: 10px;
            backdrop-filter: blur(10px);
            font-size: 12px;
            opacity: 0.8;
        }
        
        #stats {
            margin-top: 10px;
            font-size: 12px;
            opacity: 0.7;
        }
    </style>
</head>
<body>
    <div id="container">
        <div id="controls">
            <h1>🌌 Particle Simulator</h1>
            
            <div class="control-group">
                <label>Particle Speed <span class="value-display" id="speedValue">1.0</span></label>
                <input type="range" id="speedSlider" min="0" max="5" step="0.1" value="1">
            </div>
            
            <div class="control-group">
                <label>Animation Type <span class="value-display" id="typeValue">Wave</span></label>
                <input type="range" id="typeSlider" min="0" max="3" step="1" value="0">
            </div>
            
            <div class="control-group">
                <label>Particle Size <span class="value-display" id="sizeValue">0.05</span></label>
                <input type="range" id="sizeSlider" min="0.02" max="0.15" step="0.01" value="0.05">
            </div>
            
            <div class="control-group">
                <button onclick="resetParticles()">🔄 Reset</button>
                <button onclick="togglePause()">⏸️ Pause</button>
            </div>
            
            <div id="stats">
                <div>Particles: <span id="particleCount">0</span></div>
                <div>FPS: <span id="fps">60</span></div>
            </div>
        </div>
        
        <div id="info">
            <div>🖱️ Drag to rotate view</div>
            <div>🔍 Scroll to zoom</div>
            <div>⚡ Real-time particle physics</div>
        </div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        let scene, camera, renderer, particles, particleSystem;
        let particleCount = 8000;
        let positions = new Float32Array(particleCount * 3);
        let velocities = new Float32Array(particleCount * 3);
        let originalPositions = new Float32Array(particleCount * 3);
        
        let mouseX = 0, mouseY = 0;
        let targetRotationX = 0, targetRotationY = 0;
        let isMouseDown = false;
        let isPaused = false;
        
        let speedMultiplier = 1.0;
        let animationType = 0; // 0: wave, 1: explosion, 2: spiral, 3: chaos
        let particleSize = 0.05;
        
        // FPS tracking
        let frameCount = 0;
        let lastTime = performance.now();
        
        const animationTypes = ['Wave', 'Explosion', 'Spiral', 'Chaos'];
        
        function init() {
            // Create scene
            scene = new THREE.Scene();
            
            // Create camera
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 0, 10);
            
            // Create renderer
            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setClearColor(0x000000, 0);
            document.getElementById('container').appendChild(renderer.domElement);
            
            // Initialize particles
            createParticleSystem();
            
            // Setup controls
            setupControls();
            setupEventListeners();
            
            // Start animation loop
            animate();
            
            // Update particle count display
            document.getElementById('particleCount').textContent = particleCount;
        }
        
        function createParticleSystem() {
            // Create particles in a cube formation
            let i = 0;
            const size = 4;
            const spacing = 0.2;
            
            for (let x = -size; x <= size; x += spacing) {
                for (let y = -size; y <= size; y += spacing) {
                    for (let z = -size; z <= size; z += spacing) {
                        if (i >= particleCount) break;
                        
                        positions[i * 3] = x + (Math.random() - 0.5) * 0.1;
                        positions[i * 3 + 1] = y + (Math.random() - 0.5) * 0.1;
                        positions[i * 3 + 2] = z + (Math.random() - 0.5) * 0.1;
                        
                        // Store original positions
                        originalPositions[i * 3] = positions[i * 3];
                        originalPositions[i * 3 + 1] = positions[i * 3 + 1];
                        originalPositions[i * 3 + 2] = positions[i * 3 + 2];
                        
                        // Initialize velocities
                        velocities[i * 3] = (Math.random() - 0.5) * 0.02;
                        velocities[i * 3 + 1] = (Math.random() - 0.5) * 0.02;
                        velocities[i * 3 + 2] = (Math.random() - 0.5) * 0.02;
                        
                        i++;
                    }
                }
            }
            
            // Update actual particle count
            particleCount = i;
            
            // Create geometry
            const geometry = new THREE.BufferGeometry();
            geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
            
            // Create material
            const material = new THREE.PointsMaterial({
                color: 0x4ecdc4,
                size: particleSize,
                transparent: true,
                opacity: 0.8,
                vertexColors: false,
                blending: THREE.AdditiveBlending
            });
            
            // Create particle system
            particleSystem = new THREE.Points(geometry, material);
            scene.add(particleSystem);
        }
        
        function updateParticles() {
            if (isPaused) return;
            
            const time = performance.now() * 0.001 * speedMultiplier;
            
            for (let i = 0; i < particleCount; i++) {
                const i3 = i * 3;
                
                switch (animationType) {
                    case 0: // Wave
                        positions[i3] = originalPositions[i3] + Math.sin(time + originalPositions[i3] * 2) * 0.5;
                        positions[i3 + 1] = originalPositions[i3 + 1] + Math.cos(time + originalPositions[i3 + 1] * 2) * 0.5;
                        positions[i3 + 2] = originalPositions[i3 + 2] + Math.sin(time + originalPositions[i3 + 2] * 2) * 0.3;
                        break;
                        
                    case 1: // Explosion
                        const distance = Math.sqrt(
                            originalPositions[i3] ** 2 + 
                            originalPositions[i3 + 1] ** 2 + 
                            originalPositions[i3 + 2] ** 2
                        );
                        const explosionForce = Math.sin(time * 2) * 2;
                        
                        positions[i3] = originalPositions[i3] * (1 + explosionForce / distance);
                        positions[i3 + 1] = originalPositions[i3 + 1] * (1 + explosionForce / distance);
                        positions[i3 + 2] = originalPositions[i3 + 2] * (1 + explosionForce / distance);
                        break;
                        
                    case 2: // Spiral
                        const radius = Math.sqrt(originalPositions[i3] ** 2 + originalPositions[i3 + 2] ** 2);
                        const angle = Math.atan2(originalPositions[i3 + 2], originalPositions[i3]) + time;
                        
                        positions[i3] = Math.cos(angle) * radius;
                        positions[i3 + 1] = originalPositions[i3 + 1] + Math.sin(time * 2 + radius) * 0.5;
                        positions[i3 + 2] = Math.sin(angle) * radius;
                        break;
                        
                    case 3: // Chaos
                        velocities[i3] += (Math.random() - 0.5) * 0.001 * speedMultiplier;
                        velocities[i3 + 1] += (Math.random() - 0.5) * 0.001 * speedMultiplier;
                        velocities[i3 + 2] += (Math.random() - 0.5) * 0.001 * speedMultiplier;
                        
                        positions[i3] += velocities[i3];
                        positions[i3 + 1] += velocities[i3 + 1];
                        positions[i3 + 2] += velocities[i3 + 2];
                        
                        // Add slight attraction back to original position
                        positions[i3] += (originalPositions[i3] - positions[i3]) * 0.001;
                        positions[i3 + 1] += (originalPositions[i3 + 1] - positions[i3 + 1]) * 0.001;
                        positions[i3 + 2] += (originalPositions[i3 + 2] - positions[i3 + 2]) * 0.001;
                        
                        // Damping
                        velocities[i3] *= 0.99;
                        velocities[i3 + 1] *= 0.99;
                        velocities[i3 + 2] *= 0.99;
                        break;
                }
            }
            
            particleSystem.geometry.attributes.position.needsUpdate = true;
        }
        
        function setupControls() {
            const speedSlider = document.getElementById('speedSlider');
            const typeSlider = document.getElementById('typeSlider');
            const sizeSlider = document.getElementById('sizeSlider');
            
            speedSlider.addEventListener('input', (e) => {
                speedMultiplier = parseFloat(e.target.value);
                document.getElementById('speedValue').textContent = speedMultiplier.toFixed(1);
            });
            
            typeSlider.addEventListener('input', (e) => {
                animationType = parseInt(e.target.value);
                document.getElementById('typeValue').textContent = animationTypes[animationType];
            });
            
            sizeSlider.addEventListener('input', (e) => {
                particleSize = parseFloat(e.target.value);
                document.getElementById('sizeValue').textContent = particleSize.toFixed(2);
                particleSystem.material.size = particleSize;
            });
        }
        
        function setupEventListeners() {
            const container = document.getElementById('container');
            
            // Mouse controls
            container.addEventListener('mousedown', onMouseDown, false);
            container.addEventListener('mousemove', onMouseMove, false);
            container.addEventListener('mouseup', onMouseUp, false);
            container.addEventListener('wheel', onWheel, false);
            
            // Touch controls
            container.addEventListener('touchstart', onTouchStart, false);
            container.addEventListener('touchmove', onTouchMove, false);
            container.addEventListener('touchend', onTouchEnd, false);
            
            // Window resize
            window.addEventListener('resize', onWindowResize, false);
        }
        
        function onMouseDown(event) {
            isMouseDown = true;
            mouseX = event.clientX;
            mouseY = event.clientY;
        }
        
        function onMouseMove(event) {
            if (isMouseDown) {
                const deltaX = event.clientX - mouseX;
                const deltaY = event.clientY - mouseY;
                
                targetRotationY += deltaX * 0.01;
                targetRotationX += deltaY * 0.01;
                
                mouseX = event.clientX;
                mouseY = event.clientY;
            }
        }
        
        function onMouseUp(event) {
            isMouseDown = false;
        }
        
        function onWheel(event) {
            camera.position.z += event.deltaY * 0.01;
            camera.position.z = Math.max(5, Math.min(20, camera.position.z));
        }
        
        function onTouchStart(event) {
            if (event.touches.length === 1) {
                mouseX = event.touches[0].clientX;
                mouseY = event.touches[0].clientY;
                isMouseDown = true;
            }
        }
        
        function onTouchMove(event) {
            if (event.touches.length === 1 && isMouseDown) {
                const deltaX = event.touches[0].clientX - mouseX;
                const deltaY = event.touches[0].clientY - mouseY;
                
                targetRotationY += deltaX * 0.01;
                targetRotationX += deltaY * 0.01;
                
                mouseX = event.touches[0].clientX;
                mouseY = event.touches[0].clientY;
            }
            event.preventDefault();
        }
        
        function onTouchEnd(event) {
            isMouseDown = false;
        }
        
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }
        
        function resetParticles() {
            // Reset to original positions
            for (let i = 0; i < particleCount; i++) {
                positions[i * 3] = originalPositions[i * 3];
                positions[i * 3 + 1] = originalPositions[i * 3 + 1];
                positions[i * 3 + 2] = originalPositions[i * 3 + 2];
                
                velocities[i * 3] = (Math.random() - 0.5) * 0.02;
                velocities[i * 3 + 1] = (Math.random() - 0.5) * 0.02;
                velocities[i * 3 + 2] = (Math.random() - 0.5) * 0.02;
            }
            particleSystem.geometry.attributes.position.needsUpdate = true;
        }
        
        function togglePause() {
            isPaused = !isPaused;
            const button = event.target;
            button.textContent = isPaused ? '▶️ Play' : '⏸️ Pause';
        }
        
        function updateFPS() {
            frameCount++;
            const currentTime = performance.now();
            if (currentTime - lastTime >= 1000) {
                const fps = Math.round(frameCount * 1000 / (currentTime - lastTime));
                document.getElementById('fps').textContent = fps;
                frameCount = 0;
                lastTime = currentTime;
            }
        }
        
        function animate() {
            requestAnimationFrame(animate);
            
            // Update particles
            updateParticles();
            
            // Smooth camera rotation
            if (particleSystem) {
                particleSystem.rotation.y += (targetRotationY - particleSystem.rotation.y) * 0.05;
                particleSystem.rotation.x += (targetRotationX - particleSystem.rotation.x) * 0.05;
            }
            
            // Update FPS
            updateFPS();
            
            // Render
            renderer.render(scene, camera);
        }
        
        // Initialize the application
        init();
    </script>
</body>
</html>
