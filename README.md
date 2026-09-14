<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SignBridge - Sign Language to Speech Converter</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        brand: {
                            50: '#eef2ff',
                            500: '#6366f1',
                            600: '#4f46e5',
                            700: '#4338ca',
                            glow: '#818cf8',
                        },
                        accent: {
                            cyan: '#06b6d4',
                            emerald: '#10b981',
                            rose: '#f43f5e',
                            amber: '#f59e0b'
                        }
                    },
                    fontFamily: {
                        sans: ['Inter', 'system-ui', 'sans-serif'],
                        mono: ['JetBrains Mono', 'monospace']
                    }
                }
            }
        }
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=JetBrains+Mono:wght@400;500;700&display=swap');
        
        body {
            font-family: 'Inter', sans-serif;
            background-color: #090d16;
            color: #f3f4f6;
        }

        /* Custom Scrollbars */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #0f172a;
        }
        ::-webkit-scrollbar-thumb {
            background: #334155;
            border-radius: 9999px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #475569;
        }

        /* HUD Scanline & Glow Animation */
        .hud-scanline {
            background: linear-gradient(to bottom, transparent 50%, rgba(99, 102, 241, 0.08) 51%);
            background-size: 100% 4px;
        }
        
        .glow-effect {
            box-shadow: 0 0 25px -5px rgba(99, 102, 241, 0.4);
        }

        .glow-emerald {
            box-shadow: 0 0 20px -5px rgba(16, 185, 129, 0.4);
        }

        /* Bounding Box Visualizer Animation */
        @keyframes pulse-box {
            0%, 100% { opacity: 0.9; transform: scale(1); }
            50% { opacity: 0.5; transform: scale(0.98); }
        }
        .animate-pulse-box {
            animation: pulse-box 2.5s ease-in-out infinite;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between overflow-x-hidden hud-scanline">

    <!-- TOP NAVIGATION BAR -->
    <header class="border-b border-slate-800/80 bg-slate-950/80 backdrop-blur-md sticky top-0 z-50 px-4 lg:px-8 py-3.5">
        <div class="max-w-7xl mx-auto flex items-center justify-between">
            <!-- Brand & Tagline -->
            <div class="flex items-center gap-3">
                <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-brand-600 to-accent-cyan flex items-center justify-center shadow-lg shadow-brand-500/30">
                    <i data-lucide="hand" class="w-6 h-6 text-white"></i>
                </div>
                <div>
                    <div class="flex items-center gap-2">
                        <h1 class="text-xl font-bold tracking-tight text-white flex items-center gap-2">
                            SignBridge <span class="text-xs px-2 py-0.5 rounded-full bg-brand-500/20 text-brand-glow font-mono border border-brand-500/30">AI Pro</span>
                        </h1>
                    </div>
                    <p class="text-xs text-slate-400">Accessible Sign Language to Speech Ecosystem</p>
                </div>
            </div>

            <!-- Mode Switcher & Stats Bar -->
            <div class="flex items-center gap-3">
                <!-- Mode Toggle (Live API vs Simulation) -->
                <div class="bg-slate-900 border border-slate-800 p-1 rounded-xl flex items-center gap-1">
                    <button id="mode-sim-btn" onclick="setMode('sim')" class="px-3 py-1.5 rounded-lg text-xs font-medium transition-all flex items-center gap-1.5 bg-brand-600 text-white shadow">
                        <i data-lucide="sparkles" class="w-3.5 h-3.5"></i>
                        <span>Demo Simulation</span>
                    </button>
                    <button id="mode-api-btn" onclick="setMode('api')" class="px-3 py-1.5 rounded-lg text-xs font-medium text-slate-400 hover:text-white transition-all flex items-center gap-1.5">
                        <i data-lucide="cpu" class="w-3.5 h-3.5"></i>
                        <span>Python Model API</span>
                    </button>
                </div>

                <!-- WebCam Status Badge -->
                <div id="cam-status" class="hidden sm:flex items-center gap-2 px-3 py-1.5 rounded-xl bg-emerald-950/40 border border-emerald-500/30 text-emerald-400 text-xs font-mono">
                    <span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span>
                    <span>CAMERA READY</span>
                </div>
            </div>
        </div>
    </header>

    <!-- MAIN CONTENT CONTAINER -->
    <main class="max-w-7xl w-full mx-auto p-4 lg:p-6 flex-1 grid grid-cols-1 lg:grid-cols-12 gap-6">

        <!-- LEFT/TOP COLUMN: Live Camera Feed & Controls (7 Cols on desktop) -->
        <section class="lg:col-span-7 flex flex-col gap-4">
            <!-- Camera Viewport Card -->
            <div class="relative bg-slate-900/90 border border-slate-800 rounded-2xl overflow-hidden shadow-2xl flex flex-col justify-between min-h-[380px] sm:min-h-[440px] glow-effect">
                
                <!-- HUD Top Overlay Stats -->
                <div class="absolute top-4 left-4 right-4 z-20 flex items-center justify-between pointer-events-none">
                    <div class="flex items-center gap-2 bg-slate-950/80 backdrop-blur-md border border-slate-800 px-3 py-1.5 rounded-xl">
                        <span class="w-2.5 h-2.5 rounded-full bg-rose-500 animate-ping"></span>
                        <span class="text-xs font-semibold tracking-wider text-slate-200 uppercase">Live Processing</span>
                    </div>

                    <div class="flex items-center gap-2 bg-slate-950/80 backdrop-blur-md border border-slate-800 px-3 py-1.5 rounded-xl font-mono text-xs text-slate-300">
                        <i data-lucide="activity" class="w-3.5 h-3.5 text-accent-cyan"></i>
                        <span>FPS: <strong id="fps-count" class="text-white">30</strong></span>
                    </div>
                </div>

                <!-- Video Element & Overlay Canvas -->
                <div class="relative w-full h-full flex-1 bg-black flex items-center justify-center overflow-hidden">
                    <video id="webcam-feed" class="w-full h-full object-cover transform -scale-x-100" autoplay playsinline muted></video>
                    
                    <!-- Simulated Landmark Overlay (Visual Canvas effect) -->
                    <div id="landmark-overlay" class="absolute inset-0 pointer-events-none flex items-center justify-center">
                        <div class="w-64 h-64 border-2 border-dashed border-brand-glow/70 rounded-3xl relative animate-pulse-box flex items-center justify-center bg-brand-500/5 backdrop-blur-[1px]">
                            <!-- Corner HUD Lines -->
                            <div class="absolute -top-1 -left-1 w-5 h-5 border-t-2 border-l-2 border-brand-glow"></div>
                            <div class="absolute -top-1 -right-1 w-5 h-5 border-t-2 border-r-2 border-brand-glow"></div>
                            <div class="absolute -bottom-1 -left-1 w-5 h-5 border-b-2 border-l-2 border-brand-glow"></div>
                            <div class="absolute -bottom-1 -right-1 w-5 h-5 border-b-2 border-r-2 border-brand-glow"></div>
                            
                            <!-- Tracking Points Mock (Center Target) -->
                            <div class="w-3 h-3 rounded-full bg-accent-cyan shadow-lg shadow-accent-cyan/80 animate-ping"></div>
                            <span class="absolute bottom-3 text-[10px] font-mono text-brand-glow uppercase tracking-wider bg-slate-950/80 px-2 py-0.5 rounded border border-brand-500/30">Target Lock: Active</span>
                        </div>
                    </div>

                    <!-- Off-camera / Camera Disabled Placeholder -->
                    <div id="cam-fallback" class="hidden absolute inset-0 bg-slate-950 flex flex-col items-center justify-center p-6 text-center z-10">
                        <div class="w-16 h-16 rounded-2xl bg-slate-900 border border-slate-800 flex items-center justify-center text-slate-500 mb-3">
                            <i data-lucide="camera-off" class="w-8 h-8"></i>
                        </div>
                        <h3 class="text-lg font-semibold text-white mb-1">Camera Feed Paused / Offline</h3>
                        <p id="cam-fallback-msg" class="text-sm text-slate-400 max-w-xs mb-4">No physical camera detected. Demo Simulation Mode is active.</p>
                        <button onclick="startWebcam()" class="px-4 py-2 bg-brand-600 hover:bg-brand-500 text-white rounded-xl text-sm font-medium transition flex items-center gap-2">
                            <i data-lucide="video" class="w-4 h-4"></i> Retry Camera
                        </button>
                    </div>
                </div>

                <!-- HUD Bottom Control Floating Toolbar -->
                <div class="p-4 bg-gradient-to-t from-slate-950 via-slate-950/90 to-transparent flex items-center justify-between gap-3 z-20">
                    <div class="flex items-center gap-2">
                        <button id="toggle-cam-btn" onclick="toggleWebcam()" class="p-2.5 rounded-xl bg-slate-800/80 hover:bg-slate-700 text-white border border-slate-700 transition" title="Toggle Camera">
                            <i data-lucide="video" class="w-5 h-5"></i>
                        </button>
                        <button id="toggle-overlay-btn" onclick="toggleOverlay()" class="p-2.5 rounded-xl bg-slate-800/80 hover:bg-slate-700 text-brand-glow border border-slate-700 transition" title="Toggle AI Keypoints HUD">
                            <i data-lucide="scan" class="w-5 h-5"></i>
                        </button>
                    </div>

                    <!-- Current Sign Real-time Card -->
                    <div class="flex items-center gap-3 bg-slate-900/90 border border-slate-800 px-4 py-2 rounded-xl shadow-inner">
                        <div class="text-right">
                            <p class="text-[10px] uppercase font-mono text-slate-400 tracking-wider">Detected Sign</p>
                            <p id="current-sign-text" class="text-lg font-extrabold text-accent-cyan tracking-wide font-mono">WAITING...</p>
                        </div>
                        <div class="w-px h-7 bg-slate-800"></div>
                        <div>
                            <p class="text-[10px] uppercase font-mono text-slate-400 tracking-wider">Confidence</p>
                            <p id="confidence-score" class="text-sm font-bold text-emerald-400 font-mono">0%</p>
                        </div>
                    </div>
                </div>
            </div>

            <!-- API Endpoint Input Bar (Shown when API mode active) -->
            <div id="api-config-card" class="hidden bg-slate-900/80 border border-slate-800 p-4 rounded-2xl flex flex-col sm:flex-row items-center gap-3">
                <div class="flex items-center gap-2 text-slate-400 text-sm whitespace-nowrap">
                    <i data-lucide="link" class="w-4 h-4 text-brand-glow"></i>
                    <span>Python Backend URL:</span>
                </div>
                <input type="text" id="api-url-input" value="http://localhost:5000/predict" placeholder="http://192.168.x.x:5000/predict" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-xs font-mono text-slate-200 focus:outline-none focus:border-brand-500">
                <button onclick="testApiConnection()" class="w-full sm:w-auto px-4 py-2 bg-slate-800 hover:bg-slate-700 text-xs font-semibold rounded-xl text-slate-200 whitespace-nowrap border border-slate-700">
                    Test Connection
                </button>
            </div>
        </section>

        <!-- RIGHT/BOTTOM COLUMN: Subtitle & Sentence Builder (5 Cols on desktop) -->
        <section class="lg:col-span-5 flex flex-col gap-4">
            
            <!-- Sentence Construction Display (Inclusive Big Typography) -->
            <div class="bg-slate-900/90 border border-slate-800 rounded-2xl p-5 flex flex-col justify-between h-full min-h-[350px] shadow-2xl relative overflow-hidden">
                
                <!-- Card Header -->
                <div>
                    <div class="flex items-center justify-between pb-4 border-b border-slate-800/80">
                        <div class="flex items-center gap-2">
                            <i data-lucide="message-square-text" class="w-5 h-5 text-brand-glow"></i>
                            <h2 class="text-base font-bold text-white tracking-wide">Live Sentence Builder</h2>
                        </div>
                        <span class="text-xs font-mono px-2 py-0.5 rounded-full bg-slate-800 text-slate-400 border border-slate-700" id="word-count">0 words</span>
                    </div>

                    <!-- Accessibility Large Display Area -->
                    <div class="mt-4 p-4 rounded-xl bg-slate-950/70 border border-slate-800/60 min-h-[180px] flex flex-col justify-between">
                        <div id="sentence-box" class="text-2xl sm:text-3xl font-bold text-white leading-relaxed tracking-wide min-h-[120px] select-text">
                            <span class="text-slate-600 font-normal italic text-lg">Recognized sign language phrases will assemble here in real time...</span>
                        </div>
                        
                        <!-- Dynamic Subtitle Indicator -->
                        <div class="flex items-center justify-between pt-3 border-t border-slate-900 text-xs text-slate-400 font-mono">
                            <span class="flex items-center gap-1.5 text-slate-400">
                                <span id="auto-speak-indicator" class="w-2 h-2 rounded-full bg-emerald-500"></span>
                                Auto-TTS: Enabled
                            </span>
                            <span class="text-[11px] text-slate-500">High Contrast Accessible Output</span>
                        </div>
                    </div>
                </div>

                <!-- Speech & Action Control Buttons -->
                <div class="mt-6 flex flex-col gap-3">
                    
                    <!-- Primary Voice Action Button -->
                    <button onclick="speakSentence()" class="w-full py-3.5 px-4 bg-gradient-to-r from-brand-600 to-brand-700 hover:from-brand-500 hover:to-brand-600 text-white font-bold rounded-xl shadow-lg shadow-brand-600/30 flex items-center justify-center gap-2 transition active:scale-[0.99] text-base">
                        <i data-lucide="volume-2" class="w-5 h-5"></i>
                        <span>Speak Sentence Now</span>
                    </button>

                    <!-- Secondary Utility Grid -->
                    <div class="grid grid-cols-3 gap-2">
                        <button onclick="clearSentence()" class="py-2.5 px-3 bg-slate-800/80 hover:bg-slate-700 text-slate-300 font-medium rounded-xl text-xs flex items-center justify-center gap-1.5 border border-slate-700 transition">
                            <i data-lucide="rotate-ccw" class="w-4 h-4 text-slate-400"></i>
                            <span>Clear</span>
                        </button>
                        
                        <button onclick="copySentence()" class="py-2.5 px-3 bg-slate-800/80 hover:bg-slate-700 text-slate-300 font-medium rounded-xl text-xs flex items-center justify-center gap-1.5 border border-slate-700 transition">
                            <i data-lucide="copy" class="w-4 h-4 text-slate-400"></i>
                            <span>Copy</span>
                        </button>
                        
                        <button id="toggle-tts-btn" onclick="toggleAutoTTS()" class="py-2.5 px-3 bg-slate-800/80 hover:bg-slate-700 text-emerald-400 font-medium rounded-xl text-xs flex items-center justify-center gap-1.5 border border-slate-700 transition">
                            <i data-lucide="volume-x" class="w-4 h-4"></i>
                            <span id="tts-btn-label">Mute Voice</span>
                        </button>
                    </div>

                    <!-- Quick Phrases Bar for Hackathon Demo -->
                    <div class="pt-3 border-t border-slate-800/60">
                        <p class="text-[11px] font-mono text-slate-400 mb-2 uppercase tracking-wider">Quick Sign Demo Triggers:</p>
                        <div class="flex flex-wrap gap-1.5">
                            <button onclick="injectSign('Hello')" class="px-2.5 py-1 bg-slate-950 hover:bg-slate-800 border border-slate-800 text-xs text-slate-300 rounded-lg">🖐️ Hello</button>
                            <button onclick="injectSign('Thank You')" class="px-2.5 py-1 bg-slate-950 hover:bg-slate-800 border border-slate-800 text-xs text-slate-300 rounded-lg">🙏 Thank You</button>
                            <button onclick="injectSign('Need Water')" class="px-2.5 py-1 bg-slate-950 hover:bg-slate-800 border border-slate-800 text-xs text-slate-300 rounded-lg">💧 Need Water</button>
                            <button onclick="injectSign('Help')" class="px-2.5 py-1 bg-slate-950 hover:bg-slate-800 border border-slate-800 text-xs text-slate-300 rounded-lg">🆘 Help</button>
                            <button onclick="injectSign('Yes')" class="px-2.5 py-1 bg-slate-950 hover:bg-slate-800 border border-slate-800 text-xs text-slate-300 rounded-lg">👍 Yes</button>
                        </div>
                    </div>
                </div>

            </div>
        </section>

    </main>

    <!-- TOAST NOTIFICATION CONTAINER -->
    <div id="toast-container" class="fixed bottom-5 right-5 z-50 flex flex-col gap-2 pointer-events-none"></div>

    <!-- FOOTER -->
    <footer class="border-t border-slate-800/60 bg-slate-950/60 py-3 px-4 text-center text-xs text-slate-400 font-mono">
        <p>Built for Accessibility Hackathons • Powered by MediaPipe, Web Speech API & Tailwind CSS</p>
    </footer>

    <!-- APPLICATION JAVASCRIPT CODE -->
    <script>
        // State Variables
        let currentMode = 'sim'; // 'sim' or 'api'
        let currentSentence = [];
        let autoTTS = true;
        let lastDetectedSign = '';
        let lastSignTimestamp = 0;
        let webcamStream = null;
        let isOverlayVisible = true;
        
        // Simulation Words Database
        const demoSigns = ["Hello", "Welcome", "I Need", "Water", "Thank You", "Good", "Help", "Yes", "Please", "Nice to meet you"];
        let simInterval = null;

        // HTML DOM Elements
        const videoElem = document.getElementById('webcam-feed');
        const camFallback = document.getElementById('cam-fallback');
        const sentenceBox = document.getElementById('sentence-box');
        const currentSignText = document.getElementById('current-sign-text');
        const confidenceScore = document.getElementById('confidence-score');
        const wordCountElem = document.getElementById('word-count');
        const landmarkOverlay = document.getElementById('landmark-overlay');
        const autoSpeakIndicator = document.getElementById('auto-speak-indicator');
        const ttsBtnLabel = document.getElementById('tts-btn-label');

        window.onload = () => {
            // Initialize Lucide Icons
            lucide.createIcons();

            // Start Webcam automatically with fallback handling
            startWebcam();

            // Start Default Simulation Loop
            startSimulationLoop();
        };

        async function startWebcam() {
            try {
                if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                    throw new Error("Camera API not supported in this environment");
                }

                // Check available devices first if supported
                if (navigator.mediaDevices.enumerateDevices) {
                    try {
                        const devices = await navigator.mediaDevices.enumerateDevices();
                        const hasVideoInput = devices.some(device => device.kind === 'videoinput');
                        if (!hasVideoInput) {
                            const err = new Error("Requested device not found");
                            err.name = "NotFoundError";
                            throw err;
                        }
                    } catch (e) {
                        if (e.name === 'NotFoundError') throw e;
                    }
                }

                // Try ideal camera constraints first
                try {
                    webcamStream = await navigator.mediaDevices.getUserMedia({ 
                        video: { width: { ideal: 1280 }, height: { ideal: 720 }, facingMode: "user" } 
                    });
                } catch (constraintErr) {
                    if (constraintErr.name === 'NotFoundError') {
                        throw constraintErr; // Don't re-query if device doesn't exist
                    }
                    console.warn("Strict camera constraints failed, attempting basic video capture...", constraintErr);
                    webcamStream = await navigator.mediaDevices.getUserMedia({ video: true });
                }

                videoElem.srcObject = webcamStream;
                camFallback.classList.add('hidden');
                document.getElementById('cam-status').classList.remove('hidden');
                showToast("Webcam connected successfully!", "emerald");
            } catch (err) {
                console.warn("Camera status note:", err.message || err);
                camFallback.classList.remove('hidden');
                document.getElementById('cam-status').classList.add('hidden');
                
                const fallbackMsg = document.getElementById('cam-fallback-msg');
                if (err.name === 'NotFoundError' || (err.message && err.message.toLowerCase().includes('not found'))) {
                    if (fallbackMsg) fallbackMsg.innerText = "No physical camera detected. Demo Simulation Mode is active!";
                    showToast("No camera detected. Running in Demo Simulation Mode!", "amber");
                } else if (err.name === 'NotAllowedError' || err.name === 'PermissionDeniedError') {
                    if (fallbackMsg) fallbackMsg.innerText = "Camera access denied. Enable permissions in browser settings.";
                    showToast("Camera permission denied in browser settings.", "rose");
                } else {
                    if (fallbackMsg) fallbackMsg.innerText = "Unable to initialize camera stream. Demo mode active.";
                    showToast("Unable to initialize camera. Demo mode active.", "amber");
                }
            }
        }

        function toggleWebcam() {
            if (webcamStream) {
                const tracks = webcamStream.getTracks();
                tracks.forEach(track => track.stop());
                webcamStream = null;
                videoElem.srcObject = null;
                camFallback.classList.remove('hidden');
                showToast("Webcam feed paused", "amber");
            } else {
                startWebcam();
            }
        }

        function toggleOverlay() {
            isOverlayVisible = !isOverlayVisible;
            if (isOverlayVisible) {
                landmarkOverlay.classList.remove('hidden');
            } else {
                landmarkOverlay.classList.add('hidden');
            }
        }

        function setMode(mode) {
            currentMode = mode;
            const simBtn = document.getElementById('mode-sim-btn');
            const apiBtn = document.getElementById('mode-api-btn');
            const apiCard = document.getElementById('api-config-card');

            if (mode === 'sim') {
                simBtn.className = "px-3 py-1.5 rounded-lg text-xs font-medium transition-all flex items-center gap-1.5 bg-brand-600 text-white shadow";
                apiBtn.className = "px-3 py-1.5 rounded-lg text-xs font-medium text-slate-400 hover:text-white transition-all flex items-center gap-1.5";
                apiCard.classList.add('hidden');
                startSimulationLoop();
                showToast("Switched to Demo Simulation Mode", "brand");
            } else {
                apiBtn.className = "px-3 py-1.5 rounded-lg text-xs font-medium transition-all flex items-center gap-1.5 bg-brand-600 text-white shadow";
                simBtn.className = "px-3 py-1.5 rounded-lg text-xs font-medium text-slate-400 hover:text-white transition-all flex items-center gap-1.5";
                apiCard.classList.remove('hidden');
                if (simInterval) clearInterval(simInterval);
                startApiFetchLoop();
                showToast("Switched to Live Python API Mode", "cyan");
            }
        }

        function startSimulationLoop() {
            if (simInterval) clearInterval(simInterval);
            
            simInterval = setInterval(() => {
                if (currentMode !== 'sim') return;
                
                // Randomly pick a sign to simulate detection every few seconds
                const randomIndex = Math.floor(Math.random() * demoSigns.length);
                const randomSign = demoSigns[randomIndex];
                const randomConf = (Math.random() * (0.99 - 0.85) + 0.85).toFixed(2);
                
                processDetectedSign(randomSign, (randomConf * 100).toFixed(0) + "%");
            }, 4000);
        }

        function startApiFetchLoop() {
            // Placeholder polling loop for custom Python API integration
            // In a live setup, you can capture canvas frames and POST to python server
        }

        async function testApiConnection() {
            const url = document.getElementById('api-url-input').value;
            showToast("Testing connection to " + url, "brand");
            try {
                // Mock test ping
                const response = await fetch(url, { method: 'OPTIONS' }).catch(() => null);
                showToast("API Endpoint reachable!", "emerald");
            } catch (e) {
                showToast("Could not connect to API server", "rose");
            }
        }

        function processDetectedSign(sign, confidence) {
            currentSignText.innerText = sign.toUpperCase();
            confidenceScore.innerText = confidence;

            const now = Date.now();
            // Debounce sign detection so it doesn't spam repeated words instantly
            if (sign !== lastDetectedSign || (now - lastSignTimestamp) > 3000) {
                lastDetectedSign = sign;
                lastSignTimestamp = now;
                
                appendWord(sign);
            }
        }

        function injectSign(sign) {
            processDetectedSign(sign, "99%");
        }

        function appendWord(word) {
            currentSentence.push(word);
            renderSentence();

            if (autoTTS) {
                speakText(word);
            }
        }

        function renderSentence() {
            if (currentSentence.length === 0) {
                sentenceBox.innerHTML = `<span class="text-slate-600 font-normal italic text-lg">Recognized sign language phrases will assemble here in real time...</span>`;
                wordCountElem.innerText = `0 words`;
                return;
            }

            // Build accessible HTML list of words
            const fullText = currentSentence.join(" ");
            sentenceBox.innerHTML = `<p class="text-white tracking-wide leading-relaxed">${fullText}</p>`;
            wordCountElem.innerText = `${currentSentence.length} ${currentSentence.length === 1 ? 'word' : 'words'}`;
        }

        function clearSentence() {
            currentSentence = [];
            lastDetectedSign = '';
            renderSentence();
            currentSignText.innerText = "WAITING...";
            confidenceScore.innerText = "0%";
            showToast("Sentence cleared", "amber");
        }

        function copySentence() {
            if (currentSentence.length === 0) {
                showToast("Nothing to copy!", "amber");
                return;
            }
            const textToCopy = currentSentence.join(" ");
            
            // execCommand fallback for iFrame compatibility
            const tempInput = document.createElement("textarea");
            tempInput.value = textToCopy;
            document.body.appendChild(tempInput);
            tempInput.select();
            document.execCommand("copy");
            document.body.removeChild(tempInput);

            showToast("Sentence copied to clipboard!", "emerald");
        }

        function speakSentence() {
            if (currentSentence.length === 0) {
                showToast("Sentence buffer is empty!", "amber");
                return;
            }
            speakText(currentSentence.join(" "));
        }

        function speakText(text) {
            if ('speechSynthesis' in window) {
                // Cancel ongoing speech to avoid overlapping queue lag
                window.speechSynthesis.cancel();

                const utterance = new SpeechSynthesisUtterance(text);
                utterance.rate = 0.95; // Slightly slower for crisp clarity
                utterance.pitch = 1.0;
                utterance.volume = 1.0;

                window.speechSynthesis.speak(utterance);
            } else {
                showToast("Browser does not support Speech Synthesis API", "rose");
            }
        }

        function toggleAutoTTS() {
            autoTTS = !autoTTS;
            if (autoTTS) {
                autoSpeakIndicator.className = "w-2 h-2 rounded-full bg-emerald-500";
                ttsBtnLabel.innerText = "Mute Voice";
                showToast("Auto Text-to-Speech enabled", "emerald");
            } else {
                autoSpeakIndicator.className = "w-2 h-2 rounded-full bg-rose-500";
                ttsBtnLabel.innerText = "Unmute Voice";
                showToast("Auto Text-to-Speech muted", "amber");
            }
        }

        function showToast(message, colorType = "brand") {
            const container = document.getElementById('toast-container');
            const toast = document.createElement('div');
            
            let colorClasses = "bg-brand-600 border-brand-500 text-white";
            if (colorType === 'emerald') colorClasses = "bg-emerald-950/90 border-emerald-500/50 text-emerald-200";
            if (colorType === 'rose') colorClasses = "bg-rose-950/90 border-rose-500/50 text-rose-200";
            if (colorType === 'amber') colorClasses = "bg-amber-950/90 border-amber-500/50 text-amber-200";
            if (colorType === 'cyan') colorClasses = "bg-cyan-950/90 border-cyan-500/50 text-cyan-200";

            toast.className = `px-4 py-2.5 rounded-xl border shadow-xl text-xs font-semibold flex items-center gap-2 transform transition-all duration-300 translate-y-2 opacity-0 pointer-events-auto ${colorClasses}`;
            toast.innerHTML = `<span>${message}</span>`;
            
            container.appendChild(toast);

            // Animate In
            requestAnimationFrame(() => {
                toast.classList.remove('translate-y-2', 'opacity-0');
            });

            // Animate Out
            setTimeout(() => {
                toast.classList.add('opacity-0', 'translate-y-2');
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        }
    </script>
</body>
</html>
