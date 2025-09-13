<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WebXR Portfolio Experience</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background: linear-gradient(135deg, #0a0a23, #1a1a4a, #2d2d5f);
            font-family: 'Arial', sans-serif;
            color: white;
            overflow: hidden;
        }
        
        #container {
            position: relative;
            width: 100vw;
            height: 100vh;
        }
        
        #hud {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 1000;
            background: rgba(0, 0, 0, 0.8);
            padding: 20px;
            border-radius: 15px;
            backdrop-filter: blur(10px);
            border: 2px solid #00ff88;
            box-shadow: 0 0 30px rgba(0, 255, 136, 0.3);
            max-width: 350px;
        }
        
        #hud h1 {
            margin: 0 0 15px 0;
            font-size: 20px;
            background: linear-gradient(45deg, #00ff88, #44ffaa);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            text-align: center;
        }
        
        .hud-section {
            margin: 15px 0;
            padding: 10px;
            background: rgba(0, 255, 136, 0.1);
            border-radius: 8px;
        }
        
        .hud-title {
            color: #00ff88;
            font-weight: bold;
            margin-bottom: 8px;
            font-size: 14px;
        }
        
        .hud-text {
            color: #ccffcc;
            font-size: 12px;
            line-height: 1.4;
        }
        
        .controls {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 15px;
        }
        
        button {
            background: linear-gradient(45deg, #003d20, #00ff88);
            border: none;
            color: white;
            padding: 10px 15px;
            border-radius: 8px;
            cursor: pointer;
            font-family: 'Arial', sans-serif;
            font-weight: bold;
            font-size: 12px;
            transition: all 0.3s ease;
        }
        
        button:hover {
            background: linear-gradient(45deg, #00ff88, #44ffaa);
            box-shadow: 0 0 20px rgba(0, 255, 136, 0.5);
            transform: translateY(-2px);
        }
        
        button:disabled {
            background: #333;
            cursor: not-allowed;
            color: #666;
        }
        
        #loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            z-index: 500;
            color: #00ff88;
            font-size: 18px;
        }
        
        .loading-spinner {
            width: 50px;
            height: 50px;
            border: 4px solid rgba(0, 255, 136, 0.3);
            border-top: 4px solid #00ff88;
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        #projectInfo {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0, 0, 0, 0.9);
            padding: 15px 25px;
            border-radius: 10px;
            border: 1px solid #00ff88;
            color: #00ff88;
            text-align: center;
            display: none;
            z-index: 1000;
        }
        
        @media (max-width: 768px) {
            #hud {
                left: 10px;
                right: 10px;
                top: 10px;
                max-width: none;
            }
        }
    </style>
