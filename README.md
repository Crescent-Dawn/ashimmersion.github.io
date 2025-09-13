<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive 3D Portfolio</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            line-height: 1.6;
            color: white;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            position: relative;
            overflow-x: hidden;
        }
        
        body::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: 
                radial-gradient(circle at 20% 50%, rgba(120, 119, 198, 0.3) 0%, transparent 50%),
                radial-gradient(circle at 80% 20%, rgba(255, 119, 198, 0.3) 0%, transparent 50%),
                radial-gradient(circle at 40% 80%, rgba(120, 219, 255, 0.3) 0%, transparent 50%);
            pointer-events: none;
            animation: backgroundShift 15s ease-in-out infinite;
        }
        
        @keyframes backgroundShift {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.7; }
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 0 20px;
            display: flex;
            flex-direction: column;
            min-height: 100vh;
            position: relative;
            z-index: 1;
        }
        
        /* Header Section */
        .header-section {
            flex: 1;
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
            padding: 60px 0;
        }
        
        .content-block {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            border-radius: 20px;
            padding: 60px 40px;
            box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
            max-width: 800px;
            animation: fadeInUp 1s ease-out;
        }
        
        @keyframes fadeInUp {
            from {
                opacity: 0;
                transform: translateY(50px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        
        .content-block h1 {
            font-size: 3.5rem;
            font-weight: 700;
            margin-bottom: 20px;
            background: linear-gradient(45deg, #fff, #f0f8ff, #e6f3ff);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
        }
        
        .content-block p {
            font-size: 1.3rem;
            line-height: 1.8;
            margin-bottom: 25px;
            opacity: 0.95;
            text-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
        }
        
        .highlight {
            background: linear-gradient(45deg, rgba(255, 255, 255, 0.2), rgba(255, 255, 255, 0.1));
            padding: 2px 8px;
            border-radius: 6px;
            font-weight: 600;
        }
        
        /* Projects Section */
        .projects-section {
            padding: 40px 0 60px 0;
            animation: fadeInUp 1s ease-out 0.3s both;
        }
        
        .projects-section h2 {
            text-align: center;
            font-size: 2.5rem;
            margin-bottom: 40px;
            background: linear-gradient(45deg, #fff, #f0f8ff);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
        }
        
        .projects-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 25px;
            max-width: 1000px;
            margin: 0 auto;
        }
        
        .project-button {
            background: rgba(255, 255, 255, 0.15);
            backdrop-filter: blur(15px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            border-radius: 16px;
            padding: 30px 25px;
            color: white;
            text-decoration: none;
            display: block;
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            cursor: pointer;
            position: relative;
            overflow: hidden;
        }
        
        .project-button::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.1), transparent);
            transition: left 0.5s;
        }
        
        .project-button:hover::before {
            left: 100%;
        }
        
        .project-button:hover {
            transform: translateY(-8px) scale(1.02);
            background: rgba(255, 255, 255, 0.25);
            border-color: rgba(255, 255, 255, 0.4);
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
        }
        
        .project-button:active {
            transform: translateY(-4px) scale(1.01);
        }
        
        .project-icon {
            font-size: 3rem;
            display: block;
            margin-bottom: 15px;
            text-align: center;
        }
        
        .project-title {
            font-size: 1.4rem;
            font-weight: 600;
            margin-bottom: 10px;
            text-align: center;
        }
        
        .project-description {
            font-size: 1rem;
            opacity: 0.9;
            text-align: center;
            line-height: 1.5;
        }
        
        /* Responsive Design */
        @media (max-width: 768px) {
            .container {
                padding: 0 15px;
            }
            
            .content-block {
                padding: 40px 25px;
                margin: 20px 0;
            }
            
            .content-block h1 {
                font-size: 2.5rem;
            }
            
            .content-block p {
                font-size: 1.1rem;
            }
            
            .projects-section h2 {
                font-size: 2rem;
            }
            
            .projects-grid {
                grid-template-columns: 1fr;
                gap: 20px;
            }
            
            .project-button {
                padding: 25px 20px;
            }
        }
        
        @media (max-width: 480px) {
            .content-block h1 {
                font-size: 2rem;
            }
            
            .content-block p {
                font-size: 1rem;
            }
        }
        
        /* Floating particles animation */
        .particle {
            position: absolute;
            width: 4px;
            height: 4px;
            background: rgba(255, 255, 255, 0.6);
            border-radius: 50%;
            pointer-events: none;
            animation: float 15s infinite linear;
        }
        
        @keyframes float {
            0% {
                transform: translateY(100vh) scale(0);
                opacity: 0;
            }
            10% {
                opacity: 1;
            }
            90% {
                opacity: 1;
            }
            100% {
                transform: translateY(-100px) scale(1);
                opacity: 0;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Header Section with Text -->
        <section class="header-section">
            <div class="content-block">
                <h1>Hello! Welcome to My Interactive World</h1>
                <p>
                    Here's all of my <span class="highlight">really cool things!</span> built with modern web technologies. 
                    From mesmerizing particle systems to immersive WebXR environments, each project showcases the power of 
                    <span class="highlight">interactive web development</span>.
                </p>
                <p>
                    Dive into worlds where physics comes alive, particles dance to your commands, and virtual reality 
                    meets the web browser. Every experience is crafted with <span class="highlight">webxr, WebGL, and creative coding!</span> 
                    to push the boundaries of what's possible in a web browser.
                </p>
            </div>
        </section>
        
        <!-- Projects Section -->
        <section class="projects-section">
            <h2>🚀 Explore My Projects</h2>
            <div class="projects-grid">
                <button class="project-button" onclick="openProject('Orbital Trajectory Simulator')">
                    <span class="project-icon">🌌</span>
                    <div class="project-title">Orbital Trajectory Simulator</div>
                    <div class="project-description">This thingy shows how to make things into orbit!</div>
                </button>
                
                <button class="project-button" onclick="openProject('3d-text')">
                    <span class="project-icon">✨</span>
                    <div class="project-title">3D Text Experience</div>
                    <div class="project-description">Rotating 3D text with mouse controls and dynamic lighting effects</div>
                </button>
                
                <button class="project-button" onclick="openProject('vr-gallery')">
                    <span class="project-icon">🖼️</span>
                    <div class="project-title">VR Art Gallery</div>
                    <div class="project-description">Virtual reality gallery experience showcasing digital art in immersive environments</div>
                </button>
                
                <button class="project-button" onclick="openProject('ar-viewer')">
                    <span class="project-icon">📱</span>
                    <div class="project-title">AR Model Viewer</div>
                    <div class="project-description">Augmented reality 3D model viewer using WebXR for mobile devices</div>
                </button>
                
                <button class="project-button" onclick="openProject('physics-playground')">
                    <span class="project-icon">⚽</span>
                    <div class="project-title">Physics Playground</div>
                    <div class="project-description">Interactive physics simulation with gravity, collisions, and real-time dynamics</div>
                </button>
                
                <button class="project-button" onclick="openProject('space-explorer')">
                    <span class="project-icon">🚀</span>
                    <div class="project-title">Space Explorer</div>
                    <div class="project-description">Navigate through a procedural solar system with realistic celestial mechanics</div>
                </button>
            </div>
        </section>
    </div>

    <script>
        // Project URLs - Replace with your actual GitHub Pages URLs
        const projects = {
            'particle-simulator': 'https://crescent-dawn.github.io/orbitaltrajcalc1.github.io/',
            '3d-text': 'https://yourusername.github.io/3d-text/',
            'vr-gallery': 'https://yourusername.github.io/vr-gallery/',
            'ar-viewer': 'https://yourusername.github.io/ar-viewer/',
            'physics-playground': 'https://yourusername.github.io/physics-playground/',
            'space-explorer': 'https://yourusername.github.io/space-explorer/'
        };
        
        function openProject(projectId) {
            const url = projects[projectId];
            if (url && url !== 'https://yourusername.github.io/' + projectId + '/') {
                // Add smooth transition effect
                document.body.style.transition = 'all 0.3s ease';
                document.body.style.transform = 'scale(0.98)';
                document.body.style.filter = 'brightness(0.8)';
                
                setTimeout(() => {
                    window.open(url, '_blank'); // Opens in new tab
                    // Or use: window.location.href = url; // Opens in same tab
                    
                    // Reset transition
                    document.body.style.transform = 'scale(1)';
                    document.body.style.filter = 'brightness(1)';
                }, 200);
            } else {
                // Demo mode - shows what project would open
                alert(`🚀 Opening: ${projectId}\n\n💡 To connect this button to your actual project:\n\n1. Replace 'yourusername' with your GitHub username\n2. Make sure your project repository exists\n3. Ensure GitHub Pages is enabled for that repo\n\nExample URL: https://yourusername.github.io/${projectId}/`);
            }
        }
        
        // Create floating particles animation
        function createParticles() {
            const particleCount = 30;
            
            for (let i = 0; i < particleCount; i++) {
                setTimeout(() => {
                    const particle = document.createElement('div');
                    particle.className = 'particle';
                    particle.style.left = Math.random() * 100 + '%';
                    particle.style.animationDelay = Math.random() * 15 + 's';
                    particle.style.animationDuration = (Math.random() * 10 + 10) + 's';
                    document.body.appendChild(particle);
                    
                    // Remove particle after animation
                    setTimeout(() => {
                        if (particle.parentNode) {
                            particle.parentNode.removeChild(particle);
                        }
                    }, 25000);
                }, i * 500);
            }
        }
        
        // Start particle animation
        createParticles();
        
        // Recreate particles periodically
        setInterval(createParticles, 15000);
        
        // Add smooth scrolling for better UX
        document.querySelectorAll('a[href^="#"]').forEach(anchor => {
            anchor.addEventListener('click', function (e) {
                e.preventDefault();
                document.querySelector(this.getAttribute('href')).scrollIntoView({
                    behavior: 'smooth'
                });
            });
        });
    </script>
</body>
</html>
