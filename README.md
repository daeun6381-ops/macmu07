# macmu07
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Finger Synthesizer</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- MediaPipe Hands -->
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/control_utils/control_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>
    <!-- Tone.js & Tonal.js for Audio & Music Theory -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/tonal/browser/tonal.min.js" crossorigin="anonymous"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;600;700&display=swap');
        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: #05050a; /* 더 깊은 블랙으로 미니멀리즘 강조 */
            margin: 0;
            overflow: hidden;
        }
        /* 조잡한 흰색 테두리를 빼고 투명하고 깔끔한 블랙 글래스로 변경 */
        .glass-panel {
            background: rgba(0, 0, 0, 0.3);
            backdrop-filter: blur(8px);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
        }
        /* 은은하고 선명한 네온 글로우 */
        .glow-text {
            text-shadow: 0 0 8px rgba(167, 139, 250, 0.6), 0 0 16px rgba(167, 139, 250, 0.3);
        }
        #output_canvas {
            transform: scaleX(-1);
        }
        .hide-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .hide-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
    </style>
</head>
<body class="w-screen h-screen flex items-center justify-center bg-gradient-to-br from-indigo-950 via-purple-950 to-black text-white">

    <video id="input_video" class="hidden" autoplay playsinline></video>

    <canvas id="output_canvas" class="absolute top-0 left-0 w-full h-full object-cover opacity-50 mix-blend-screen"></canvas>

    <div id="ad-modal" class="absolute inset-0 z-[60] hidden flex flex-col items-center justify-center bg-black/95 backdrop-blur-lg">
        <div class="text-center">
            <span class="text-6xl mb-4 block opacity-80">📺</span>
            <h2 class="text-2xl font-bold text-indigo-300 mb-2 tracking-wide">광고 시청 중...</h2>
            <p class="text-indigo-200/60 mb-6 font-light">스폰서 광고를 시청하시면 코드 세트가 저장됩니다.</p>
            <div class="text-5xl font-bold text-fuchsia-400 mb-8 glow-text" id="ad-timer">3</div>
            <p class="text-sm text-gray-600">광고가 끝나면 자동으로 저장됩니다.</p>
        </div>
    </div>

    <div id="start-screen" class="absolute inset-0 z-50 flex flex-col items-center bg-black/80 backdrop-blur-md transition-opacity duration-500 overflow-y-auto py-16 hide-scrollbar">
        <h1 class="text-4xl md:text-5xl font-bold mb-6 glow-text text-center mt-auto tracking-tight">Finger Synthesizer</h1>
        
        <!-- 투박한 배경들을 걷어내고 얇은 선(border) 위주로 깔끔하게 정리 -->
        <div class="glass-panel p-8 rounded-2xl max-w-xl mb-6 w-[90%] text-center space-y-4 border-2 border-indigo-500/30">
            <p class="text-lg text-indigo-200/80 font-light">Unfold your fingers to play dreamy melodies and chords.</p>
            
            <div class="w-full mt-4 bg-black/20 p-5 rounded-xl border-2 border-indigo-500/20">
                <div class="flex justify-between items-end mb-4">
                    <label for="chord-input" class="block text-sm text-indigo-300/80 font-semibold text-left tracking-wide">연주할 코드 (쉼표로 구분)</label>
                    <!-- 그라데이션 제거, 심플한 홀로그램 느낌의 버튼으로 변경 -->
                    <button id="save-ad-btn" class="text-xs bg-fuchsia-950/40 hover:bg-fuchsia-900/60 border-2 border-fuchsia-500/40 text-fuchsia-300 px-3 py-1.5 rounded-md font-medium flex items-center transition-all">
                        <span class="mr-1">📺</span> 광고 보고 코드 저장
                    </button>
                </div>
                <!-- 텍스트 입력창 미니멀리즘 스타일 적용 -->
                <input type="text" id="chord-input" class="w-full bg-transparent border-b-2 border-indigo-500/40 p-2 text-white outline-none focus:border-indigo-400 transition-all text-lg font-mono tracking-wider text-center" value="Cmaj7, D7, Em7, Fmaj7, G7">
                
                <div class="mt-5 text-xs md:text-sm text-indigo-200/60 text-left space-y-2 font-light border-l-2 border-indigo-500/30 pl-4">
                    <p>✨ <strong>[1~5개 입력 시]</strong> 양손이 동일하게 1~5번 화음을 공유합니다.</p>
                    <p>✨ <strong>[6~10개 입력 시]</strong> 양손이 각각 완전히 다른 화음을 가집니다!</p>
                </div>
            </div>

            <div class="w-full bg-black/20 p-5 rounded-xl border-2 border-purple-500/20 mt-4 text-left">
                <h3 class="text-sm font-semibold text-purple-300/80 mb-3 flex items-center tracking-wide">
                    <span class="mr-2 opacity-70">💾</span> 내 저장된 코드 세트
                </h3>
                <ul id="saved-chords-list" class="space-y-1 max-h-32 overflow-y-auto hide-scrollbar">
                    <li class="text-xs text-gray-500 font-light text-center py-4">저장된 코드가 없습니다.</li>
                </ul>
            </div>
        </div>

        <!-- 그라데이션 그림자 제거, 엣지있는 테두리와 은은한 백그라운드 -->
        <button id="start-btn" class="px-12 py-4 bg-indigo-900/30 border-2 border-indigo-500/50 hover:bg-indigo-800/50 hover:border-indigo-400 text-indigo-200 hover:text-white rounded-full font-semibold text-lg transition-all mb-auto tracking-widest">
            연주 시작하기
        </button>
        <p id="loading-text" class="mt-4 text-indigo-400/60 font-light hidden mb-auto animate-pulse">카메라와 AI 모델을 불러오는 중입니다...</p>
    </div>

    <div class="absolute top-0 left-0 w-full h-full pointer-events-none p-4 md:p-6 flex flex-col justify-between z-10">
        <div class="flex justify-between items-start w-full gap-2">
            
            <div class="flex flex-col gap-3 w-32 md:w-40">
                <div class="glass-panel p-3 md:p-4 rounded-xl text-xs md:text-sm text-indigo-100/90 border-2 border-indigo-500/30 pointer-events-auto">
                    <h3 class="font-semibold mb-3 text-indigo-400/80 text-center text-[10px] uppercase tracking-widest">화면 왼쪽</h3>
                    <ul id="left-chord-list" class="space-y-1 md:space-y-1.5 text-left font-light"></ul>
                </div>
                <!-- 얇은 선(border)과 매우 투명한 배경으로 세련된 무음존 -->
                <div id="mute-zone-left" class="h-48 md:h-64 rounded-xl border-dashed border-2 border-indigo-500/40 bg-black/20 text-center text-indigo-400/50 transition-all duration-300 pointer-events-auto flex flex-col items-center justify-center backdrop-blur-sm">
                    <span class="text-xl mb-2 opacity-60">🔇</span>
                    <span class="text-[11px] uppercase tracking-widest font-semibold">무음 존</span>
                </div>
            </div>

            <!-- 볼륨 인디케이터 심플화 -->
            <div id="volume-indicator" class="glass-panel px-4 py-2 md:px-6 rounded-full text-[10px] md:text-xs font-semibold tracking-[0.2em] text-indigo-300/60 border-2 border-indigo-500/20 transition-all duration-300 h-10 flex items-center justify-center min-w-[140px] uppercase">
                NORMAL VOLUME
            </div>
            
            <div class="flex flex-col gap-3 w-32 md:w-40">
                <div class="glass-panel p-3 md:p-4 rounded-xl text-xs md:text-sm text-purple-100/90 border-2 border-purple-500/30 pointer-events-auto">
                    <h3 class="font-semibold mb-3 text-purple-400/80 text-center text-[10px] uppercase tracking-widest">화면 오른쪽</h3>
                    <ul id="right-chord-list" class="space-y-1 md:space-y-1.5 text-left font-light"></ul>
                </div>
                <!-- 얇은 선(border)과 매우 투명한 배경으로 세련된 무음존 -->
                <div id="mute-zone-right" class="h-48 md:h-64 rounded-xl border-dashed border-2 border-purple-500/40 bg-black/20 text-center text-purple-400/50 transition-all duration-300 pointer-events-auto flex flex-col items-center justify-center backdrop-blur-sm">
                    <span class="text-xl mb-2 opacity-60">🔇</span>
                    <span class="text-[11px] uppercase tracking-widest font-semibold">무음 존</span>
                </div>
            </div>
        </div>

        <div class="flex justify-between items-end w-full">
            <div class="glass-panel p-4 md:p-5 rounded-2xl w-32 md:w-40 text-center flex flex-col items-center border-2 border-indigo-500/30">
                <p class="text-[10px] uppercase tracking-widest text-indigo-400/60 mb-2 font-semibold">현재 상태</p>
                <h2 id="left-chord-display" class="text-2xl md:text-3xl font-light text-white mb-3 glow-text">-</h2>
                <!-- 꽉 찬 동그라미 대신 투명한 링 형태로 변경 -->
                <div class="w-10 h-10 md:w-12 md:h-12 rounded-full bg-transparent flex items-center justify-center border-2 border-indigo-500/40">
                    <span id="left-finger-display" class="text-sm md:text-base font-light text-indigo-300">0</span>
                </div>
            </div>

            <div class="glass-panel p-4 md:p-5 rounded-2xl w-32 md:w-40 text-center flex flex-col items-center border-2 border-purple-500/30">
                <p class="text-[10px] uppercase tracking-widest text-purple-400/60 mb-2 font-semibold">현재 상태</p>
                <h2 id="right-chord-display" class="text-2xl md:text-3xl font-light text-white mb-3 glow-text">-</h2>
                <!-- 꽉 찬 동그라미 대신 투명한 링 형태로 변경 -->
                <div class="w-10 h-10 md:w-12 md:h-12 rounded-full bg-transparent flex items-center justify-center border-2 border-purple-500/40">
                    <span id="right-finger-display" class="text-sm md:text-base font-light text-purple-300">0</span>
                </div>
            </div>
        </div>
    </div>

    <script type="module">
        // ==========================================
        // 🚀 FIREBASE 연동 설정 (데이터 저장용)
        // ==========================================
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInWithCustomToken, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, deleteDoc, doc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyDx0EbBZXNESSTwMbwt2ivnRJZsiE8kPwQ",
            authDomain: "story-2924c.firebaseapp.com",
            projectId: "story-2924c",
            storageBucket: "story-2924c.firebasestorage.app",
            messagingSenderId: "160249336039",
            appId: "1:160249336039:web:36a453f2a4264547c21242",
            measurementId: "G-6Z61XM2MMW"
        };

        const isPlatformEnv = typeof __firebase_config !== 'undefined' && __firebase_config !== '{}';
        const finalConfig = isPlatformEnv ? JSON.parse(__firebase_config) : firebaseConfig;
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

        let app, auth, db, currentUser;

        try {
            app = initializeApp(finalConfig);
            auth = getAuth(app);
            db = getFirestore(app);

            const initAuth = async () => {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            };
            
            initAuth();

            onAuthStateChanged(auth, (user) => {
                currentUser = user;
                if (user) {
                    loadSavedChords(); 
                }
            });
        } catch (error) {
            console.log("Firebase가 아직 구성되지 않았습니다. 로컬 모드로 작동합니다.", error);
        }

        const adModal = document.getElementById('ad-modal');
        const adTimerText = document.getElementById('ad-timer');
        const savedChordsList = document.getElementById('saved-chords-list');

        document.getElementById('save-ad-btn').addEventListener('click', (e) => {
            if (!currentUser) {
                const btn = e.currentTarget;
                const originalText = btn.innerHTML;
                btn.innerHTML = "연결 중입니다...";
                setTimeout(() => btn.innerHTML = originalText, 2000);
                return;
            }
            
            adModal.classList.remove('hidden');
            let timeLeft = 3; 
            adTimerText.innerText = timeLeft;
            
            const timer = setInterval(() => {
                timeLeft--;
                adTimerText.innerText = timeLeft;
                if (timeLeft <= 0) {
                    clearInterval(timer);
                    adModal.classList.add('hidden');
                    saveCurrentChordsToFirestore();
                }
            }, 1000);
        });

        async function saveCurrentChordsToFirestore() {
            if (!currentUser || !db) return;
            const chordsValue = document.getElementById('chord-input').value;
            
            const chordsRef = collection(db, 'artifacts', appId, 'users', currentUser.uid, 'savedChords');
            try {
                await addDoc(chordsRef, { chords: chordsValue, createdAt: new Date().toISOString() });
            } catch (error) {
                console.error("저장 실패:", error);
            }
        }

        function loadSavedChords() {
            if (!currentUser || !db) return;
            const chordsRef = collection(db, 'artifacts', appId, 'users', currentUser.uid, 'savedChords');
            
            onSnapshot(chordsRef, (snapshot) => {
                savedChordsList.innerHTML = ''; 
                
                if(snapshot.empty) {
                    savedChordsList.innerHTML = '<li class="text-xs text-gray-500 font-light text-center py-4">저장된 코드가 없습니다.</li>';
                    return;
                }

                snapshot.forEach(docSnap => {
                    const data = docSnap.data();
                    const docId = docSnap.id;
                    const li = document.createElement('li');
                    /* 불러오기 리스트 디자인도 아주 심플하게 선과 텍스트로만 정리 */
                    li.className = "flex justify-between items-center border-b-2 border-purple-500/20 py-2.5";
                    li.innerHTML = `
                        <span class="truncate w-40 text-xs md:text-sm text-indigo-200 font-mono tracking-wider">${data.chords}</span>
                        <div class="flex gap-1.5">
                            <button class="bg-indigo-900/30 hover:bg-indigo-800/50 border-2 border-indigo-500/40 px-3 py-1 rounded text-[10px] load-btn text-indigo-200 transition-colors uppercase tracking-wider" data-chords="${data.chords}">적용</button>
                            <button class="bg-transparent hover:bg-red-900/20 px-2 py-1 rounded text-[10px] delete-btn text-red-400/70 hover:text-red-300 transition-colors" data-id="${docId}">✕</button>
                        </div>
                    `;
                    savedChordsList.appendChild(li);
                });

                document.querySelectorAll('.load-btn').forEach(btn => {
                    btn.addEventListener('click', (e) => {
                        document.getElementById('chord-input').value = e.target.getAttribute('data-chords');
                    });
                });

                document.querySelectorAll('.delete-btn').forEach(btn => {
                    btn.addEventListener('click', async (e) => {
                        const targetId = e.target.getAttribute('data-id');
                        await deleteDoc(doc(db, 'artifacts', appId, 'users', currentUser.uid, 'savedChords', targetId));
                    });
                });

            }, (error) => {
                console.error("리스너 오류:", error);
            });
        }


        // ==========================================
        // 🎹 기존 악기 로직 (수정 내용 없음 - 100% 동일)
        // ==========================================
        const videoElement = document.getElementById('input_video');
        const canvasElement = document.getElementById('output_canvas');
        const canvasCtx = canvasElement.getContext('2d');
        const startBtn = document.getElementById('start-btn');
        const startScreen = document.getElementById('start-screen');
        const loadingText = document.getElementById('loading-text');
        const chordInput = document.getElementById('chord-input');
        
        const uiLeftChord = document.getElementById('left-chord-display');
        const uiRightChord = document.getElementById('right-chord-display');
        const uiLeftFinger = document.getElementById('left-finger-display');
        const uiRightFinger = document.getElementById('right-finger-display');
        const uiVolume = document.getElementById('volume-indicator');
        const leftChordListUI = document.getElementById('left-chord-list');
        const rightChordListUI = document.getElementById('right-chord-list');
        const muteZoneLeftEl = document.getElementById('mute-zone-left');
        const muteZoneRightEl = document.getElementById('mute-zone-right');

        let leftChordsMap = {};  
        let rightChordsMap = {};
        let leftChordNames = {}; 
        let rightChordNames = {};
        
        const FINGER_LABELS = ['1️⃣ 검지', '2️⃣ 중지', '3️⃣ 약지', '4️⃣ 새끼', '5️⃣ 엄지'];

        function getNotesWithOctave(chordName, rootOctave = 4) {
            const chordInfo = Tonal.Chord.get(chordName);
            if (chordInfo.empty) {
                const fallback = Tonal.Chord.get("C");
                return fallback.intervals.map(iv => Tonal.Note.transpose(`C${rootOctave}`, iv));
            }
            const root = chordInfo.tonic || chordInfo.root || "C";
            return chordInfo.intervals.map(iv => Tonal.Note.transpose(`${root}${rootOctave}`, iv));
        }

        function parseAndAssignChords() {
            const inputStr = chordInput.value;
            let rawChords = inputStr.split(',').map(s => s.trim()).filter(s => s.length > 0);
            
            if (rawChords.length === 0) {
                rawChords = ['Cmaj7', 'D7', 'Em7', 'Fmaj7', 'G7'];
            }

            let expandedChords = [];
            for (let i = 0; i < 10; i++) {
                expandedChords.push(rawChords[i % rawChords.length]);
            }

            leftChordsMap = {}; rightChordsMap = {};
            leftChordNames = {}; rightChordNames = {};

            if (rawChords.length <= 5) {
                for (let i = 1; i <= 5; i++) {
                    const chordName = rawChords[(i - 1) % rawChords.length];
                    const notes = getNotesWithOctave(chordName, 4);
                    
                    leftChordsMap[i] = notes;
                    rightChordsMap[i] = notes;
                    leftChordNames[i] = chordName;
                    rightChordNames[i] = chordName;
                }
            } else {
                for (let i = 1; i <= 5; i++) {
                    const leftName = expandedChords[i - 1]; 
                    const rightName = expandedChords[i + 4]; 
                    
                    leftChordsMap[i] = getNotesWithOctave(leftName, 4);
                    leftChordNames[i] = leftName;
                    
                    rightChordsMap[i] = getNotesWithOctave(rightName, 4);
                    rightChordNames[i] = rightName;
                }
            }

            renderChordGuidesUI();
        }

        function renderChordGuidesUI() {
            leftChordListUI.innerHTML = '';
            rightChordListUI.innerHTML = '';
            
            /* 목록 텍스트 간격도 미니멀하게 정리 */
            for (let i = 1; i <= 5; i++) {
                leftChordListUI.innerHTML += `
                    <li class="flex justify-between items-center w-full">
                        <span class="tracking-[0.1em] mr-2 text-[10px] text-indigo-300/60">${FINGER_LABELS[i-1]}</span> 
                        <strong class="text-white truncate font-light">${leftChordNames[i]}</strong>
                    </li>`;
                rightChordListUI.innerHTML += `
                    <li class="flex justify-between items-center w-full">
                        <span class="tracking-[0.1em] mr-2 text-[10px] text-purple-300/60">${FINGER_LABELS[i-1]}</span> 
                        <strong class="text-white truncate font-light">${rightChordNames[i]}</strong>
                    </li>`;
            }
        }

        const synthOptions = {
            oscillator: { type: "sine" },
            envelope: { attack: 0.03, decay: 0.1, sustain: 1.0, release: 2.5 },
            volume: -12
        };

        let leftSynth, rightSynth, masterVolume;

        async function initAudio() {
            Tone.setContext(new Tone.Context({ latencyHint: "interactive", lookAhead: 0 }));
            await Tone.start();
            
            const limiter = new Tone.Limiter(-2).toDestination();
            const reverb = new Tone.Reverb({ decay: 5, wet: 0.5 });
            const delay = new Tone.FeedbackDelay("8n", 0.2); 
            const lowcutFilter = new Tone.Filter({ frequency: 250, type: "highpass", Q: 0 }); 
            
            masterVolume = new Tone.Volume(-10);

            masterVolume.connect(limiter);
            reverb.connect(masterVolume);
            delay.connect(reverb);
            lowcutFilter.connect(delay);

            leftSynth = new Tone.PolySynth(Tone.Synth, synthOptions).connect(lowcutFilter);
            rightSynth = new Tone.PolySynth(Tone.Synth, synthOptions).connect(lowcutFilter);
        }

        function getDistance(p1, p2) {
            return Math.sqrt(Math.pow(p1.x - p2.x, 2) + Math.pow(p1.y - p2.y, 2));
        }

        function identifyFinger(landmarks) {
            const isIndex = getDistance(landmarks[8], landmarks[0]) > getDistance(landmarks[6], landmarks[0]);
            const isMiddle = getDistance(landmarks[12], landmarks[0]) > getDistance(landmarks[10], landmarks[0]);
            const isRing = getDistance(landmarks[16], landmarks[0]) > getDistance(landmarks[14], landmarks[0]);
            const isPinky = getDistance(landmarks[20], landmarks[0]) > getDistance(landmarks[18], landmarks[0]);
            const isThumb = getDistance(landmarks[4], landmarks[9]) > getDistance(landmarks[3], landmarks[9]) * 1.1;

            const openFingers = [];
            if (isIndex) openFingers.push(1);
            if (isMiddle) openFingers.push(2);
            if (isRing) openFingers.push(3);
            if (isPinky) openFingers.push(4);
            if (isThumb) openFingers.push(5);

            if (openFingers.length === 0) return -1; 
            if (openFingers.length === 1) return openFingers[0];
            if (openFingers.length > 1) return openFingers.length;
            
            return 0; 
        }

        class FingerSmoother {
            constructor(threshold = 1) {
                this.threshold = threshold;
                this.currentValue = 0;
                this.candidateValue = 0;
                this.count = 0;
            }
            add(val) {
                if (val === this.candidateValue) {
                    this.count++;
                } else {
                    this.candidateValue = val;
                    this.count = 1;
                }
                
                if (this.count >= this.threshold) {
                    this.currentValue = val;
                }
                return this.currentValue;
            }
        }

        const leftSmoother = new FingerSmoother();
        const rightSmoother = new FingerSmoother();

        let currentLeftFingers = 0;
        let currentRightFingers = 0;

        function updateAudioAndUI(leftCount, rightCount) {
            const isLeftActive = leftCount > 0;
            const isRightActive = rightCount > 0;

            if (isLeftActive && isRightActive && leftCount === rightCount) {
                masterVolume.volume.rampTo(-4, 0.5); 
                uiVolume.innerText = "MAX RESONANCE";
                uiVolume.classList.add("text-fuchsia-300/90", "border-fuchsia-500/40");
            } else {
                masterVolume.volume.rampTo(-10, 0.5);
                uiVolume.innerText = "NORMAL VOLUME";
                uiVolume.classList.remove("text-fuchsia-300/90", "border-fuchsia-500/40");
            }

            if (leftCount !== currentLeftFingers) {
                const releaseTime = leftCount === -1 ? 0.1 : 2.5;
                leftSynth.set({ envelope: { release: releaseTime } });
                leftSynth.releaseAll(); 
                
                if (leftCount > 0 && leftChordsMap[leftCount]) {
                    leftSynth.triggerAttack(leftChordsMap[leftCount], Tone.now());
                }
                currentLeftFingers = leftCount;
            }

            if (rightCount !== currentRightFingers) {
                const releaseTime = rightCount === -1 ? 0.1 : 2.5;
                rightSynth.set({ envelope: { release: releaseTime } });
                rightSynth.releaseAll();

                if (rightCount > 0 && rightChordsMap[rightCount]) {
                    rightSynth.triggerAttack(rightChordsMap[rightCount], Tone.now());
                }
                currentRightFingers = rightCount;
            }

            uiLeftChord.innerText = leftCount > 0 ? leftChordNames[leftCount] : (leftCount === -1 ? '주먹' : '무음 존');
            uiLeftFinger.innerText = leftCount > 0 ? leftCount : (leftCount === -1 ? '✊' : '🔇');
            
            uiRightChord.innerText = rightCount > 0 ? rightChordNames[rightCount] : (rightCount === -1 ? '주먹' : '무음 존');
            uiRightFinger.innerText = rightCount > 0 ? rightCount : (rightCount === -1 ? '✊' : '🔇');
        }

        function onResults(results) {
            canvasCtx.save();
            canvasCtx.clearRect(0, 0, canvasElement.width, canvasElement.height);
            
            canvasCtx.drawImage(results.image, 0, 0, canvasElement.width, canvasElement.height);
            canvasCtx.fillStyle = "rgba(5, 5, 10, 0.6)"; // 카메라 배경도 더 어둡게 미니멀하게 처리
            canvasCtx.fillRect(0, 0, canvasElement.width, canvasElement.height);

            let rawLeft = null;
            let rawRight = null;
            
            let isLeftZoneActive = false;
            let isRightZoneActive = false;

            const leftZoneRect = muteZoneLeftEl.getBoundingClientRect();
            const rightZoneRect = muteZoneRightEl.getBoundingClientRect();

            if (results.multiHandLandmarks && results.multiHandLandmarks.length > 0) {
                for (let i = 0; i < results.multiHandLandmarks.length; i++) {
                    const landmarks = results.multiHandLandmarks[i];
                    if (!landmarks || landmarks.length < 21) continue;

                    const screenX = window.innerWidth * (1 - landmarks[9].x); 
                    const screenY = window.innerHeight * landmarks[9].y;

                    let fingerCount = identifyFinger(landmarks);
                    const isLeftScreen = landmarks[0].x > 0.5;

                    if (screenX >= leftZoneRect.left && screenX <= leftZoneRect.right &&
                        screenY >= leftZoneRect.top && screenY <= leftZoneRect.bottom) {
                        fingerCount = 0; 
                        isLeftZoneActive = true;
                    }
                    else if (screenX >= rightZoneRect.left && screenX <= rightZoneRect.right &&
                             screenY >= rightZoneRect.top && screenY <= rightZoneRect.bottom) {
                        fingerCount = 0; 
                        isRightZoneActive = true;
                    }

                    // 선 색상도 미니멀하게 조정
                    const lineColor = isLeftScreen ? 'rgba(129, 140, 248, 0.4)' : 'rgba(192, 132, 252, 0.4)';
                    const pointColor = isLeftScreen ? 'rgba(99, 102, 241, 0.8)' : 'rgba(168, 85, 247, 0.8)';
                    window.drawConnectors(canvasCtx, landmarks, window.HAND_CONNECTIONS, {color: lineColor, lineWidth: 2}); // 뼈대 굵기 얇게
                    window.drawLandmarks(canvasCtx, landmarks, {color: pointColor, lineWidth: 1, radius: 2}); // 포인트 크기 작게

                    if (isLeftScreen) {
                        if (rawLeft === null) rawLeft = fingerCount;
                    } else {
                        if (rawRight === null) rawRight = fingerCount;
                    }
                }
            }

            // 활성화 되었을 때의 효과도 은은하게
            if (isLeftZoneActive) {
                muteZoneLeftEl.style.backgroundColor = 'rgba(79, 70, 229, 0.2)';
                muteZoneLeftEl.style.borderColor = 'rgba(129, 140, 248, 0.6)';
            } else {
                muteZoneLeftEl.style.backgroundColor = 'rgba(0, 0, 0, 0.2)';
                muteZoneLeftEl.style.borderColor = 'rgba(99, 102, 241, 0.3)';
            }

            if (isRightZoneActive) {
                muteZoneRightEl.style.backgroundColor = 'rgba(147, 51, 234, 0.2)';
                muteZoneRightEl.style.borderColor = 'rgba(192, 132, 252, 0.6)';
            } else {
                muteZoneRightEl.style.backgroundColor = 'rgba(0, 0, 0, 0.2)';
                muteZoneRightEl.style.borderColor = 'rgba(168, 85, 247, 0.3)';
            }

            if (rawLeft === null) rawLeft = 0;
            if (rawRight === null) rawRight = 0;

            const smoothedLeft = leftSmoother.add(rawLeft);
            const smoothedRight = rightSmoother.add(rawRight);

            if (leftSynth && rightSynth) {
                updateAudioAndUI(smoothedLeft, smoothedRight);
            }

            canvasCtx.restore();
        }

        const hands = new window.Hands({locateFile: (file) => {
            return `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`;
        }});
        
        hands.setOptions({
            maxNumHands: 2,
            modelComplexity: 0, 
            minDetectionConfidence: 0.7,
            minTrackingConfidence: 0.7
        });
        hands.onResults(onResults);

        const camera = new window.Camera(videoElement, {
            onFrame: async () => {
                await hands.send({image: videoElement});
            },
            width: 640,
            height: 480
        });

        function resizeCanvas() {
            canvasElement.width = window.innerWidth;
            canvasElement.height = window.innerHeight;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        startBtn.addEventListener('click', async () => {
            parseAndAssignChords();

            startBtn.classList.add('hidden');
            chordInput.parentElement.classList.add('hidden'); 
            document.getElementById('saved-chords-list').parentElement.classList.add('hidden'); 
            loadingText.classList.remove('hidden');
            
            try {
                await initAudio();
                await camera.start();
                
                startScreen.style.opacity = '0';
                setTimeout(() => {
                    startScreen.style.display = 'none';
                }, 500);
                
            } catch (error) {
                console.error("초기화 오류:", error);
                loadingText.innerText = "카메라 접근 권한이 필요합니다. 새로고침 후 허용해주세요.";
            }
        });
    </script>
</body>
</html>