</head>
<body>
    <div id="container">
        <div id="loading">
            <div class="loading-spinner"></div>
            <div>Loading WebXR Portfolio...</div>
            <div style="font-size: 12px; margin-top: 10px; opacity: 0.8;">
                Preparing immersive experience...
            </div>
        </div>
        
        <div id="hud">
            <h1>🌌 WebXR Portfolio</h1>
            
            <div class="hud-section">
                <div class="hud-title">🎮 CONTROLS</div>
                <div class="hud-text">
                    <strong>Desktop:</strong> WASD to move, Mouse to look<br>
                    <strong>VR:</strong> Use controllers to teleport<br>
                    <strong>Mobile:</strong> Touch to move, drag to look
                </div>
            </div>
            
            <div class="hud-section">
                <div class="hud-title">🎯 MISSION</div>
                <div class="hud-text">
                    Walk around the virtual gallery and approach the glowing project portals. 
                    Click or point at them to launch projects!
                </div>
            </div>
            
            <div class="hud-section">
                <div class="hud-title">📍 STATUS</div>
                <div class="hud-text">
                    Position: <span id="position">Loading...</span><br>
                    Mode: <span id="mode">Desktop</span><br>
                    Projects: <span id="projectCount">6</span> available
                </div>
            </div>
            
            <div class="controls">
                <button id="vrButton">Enter VR</button>
                <button id="resetPos">Reset Position</button>
                <button onclick="toggleHUD()">Hide HUD</button>
                <button onclick="toggleAudio()">🔊 Audio</button>
            </div>
        </div>
        
        <div id="projectInfo">
            <h3 id="projectTitle"></h3>
            <p id="projectDesc"></p>
            <div style="margin-top: 10px;">
                <em>Click to launch project</em>
            </div>
        </div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        let scene, camera, renderer, controls;
        let player = { position: new THREE.Vector3(0, 1.6, 5), rotation: new THREE.Euler() };
        let keys = { w: false, a: false, s: false, d: false };
        let mouse = { x: 0, y: 0, sensitivity: 0.002 };
        let isPointerLocked = false;
        let projectPortals = [];
        let currentProject = null;
        let audioEnabled = true;
        let hudVisible = true;
        
        // Project definitions
        const projects = [
            {
                id: 'particle-simulator',
                title: 'Particle Simulator',
                description: 'Interactive 3D particle system with real-time physics',
                url: 'https://yourusername.github.io/particle-simulator/',
                position: { x: -8, y: 0, z: -3 },
                color: 0x4ecdc4,
                icon: '🌌'
            },
            {
                id: 'orbital-calc',
                title: 'Orbital Calculator',
                description: 'Spacecraft trajectory simulation with gravitational physics',
                url: 'https://yourusername.github.io/orbital-calculator/',
                position: { x: 8, y: 0, z: -3 },
                color: 0xff6b6b,
                icon: '🚀'
            },
            {
                id: '3d-text',
                title: '3D Text Demo',
                description: 'Rotating 3D text with interactive controls',
                url: 'https://yourusername.github.io/3d-text/',
                position: { x: -8, y: 0, z: 3 },
                color: 0x45b7d1,
                icon: '✨'
            },
            {
                id: 'vr-gallery',
                title: 'VR Gallery',
                description: 'Virtual reality art gallery experience',
                url: 'https://yourusername.github.io/vr-gallery/',
                position: { x: 8, y: 0, z: 3 },
                color: 0x96ceb4,
                icon: '🖼️'
            },
            {
                id: 'physics-sim',
                title: 'Physics Playground',
                description: 'Interactive physics simulation with real-time dynamics',
                url: 'https://yourusername.github.io/physics-playground/',
                position: { x: 0, y: 0, z: -8 },
                color: 0xfeca57,
                icon: '⚽'
            },
            {
                id: 'space-explorer',
                title: 'Space Explorer',
                description: 'Navigate through procedural solar systems',
                url: 'https://yourusername.github.io/space-explorer/',
                position: { x: 0, y: 0, z: 8 },
                color: 0xff9ff3,
                icon: '🌟'
            }
        ];
        
        function init() {
            // Create scene
            scene = new THREE.Scene();
            scene.fog = new THREE.Fog(0x0a0a23, 10, 100);
            
            // Create camera
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.copy(player.position);
            
            // Create renderer
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setClearColor(0x0a0a23);
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            renderer.xr.enabled = true;
            document.getElementById('container').appendChild(renderer.domElement);
            
            // Setup lighting
            setupLighting();
            
            // Create environment
            createEnvironment();
            
            // Create project portals
            createProjectPortals();
            
            // Setup controls
            setupControls();
            
            // Setup WebXR
            setupWebXR();
            
            // Hide loading screen
            document.getElementById('loading').style.display = 'none';
            
            // Start render loop
            animate();
        }
        
        function setupLighting() {
            // Ambient light
            const ambientLight = new THREE.AmbientLight(0x404080, 0.4);
            scene.add(ambientLight);
            
            // Main directional light
            const directionalLight = new THREE.DirectionalLight(0xffffff, 0.6);
            directionalLight.position.set(10, 20, 10);
            directionalLight.castShadow = true;
            directionalLight.shadow.mapSize.width = 2048;
            directionalLight.shadow.mapSize.height = 2048;
            scene.add(directionalLight);
            
            // Colored accent lights
            const light1 = new THREE.PointLight(0x00ff88, 0.8, 20);
            light1.position.set(-10, 5, -10);
            scene.add(light1);
            
            const light2 = new THREE.PointLight(0xff6b6b, 0.8, 20);
            light2.position.set(10, 5, 10);
            scene.add(light2);
        }
        
        function createEnvironment() {
            // Create floor
            const floorGeometry = new THREE.PlaneGeometry(50, 50);
            const floorMaterial = new THREE.MeshLambertMaterial({ 
                color: 0x1a1a4a,
                transparent: true,
                opacity: 0.8
            });
            const floor = new THREE.Mesh(floorGeometry, floorMaterial);
            floor.rotation.x = -Math.PI / 2;
            floor.receiveShadow = true;
            scene.add(floor);
            
            // Create walls
            const wallMaterial = new THREE.MeshLambertMaterial({ 
                color: 0x2d2d5f,
                transparent: true,
                opacity: 0.6
            });
            
            // Back wall
            const backWall = new THREE.Mesh(new THREE.PlaneGeometry(50, 20), wallMaterial);
            backWall.position.set(0, 10, -25);
            scene.add(backWall);
            
            // Side walls
            const leftWall = new THREE.Mesh(new THREE.PlaneGeometry(50, 20), wallMaterial);
            leftWall.position.set(-25, 10, 0);
            leftWall.rotation.y = Math.PI / 2;
            scene.add(leftWall);
            
            const rightWall = new THREE.Mesh(new THREE.PlaneGeometry(50, 20), wallMaterial);
            rightWall.position.set(25, 10, 0);
            rightWall.rotation.y = -Math.PI / 2;
            scene.add(rightWall);
            
            // Add floating particles
            createFloatingParticles();
        }
        
        function createFloatingParticles() {
            const particleCount = 500;
            const particles = new THREE.BufferGeometry();
            const positions = new Float32Array(particleCount * 3);
            
            for (let i = 0; i < particleCount; i++) {
                positions[i * 3] = (Math.random() - 0.5) * 100;
                positions[i * 3 + 1] = Math.random() * 20;
                positions[i * 3 + 2] = (Math.random() - 0.5) * 100;
            }
            
            particles.setAttribute('position', new THREE.BufferAttribute(positions, 3));
            
            const particleMaterial = new THREE.PointsMaterial({
                color: 0x00ff88,
                size: 2,
                transparent: true,
                opacity: 0.6,
                blending: THREE.AdditiveBlending
            });
            
            const particleSystem = new THREE.Points(particles, particleMaterial);
            scene.add(particleSystem);
        }
        
        function createProjectPortals() {
            projects.forEach((project, index) => {
                // Portal base
                const portalGeometry = new THREE.CylinderGeometry(2, 2, 0.2, 16);
                const portalMaterial = new THREE.MeshLambertMaterial({ 
                    color: project.color,
                    emissive: project.color,
                    emissiveIntensity: 0.3
                });
                const portal = new THREE.Mesh(portalGeometry, portalMaterial);
                portal.position.set(project.position.x, 0.1, project.position.z);
                portal.castShadow = true;
                
                // Portal glow effect
                const glowGeometry = new THREE.CylinderGeometry(2.5, 2.5, 0.1, 16);
                const glowMaterial = new THREE.MeshBasicMaterial({
                    color: project.color,
                    transparent: true,
                    opacity: 0.3,
                    blending: THREE.AdditiveBlending
                });
                const glow = new THREE.Mesh(glowGeometry, glowMaterial);
                glow.position.copy(portal.position);
                glow.position.y = 0.05;
                
                // Floating hologram
                const textGeometry = new THREE.BoxGeometry(1, 0.5, 0.1);
                const textMaterial = new THREE.MeshLambertMaterial({
                    color: 0xffffff,
                    emissive: 0x444444
                });
                const textMesh = new THREE.Mesh(textGeometry, textMaterial);
                textMesh.position.set(project.position.x, 2, project.position.z);
                
                // Add floating animation
                textMesh.userData = {
                    originalY: 2,
                    floatSpeed: 0.002 + index * 0.0003
                };
                
                // Store project data
                portal.userData = project;
                textMesh.userData.project = project;
                
                scene.add(portal);
                scene.add(glow);
                scene.add(textMesh);
                
                projectPortals.push({ portal, glow, textMesh, project });
            });
        }
        
        function setupControls() {
            // Keyboard controls
            document.addEventListener('keydown', (e) => {
                switch(e.code) {
                    case 'KeyW': keys.w = true; break;
                    case 'KeyA': keys.a = true; break;
                    case 'KeyS': keys.s = true; break;
                    case 'KeyD': keys.d = true; break;
                }
            });
            
            document.addEventListener('keyup', (e) => {
                switch(e.code) {
                    case 'KeyW': keys.w = false; break;
                    case 'KeyA': keys.a = false; break;
                    case 'KeyS': keys.s = false; break;
                    case 'KeyD': keys.d = false; break;
                }
            });
            
            // Mouse controls
            document.addEventListener('click', () => {
                if (!isPointerLocked) {
                    renderer.domElement.requestPointerLock();
                }
            });
            
            document.addEventListener('pointerlockchange', () => {
                isPointerLocked = document.pointerLockElement === renderer.domElement;
            });
            
            document.addEventListener('mousemove', (e) => {
                if (isPointerLocked) {
                    mouse.x += e.movementX * mouse.sensitivity;
                    mouse.y += e.movementY * mouse.sensitivity;
                    mouse.y = Math.max(-Math.PI/2, Math.min(Math.PI/2, mouse.y));
                }
            });
            
            // Click detection for project portals
            const raycaster = new THREE.Raycaster();
            const mouseVector = new THREE.Vector2();
            
            renderer.domElement.addEventListener('click', (e) => {
                if (isPointerLocked) {
                    mouseVector.x = 0; // Center of screen in pointer lock
                    mouseVector.y = 0;
                    
                    raycaster.setFromCamera(mouseVector, camera);
                    const intersects = raycaster.intersectObjects(projectPortals.map(p => p.portal));
                    
                    if (intersects.length > 0) {
                        const project = intersects[0].object.userData;
                        launchProject(project);
                    }
                }
            });
            
            // Reset position button
            document.getElementById('resetPos').addEventListener('click', () => {
                player.position.set(0, 1.6, 5);
                mouse.x = 0;
                mouse.y = 0;
            });
        }
        
        function setupWebXR() {
            // Check for WebXR support
            if ('xr' in navigator) {
                navigator.xr.isSessionSupported('immersive-vr').then((supported) => {
                    const vrButton = document.getElementById('vrButton');
                    if (supported) {
                        vrButton.addEventListener('click', () => {
                            if (renderer.xr.isPresenting) {
                                renderer.xr.getSession().end();
                            } else {
                                navigator.xr.requestSession('immersive-vr').then((session) => {
                                    renderer.xr.setSession(session);
                                    document.getElementById('mode').textContent = 'VR';
                                });
                            }
                        });
                    } else {
                        vrButton.disabled = true;
                        vrButton.textContent = 'VR Not Available';
                    }
                });
            } else {
                document.getElementById('vrButton').disabled = true;
                document.getElementById('vrButton').textContent = 'WebXR Not Supported';
            }
        }
        
        function updateMovement() {
            if (renderer.xr.isPresenting) return; // Skip movement in VR mode
            
            const moveSpeed = 0.1;
            const direction = new THREE.Vector3();
            
            if (keys.w) direction.z -= 1;
            if (keys.s) direction.z += 1;
            if (keys.a) direction.x -= 1;
            if (keys.d) direction.x += 1;
            
            if (direction.length() > 0) {
                direction.normalize();
                direction.multiplyScalar(moveSpeed);
                
                // Apply rotation to movement
                const euler = new THREE.Euler(0, mouse.x, 0, 'YXZ');
                direction.applyEuler(euler);
                
                player.position.add(direction);
            }
            
            // Update camera
            camera.position.copy(player.position);
            camera.rotation.set(mouse.y, mouse.x, 0);
            
            // Update position display
            const pos = player.position;
            document.getElementById('position').textContent = 
                `${pos.x.toFixed(1)}, ${pos.y.toFixed(1)}, ${pos.z.toFixed(1)}`;
        }
        
        function updateProjectPortals() {
            const time = Date.now() * 0.001;
            
            projectPortals.forEach(({portal, glow, textMesh}) => {
                // Floating animation
                textMesh.position.y = textMesh.userData.originalY + Math.sin(time * textMesh.userData.floatSpeed * 1000) * 0.2;
                
                // Rotation animation
                portal.rotation.y += 0.01;
                textMesh.rotation.y += 0.02;
                
                // Pulsing glow
                glow.material.opacity = 0.2 + Math.sin(time * 2) * 0.1;
                
                // Check distance to player for info display
                const distance = player.position.distanceTo(portal.position);
                if (distance < 4 && currentProject !== textMesh.userData.project) {
                    showProjectInfo(textMesh.userData.project);
                } else if (distance >= 4 && currentProject === textMesh.userData.project) {
                    hideProjectInfo();
                }
            });
        }
        
        function showProjectInfo(project) {
            currentProject = project;
            document.getElementById('projectTitle').textContent = project.icon + ' ' + project.title;
            document.getElementById('projectDesc').textContent = project.description;
            document.getElementById('projectInfo').style.display = 'block';
        }
        
        function hideProjectInfo() {
            currentProject = null;
            document.getElementById('projectInfo').style.display = 'none';
        }
        
        function launchProject(project) {
            // Visual feedback
            const portal = projectPortals.find(p => p.project.id === project.id);
            if (portal) {
                portal.glow.material.opacity = 1;
                setTimeout(() => {
                    portal.glow.material.opacity = 0.3;
                }, 300);
            }
            
            // Launch project
            if (project.url.includes('yourusername')) {
                alert(`🚀 Launching: ${project.title}\n\n💡 Demo Mode: Replace the URL in the projects array with your actual project link!\n\nExample: ${project.url.replace('yourusername', 'your-github-username')}`);
            } else {
                window.open(project.url, '_blank');
            }
        }
        
        function toggleHUD() {
            hudVisible = !hudVisible;
            document.getElementById('hud').style.display = hudVisible ? 'block' : 'none';
            event.target.textContent = hudVisible ? 'Hide HUD' : 'Show HUD';
        }
        
        function toggleAudio() {
            audioEnabled = !audioEnabled;
            event.target.textContent = audioEnabled ? '🔊 Audio' : '🔇 Audio';
        }
        
        function animate() {
            renderer.setAnimationLoop(() => {
                updateMovement();
                updateProjectPortals();
                renderer.render(scene, camera);
            });
        }
        
        // Handle window resize
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
        
        // Initialize the experience
        init();
    </script>
</body>
</html>
