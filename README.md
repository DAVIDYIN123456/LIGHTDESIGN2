<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>网球学练馆 · 灯光引导训练</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none;
        }
        body {
            margin: 0;
            padding: 0;
            background-color: #2c3e50;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            cursor: pointer;
        }
        body.game-active {
            cursor: none;
        }

        #customCursor {
            position: fixed;
            pointer-events: none;
            z-index: 9999;
            width: 105px;
            height: 105px;
            transform: translate(-15px, -15px);
            transition: none;
            filter: drop-shadow(0 4px 20px rgba(0,0,0,0.7));
            display: none;
        }
        body.game-active #customCursor {
            display: block;
        }
        #customCursor svg {
            width: 100%;
            height: 100%;
            display: block;
        }
        #customCursor .motion-lines {
            position: absolute;
            top: -8px;
            right: -22px;
            width: 75px;
            height: 105px;
            border-left: 4px solid rgba(255,255,255,0.5);
            border-radius: 50%;
            transform: rotate(45deg);
            z-index: -1;
            animation: swing-cursor 0.7s infinite ease-in-out;
        }
        @keyframes swing-cursor {
            0% { transform: rotate(35deg) translateX(0px); opacity: 0.2; }
            50% { transform: rotate(55deg) translateX(8px); opacity: 0.9; }
            100% { transform: rotate(35deg) translateX(0px); opacity: 0.2; }
        }
        #customCursor.hit {
            animation: hit-swing 0.3s ease-out;
        }
        @keyframes hit-swing {
            0% { transform: translate(-15px, -15px) rotate(0deg) scale(1); }
            30% { transform: translate(-15px, -15px) rotate(-15deg) scale(1.2); }
            100% { transform: translate(-15px, -15px) rotate(0deg) scale(1); }
        }

        .scene {
            width: 800px;
            height: 600px;
            perspective: 900px;
            position: relative;
        }

        .court-surround {
            position: absolute;
            width: 800px;
            height: 600px;
            background-color: #4a8f3c;
            transform: rotateX(55deg);
            transform-origin: center bottom;
            border: 4px solid #fff;
            box-sizing: border-box;
            box-shadow: 0 20px 40px rgba(0,0,0,0.5);
        }

        .blue-court {
            position: absolute;
            top: 50px;
            left: 50px;
            right: 50px;
            bottom: 50px;
            background-color: #1e5799;
            border: 4px solid #fff;
            box-sizing: border-box;
        }

        .service-line {
            position: absolute;
            bottom: 35%;
            left: 0;
            width: 100%;
            height: 6px;
            background-color: #fff;
            box-shadow: 0 2px 4px rgba(0,0,0,0.5);
            z-index: 2;
        }

        .center-line {
            position: absolute;
            bottom: 35%;
            left: 50%;
            width: 6px;
            height: 65%;
            background-color: #fff;
            transform: translateX(-50%);
            z-index: 2;
        }

        .fade-out {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 30%;
            background: linear-gradient(to bottom, rgba(44, 62, 80, 1), transparent);
            pointer-events: none;
            z-index: 5;
        }

        .target-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 10;
        }

        .target-dot {
            position: absolute;
            border-radius: 50%;
            pointer-events: none;
            transform: translate(-50%, -50%);
            transition: transform 0.15s cubic-bezier(0.34, 1.56, 0.64, 1);
            will-change: transform, box-shadow;
            background: #ffffff;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.3), inset 0 0 10px rgba(255, 255, 255, 0.1);
            border: 2px solid rgba(255, 255, 255, 0.3);
        }

        .target-dot.active {
            background: #ff1a1a;
            box-shadow: 0 0 40px #ff1a1a, 0 0 80px rgba(255, 26, 26, 0.6), 0 0 120px rgba(255, 26, 26, 0.3);
            border-color: #ff4444;
            animation: pulse-red 0.8s infinite alternate;
        }
        @keyframes pulse-red {
            0% { box-shadow: 0 0 30px #ff1a1a, 0 0 60px rgba(255, 26, 26, 0.4); }
            100% { box-shadow: 0 0 50px #ff1a1a, 0 0 100px rgba(255, 26, 26, 0.7), 0 0 150px rgba(255, 26, 26, 0.3); }
        }

        .target-dot.arrived {
            background: #2ecc71;
            box-shadow: 0 0 40px #2ecc71, 0 0 80px rgba(46, 204, 113, 0.5);
            border-color: #2ecc71;
            animation: none;
        }

        .target-dot.timeout {
            background: #666666;
            box-shadow: 0 0 20px rgba(100,100,100,0.3);
            border-color: #555555;
            animation: none;
            opacity: 0.5;
        }

        .target-dot.hidden-dot {
            opacity: 0.3;
            pointer-events: none;
            animation: none;
        }

        .arrow-indicator {
            position: absolute;
            z-index: 20;
            pointer-events: none;
            font-size: 34px;
            color: #ff2a2a;
            text-shadow: 0 0 30px #ff0000, 0 0 60px #ff4444;
            transform: translate(-50%, -50%);
            animation: bounce-arrow 0.6s infinite alternate;
            display: none;
            font-weight: bold;
            line-height: 1;
            filter: drop-shadow(0 0 20px #ff0000);
        }
        .arrow-indicator.show {
            display: block;
        }
        .arrow-indicator span {
            display: block;
            font-size: 16px;
            background: rgba(0, 0, 0, 0.75);
            padding: 6px 20px;
            border-radius: 40px;
            color: #fff;
            text-shadow: 0 0 12px #ff3b3b;
            margin-top: 2px;
            letter-spacing: 2px;
            backdrop-filter: blur(4px);
            border: 1px solid rgba(255,50,50,0.3);
        }

        @keyframes bounce-arrow {
            0% { transform: translate(-50%, -50%) translateY(-10px); }
            100% { transform: translate(-50%, -50%) translateY(12px); }
        }

        .machine {
            position: absolute;
            width: 60px;
            height: 40px;
            background: #2c3e50;
            border-radius: 14px 14px 6px 6px;
            top: 68px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 15;
            border: 2px solid #5d7a8a;
            box-shadow: 0 8px 0 #1a2630, 0 12px 30px rgba(0,0,0,0.5);
            pointer-events: none;
        }
        .machine::before {
            content: '';
            position: absolute;
            width: 20px;
            height: 20px;
            background: #27ae60;
            border-radius: 50%;
            top: 8px;
            left: 18px;
            box-shadow: 0 0 24px #2ecc71;
            transition: 0.2s;
            border: 1px solid rgba(255,255,255,0.15);
        }
        .machine.firing::before {
            background: #e74c3c;
            box-shadow: 0 0 30px #ff4444, 0 0 60px #ff2222;
            animation: blink-red 0.25s infinite alternate;
        }

        @keyframes blink-red {
            0% { opacity: 0.6; transform: scale(0.95); }
            100% { opacity: 1; transform: scale(1.1); }
        }

        .tennis-ball {
            position: absolute;
            width: 22px;
            height: 22px;
            background: radial-gradient(circle at 35% 28%, #f9e56a, #c7b32a);
            border-radius: 50%;
            z-index: 30;
            pointer-events: none;
            box-shadow: 0 0 30px rgba(255, 215, 0, 0.6), 0 8px 20px rgba(0,0,0,0.4);
            display: none;
            will-change: transform;
            transform: translate(-50%, -50%);
            border: 1px solid rgba(255,255,200,0.2);
        }

        .trail-canvas {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 18;
        }

        /* --- 击球文字提示 --- */
        .hit-text {
            position: absolute;
            z-index: 50;
            pointer-events: none;
            font-size: 72px;
            font-weight: 900;
            color: #ff6b35;
            text-shadow: 
                0 0 20px rgba(255, 107, 53, 0.8),
                0 0 40px rgba(255, 107, 53, 0.5),
                0 4px 8px rgba(0,0,0,0.5);
            transform: translate(-50%, -50%) scale(0);
            opacity: 0;
            transition: none;
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            letter-spacing: 4px;
            white-space: nowrap;
        }
        .hit-text.show {
            animation: hit-text-pop 0.7s ease-out forwards;
        }
        @keyframes hit-text-pop {
            0% { transform: translate(-50%, -50%) scale(0) rotate(-10deg); opacity: 0; }
            30% { transform: translate(-50%, -50%) scale(1.5) rotate(3deg); opacity: 1; }
            60% { transform: translate(-50%, -50%) scale(1.1) rotate(-2deg); opacity: 1; }
            80% { transform: translate(-50%, -50%) scale(1.3) rotate(1deg); opacity: 1; }
            100% { transform: translate(-50%, -50%) scale(1) rotate(0deg); opacity: 0; }
        }

        .ui-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 40;
        }
        .ui-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 12px 20px 0;
            color: #e8f0f5;
            font-weight: 500;
            letter-spacing: 0.5px;
            pointer-events: none;
        }
        .ui-header .brand {
            font-size: 17px;
            background: rgba(0,0,0,0.5);
            padding: 4px 20px;
            border-radius: 40px;
            border-left: 3px solid #f1c40f;
            backdrop-filter: blur(4px);
            box-shadow: 0 2px 10px rgba(0,0,0,0.3);
        }
        .ui-header .timer {
            font-size: 26px;
            font-weight: 700;
            color: #f1c40f;
            background: rgba(0,0,0,0.5);
            padding: 2px 20px;
            border-radius: 40px;
            backdrop-filter: blur(4px);
            min-width: 70px;
            text-align: center;
        }
        .ui-footer {
            position: absolute;
            bottom: 16px;
            left: 0;
            width: 100%;
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 0 20px;
            color: #b0c7d4;
            pointer-events: none;
        }
        .ui-footer > * {
            pointer-events: auto;
        }
        .score-area {
            font-size: 28px;
            font-weight: 700;
            color: #f1c40f;
            background: rgba(26, 44, 54, 0.85);
            padding: 0 24px;
            border-radius: 40px;
            line-height: 48px;
            box-shadow: inset 0 2px 8px rgba(0,0,0,0.5), 0 4px 12px rgba(0,0,0,0.3);
            backdrop-filter: blur(4px);
        }
        .status-area {
            font-size: 20px;
            font-weight: 600;
            background: rgba(31, 52, 63, 0.85);
            padding: 4px 28px;
            border-radius: 40px;
            color: #b3d9e8;
            box-shadow: inset 0 2px 8px rgba(0,0,0,0.4);
            transition: 0.2s;
            min-width: 150px;
            text-align: center;
            backdrop-filter: blur(4px);
        }
        .status-area.highlight {
            color: #f1c40f;
            background: rgba(31, 52, 63, 0.9);
            box-shadow: 0 0 30px rgba(241, 196, 15, 0.15), inset 0 2px 8px rgba(0,0,0,0.4);
        }
        .status-area.fail {
            color: #ff6b6b;
            background: rgba(31, 52, 63, 0.9);
            box-shadow: 0 0 30px rgba(255, 50, 50, 0.15);
        }
        .btn-start {
            background: #27ae60;
            border: none;
            padding: 8px 32px;
            border-radius: 40px;
            font-weight: 700;
            font-size: 18px;
            color: #fff;
            cursor: pointer;
            box-shadow: 0 6px 0 #1a6e3f, 0 6px 20px rgba(0,0,0,0.3);
            transition: 0.06s linear;
            letter-spacing: 1px;
            border: 1px solid rgba(255,255,255,0.08);
        }
        .btn-start:active {
            transform: translateY(6px);
            box-shadow: 0 0px 0 #1a6e3f;
        }
        .btn-start:disabled {
            opacity: 0.4;
            transform: translateY(4px);
            box-shadow: 0 2px 0 #1a6e3f;
            pointer-events: none;
        }

        @media (max-width: 860px) {
            .scene { width: 100%; height: auto; aspect-ratio: 4/3; }
            .ui-header .brand { font-size: 13px; padding: 2px 14px; }
            .ui-header .timer { font-size: 20px; padding: 2px 14px; min-width: 50px; }
            .score-area { font-size: 20px; line-height: 38px; padding: 0 16px; }
            .status-area { font-size: 15px; padding: 2px 16px; min-width: 90px; }
            .btn-start { font-size: 14px; padding: 6px 18px; }
            .arrow-indicator { font-size: 28px; }
            .arrow-indicator span { font-size: 12px; padding: 4px 12px; }
            #customCursor { width: 75px; height: 75px; }
            .hit-text { 
                font-size: 48px;
                letter-spacing: 2px;
            }
        }
        @media (max-width: 480px) {
            .hit-text { 
                font-size: 36px;
                letter-spacing: 1px;
            }
        }
    </style>
</head>
<body>

<div id="customCursor">
    <div class="motion-lines"></div>
    <svg viewBox="0 0 280 280" xmlns="http://www.w3.org/2000/svg">
        <g transform="translate(125, 150) rotate(55)">
            <line x1="0" y1="0" x2="-20" y2="20" stroke="#feca57" stroke-width="8" stroke-linecap="round" />
            <ellipse cx="35" cy="-40" rx="35" ry="60" fill="none" stroke="#feca57" stroke-width="8" />
            <line x1="0" y1="-40" x2="70" y2="-40" stroke="rgba(255,255,255,0.6)" stroke-width="2.5" />
            <line x1="0" y1="-60" x2="70" y2="-60" stroke="rgba(255,255,255,0.6)" stroke-width="2.5" />
            <line x1="0" y1="-20" x2="70" y2="-20" stroke="rgba(255,255,255,0.6)" stroke-width="2.5" />
            <line x1="35" y1="-100" x2="35" y2="20" stroke="rgba(255,255,255,0.6)" stroke-width="2.5" />
            <line x1="15" y1="-90" x2="15" y2="10" stroke="rgba(255,255,255,0.6)" stroke-width="2.5" />
            <line x1="55" y1="-90" x2="55" y2="10" stroke="rgba(255,255,255,0.6)" stroke-width="2.5" />
        </g>
        <circle cx="80" cy="50" r="15" fill="none" stroke="#fff" stroke-width="4" />
        <path d="M 65 45 L 95 45" stroke="#ff4757" stroke-width="5" stroke-linecap="round" />
        <path d="M 60 70 C 70 90, 90 90, 100 70 L 95 150 C 85 160, 75 160, 65 150 Z" fill="none" stroke="#fff" stroke-width="6" stroke-linejoin="round" />
        <path d="M 75 80 L 80 140" stroke="rgba(255,255,255,0.4)" stroke-width="3" stroke-dasharray="5,5" />
        <path d="M 65 80 Q 30 110 40 140" fill="none" stroke="#fff" stroke-width="6" stroke-linecap="round" />
        <path d="M 35 135 L 45 145" fill="none" stroke="#fff" stroke-width="6" stroke-linecap="round" />
        <path d="M 95 80 Q 125 100 125 150" fill="none" stroke="#fff" stroke-width="6" stroke-linecap="round" />
        <path d="M 125 150 Q 135 140 145 155" fill="none" stroke="rgba(255,255,255,0.6)" stroke-width="4" stroke-linecap="round" />
        <path d="M 75 150 Q 60 190 50 230 L 45 250" fill="none" stroke="#fff" stroke-width="8" stroke-linecap="round" />
        <path d="M 85 150 Q 100 190 105 230 L 110 250" fill="none" stroke="#fff" stroke-width="8" stroke-linecap="round" />
        <path d="M 40 250 L 55 250 L 50 260 L 35 260 Z" fill="#fff" />
        <path d="M 100 250 L 115 250 L 120 260 L 105 260 Z" fill="#fff" />
    </svg>
</div>

<div class="scene" id="scene">
    <div class="court-surround">
        <div class="blue-court">
            <div class="service-line"></div>
            <div class="center-line"></div>
        </div>
        <div class="fade-out"></div>
    </div>

    <div class="machine" id="machine"></div>
    <div class="arrow-indicator" id="arrowIndicator">▼<span>移动到此处</span></div>
    <canvas class="trail-canvas" id="trailCanvas"></canvas>
    <div class="tennis-ball" id="tennisBall"></div>
    <div class="target-layer" id="targetLayer"></div>

    <div class="hit-text" id="hitText">🏏 击球！</div>

    <div class="ui-container">
        <div class="ui-header">
            <span class="brand">🎾 Understand. Predict. Train.</span>
            <span class="timer" id="timerDisplay">30s</span>
        </div>
        <div class="ui-footer">
            <div class="score-area">🏆 <span id="scoreDisplay">0</span></div>
            <div class="status-area" id="statusDisplay">⏸ 准备开始</div>
            <button class="btn-start" id="btnStart">▶ 开始训练</button>
        </div>
    </div>
</div>

<script>
    (function(){
        // ================================================================
        // 自定义光标控制
        // ================================================================
        const customCursor = document.getElementById('customCursor');
        const hitText = document.getElementById('hitText');
        let cursorX = 0, cursorY = 0;
        let isGameActiveCursor = false;

        document.addEventListener('mousemove', function(e) {
            cursorX = e.clientX;
            cursorY = e.clientY;
            if (isGameActiveCursor) {
                customCursor.style.left = cursorX + 'px';
                customCursor.style.top = cursorY + 'px';
            }
        });
        document.addEventListener('mouseleave', function() {
            if (isGameActiveCursor) {
                customCursor.style.display = 'none';
            }
        });
        document.addEventListener('mouseenter', function() {
            if (isGameActiveCursor) {
                customCursor.style.display = 'block';
            }
        });

        function setCursorMode(gameActive) {
            isGameActiveCursor = gameActive;
            if (gameActive) {
                document.body.classList.add('game-active');
                customCursor.style.display = 'block';
                customCursor.style.left = cursorX + 'px';
                customCursor.style.top = cursorY + 'px';
            } else {
                document.body.classList.remove('game-active');
                customCursor.style.display = 'none';
            }
        }

        function triggerHitAnimation() {
            customCursor.classList.remove('hit');
            void customCursor.offsetWidth;
            customCursor.classList.add('hit');
            setTimeout(() => {
                customCursor.classList.remove('hit');
            }, 400);
        }

        // ================================================================
        // 击球文字提示
        // ================================================================
        function showHitText(x, y) {
            const rect = scene.getBoundingClientRect();
            const scaleX = rect.width / scene.clientWidth;
            const scaleY = rect.height / scene.clientHeight;
            const pageX = rect.left + x * scaleX;
            const pageY = rect.top + y * scaleY;
            
            hitText.style.left = pageX + 'px';
            hitText.style.top = pageY + 'px';
            hitText.classList.remove('show');
            void hitText.offsetWidth;
            hitText.classList.add('show');
            
            setTimeout(() => {
                hitText.classList.remove('show');
            }, 750);
        }

        // ================================================================
        // 碰撞检测
        // ================================================================
        function isCursorOverlappingDot(cursorX, cursorY, dotX, dotY, dotRadius) {
            const cursorRadius = 40;
            const dx = cursorX - dotX;
            const dy = cursorY - dotY;
            const distance = Math.sqrt(dx * dx + dy * dy);
            return distance < (cursorRadius + dotRadius);
        }

        // ================================================================
        // 灯泡布局
        // ================================================================
        const COURT_TOP = 50;
        const COURT_BOTTOM = 550;
        const COURT_LEFT = 50;
        const COURT_RIGHT = 750;
        
        function perspectiveProject(px, py) {
            const nx = (px - COURT_LEFT) / (COURT_RIGHT - COURT_LEFT);
            const ny = (py - COURT_TOP) / (COURT_BOTTOM - COURT_TOP);
            const perspectiveFactor = 0.50 + 0.50 * ny;
            const cx = 0.5 + (nx - 0.5) * perspectiveFactor;
            const cy = ny * ny * 0.15 + ny * 0.85;
            return {
                x: COURT_LEFT + cx * (COURT_RIGHT - COURT_LEFT),
                y: COURT_TOP + cy * (COURT_BOTTOM - COURT_TOP)
            };
        }
        
        const rows = 3;
        const cols = 4;
        const rawPositions = [];
        const yStart = 0.60;
        const yEnd = 0.92;
        const xStart = 0.10;
        const xEnd = 0.90;
        
        for (let r = 0; r < rows; r++) {
            for (let c = 0; c < cols; c++) {
                let py = yStart + (r / (rows - 1)) * (yEnd - yStart);
                let px = xStart + (c / (cols - 1)) * (xEnd - xStart);
                const rawX = COURT_LEFT + px * (COURT_RIGHT - COURT_LEFT);
                const rawY = COURT_TOP + py * (COURT_BOTTOM - COURT_TOP);
                rawPositions.push({ x: rawX, y: rawY });
            }
        }
        
        const DOT_POSITIONS = rawPositions.map((pos) => {
            const proj = perspectiveProject(pos.x, pos.y);
            const ny = (pos.y - COURT_TOP) / (COURT_BOTTOM - COURT_TOP);
            const r = 14 + ny * 16;
            return {
                x: proj.x,
                y: proj.y,
                r: Math.round(r)
            };
        });
        
        DOT_POSITIONS.forEach((pos) => {
            const marginPx = 15;
            pos.x = Math.max(COURT_LEFT + marginPx, Math.min(COURT_RIGHT - marginPx, pos.x));
            pos.y = Math.max(COURT_TOP + marginPx, Math.min(COURT_BOTTOM - marginPx, pos.y));
        });

        // DOM refs
        const scene = document.getElementById('scene');
        const targetLayer = document.getElementById('targetLayer');
        const arrowIndicator = document.getElementById('arrowIndicator');
        const tennisBall = document.getElementById('tennisBall');
        const trailCanvas = document.getElementById('trailCanvas');
        const ctx = trailCanvas.getContext('2d');
        const machine = document.getElementById('machine');
        const scoreSpan = document.getElementById('scoreDisplay');
        const statusDisplay = document.getElementById('statusDisplay');
        const timerDisplay = document.getElementById('timerDisplay');
        const btnStart = document.getElementById('btnStart');

        let dotElements = [];
        let currentTargetIndex = -1;
        let isGameActive = false;
        let isWaitingForHover = false;
        let isFiring = false;
        let isHitAnimating = false;
        let isWaitingForClick = false;
        let clickTargetIndex = -1;
        let score = 0;
        let timeLeft = 30;
        let timerInterval = null;
        let lightTimeout = null;
        let roundTimeout = null;
        let clickTimeout = null;
        let ballAnimId = null;
        let hitAnimId = null;
        let hoverCheckInterval = null;
        let audioCtx = null;

        const ROUND_TIMEOUT = 4000;
        const CLICK_TIMEOUT = 4000;

        function initAudio() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
        }

        function playBeep(freq = 800, duration = 100, type = 'sine') {
            try {
                initAudio();
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = type;
                osc.frequency.value = freq;
                gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration/1000);
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start();
                osc.stop(audioCtx.currentTime + duration/1000);
            } catch(e) {}
        }

        function playSuccess() {
            playBeep(1200, 100);
            setTimeout(() => playBeep(1500, 80), 120);
        }

        function playClick() {
            playBeep(650, 70);
        }

        function playFail() {
            playBeep(300, 150, 'sawtooth');
        }

        function buildDots() {
            targetLayer.innerHTML = '';
            dotElements = [];
            DOT_POSITIONS.forEach((pos, idx) => {
                const dot = document.createElement('div');
                dot.className = 'target-dot hidden-dot';
                const size = pos.r * 2;
                dot.style.width = size + 'px';
                dot.style.height = size + 'px';
                dot.style.left = pos.x + 'px';
                dot.style.top = pos.y + 'px';
                dot.dataset.index = idx;
                targetLayer.appendChild(dot);
                dotElements.push(dot);
            });
        }

        function resetDots() {
            dotElements.forEach((dot) => {
                dot.className = 'target-dot hidden-dot';
                dot.style.animation = 'none';
            });
        }

        // ================================================================
        // 全局点击处理 - 任意位置点击触发击球
        // ================================================================
        function handleGlobalClick(e) {
            // 忽略按钮点击
            if (e.target.closest('.btn-start')) return;
            
            if (!isGameActive || !isWaitingForClick || isFiring || isHitAnimating) {
                playBeep(300, 50);
                return;
            }
            
            // 触发击球！
            playClick();
            isWaitingForClick = false;
            isFiring = true;
            
            if (clickTimeout) {
                clearTimeout(clickTimeout);
                clickTimeout = null;
            }
            
            const pos = DOT_POSITIONS[clickTargetIndex];
            statusDisplay.textContent = '🏏 击球！';
            statusDisplay.className = 'status-area highlight';
            
            // 触发击球动画（从灯光位置向发球机方向）
            setTimeout(() => {
                if (isGameActive) {
                    playHitAnimation(pos.x, pos.y);
                }
            }, 200);
        }

        // ================================================================
        // 击球动画
        // ================================================================
        function playHitAnimation(ballX, ballY) {
            if (isHitAnimating) return;
            isHitAnimating = true;

            showHitText(ballX, ballY);
            triggerHitAnimation();
            playBeep(600, 80, 'square');
            playBeep(900, 60, 'square');

            const startX = ballX;
            const startY = ballY;
            const endX = 400;
            const endY = 75;

            const steps = 30;
            const points = [];
            for (let i = 0; i <= steps; i++) {
                const t = i / steps;
                const x = startX + (endX - startX) * t;
                const y = startY + (endY - startY) * t - 60 * Math.sin(t * Math.PI);
                points.push({ x, y });
            }

            ctx.beginPath();
            ctx.setLineDash([6, 8]);
            ctx.strokeStyle = '#ffffff';
            ctx.lineWidth = 2.5;
            ctx.shadowColor = '#ffffff';
            ctx.shadowBlur = 20;
            ctx.moveTo(points[0].x, points[0].y);
            for (let i = 1; i < points.length; i++) {
                ctx.lineTo(points[i].x, points[i].y);
            }
            ctx.stroke();
            ctx.setLineDash([]);
            ctx.shadowBlur = 0;

            let progress = 0;
            if (hitAnimId) cancelAnimationFrame(hitAnimId);

            function animateHitBall() {
                progress += 0.04;
                if (progress >= 1) {
                    progress = 1;
                    const p = points[points.length-1];
                    tennisBall.style.left = p.x + 'px';
                    tennisBall.style.top = p.y + 'px';
                    setTimeout(() => {
                        tennisBall.style.display = 'none';
                        ctx.clearRect(0, 0, trailCanvas.width, trailCanvas.height);
                        isHitAnimating = false;
                        isFiring = false;
                        // 击球得分
                        score += 2;
                        scoreSpan.textContent = score;
                        statusDisplay.textContent = '💥 击球得分 +2';
                        statusDisplay.className = 'status-area highlight';
                        if (isGameActive) {
                            scheduleNextLight(2000);
                        }
                    }, 200);
                    hitAnimId = null;
                    return;
                }

                const idx = Math.floor(progress * steps);
                const p1 = points[idx] || points[0];
                const p2 = points[Math.min(idx+1, points.length-1)] || points[points.length-1];
                const mix = (progress * steps) % 1;
                const cx = p1.x + (p2.x - p1.x) * mix;
                const cy = p1.y + (p2.y - p1.y) * mix;
                tennisBall.style.left = cx + 'px';
                tennisBall.style.top = cy + 'px';
                hitAnimId = requestAnimationFrame(animateHitBall);
            }
            hitAnimId = requestAnimationFrame(animateHitBall);

            setTimeout(() => {
                if (isHitAnimating) {
                    ctx.clearRect(0, 0, trailCanvas.width, trailCanvas.height);
                    tennisBall.style.display = 'none';
                    if (hitAnimId) {
                        cancelAnimationFrame(hitAnimId);
                        hitAnimId = null;
                    }
                    isHitAnimating = false;
                    isFiring = false;
                    if (isGameActive) {
                        scheduleNextLight(2000);
                    }
                }
            }, 1000);
        }

        // ================================================================
        // 发球机发球
        // ================================================================
        function fireBall(targetIndex) {
            const pos = DOT_POSITIONS[targetIndex];
            const startX = 400, startY = 75;
            const endX = pos.x, endY = pos.y;

            machine.classList.add('firing');
            playBeep(280, 200, 'square');

            tennisBall.style.display = 'block';
            tennisBall.style.left = startX + 'px';
            tennisBall.style.top = startY + 'px';

            ctx.clearRect(0, 0, trailCanvas.width, trailCanvas.height);

            const steps = 40;
            const points = [];
            for (let i = 0; i <= steps; i++) {
                const t = i / steps;
                const x = startX + (endX - startX) * t;
                const y = startY + (endY - startY) * t - 85 * Math.sin(t * Math.PI);
                points.push({ x, y });
            }

            ctx.beginPath();
            ctx.setLineDash([8, 10]);
            ctx.strokeStyle = '#f1c40f';
            ctx.lineWidth = 3;
            ctx.shadowColor = '#f1c40f';
            ctx.shadowBlur = 16;
            ctx.moveTo(points[0].x, points[0].y);
            for (let i = 1; i < points.length; i++) {
                ctx.lineTo(points[i].x, points[i].y);
            }
            ctx.stroke();
            ctx.setLineDash([]);
            ctx.shadowBlur = 0;

            let progress = 0;
            if (ballAnimId) cancelAnimationFrame(ballAnimId);

            function animateBall() {
                progress += 0.025;
                if (progress >= 1) {
                    progress = 1;
                    const p = points[points.length-1];
                    tennisBall.style.left = p.x + 'px';
                    tennisBall.style.top = p.y + 'px';

                    machine.classList.remove('firing');
                    
                    // 灯泡保持绿色，但不再闪烁
                    const dot = dotElements[targetIndex];
                    if (dot) {
                        dot.className = 'target-dot arrived';
                        dot.style.animation = 'none';
                    }
                    
                    // 进入点击等待状态 - 任意位置点击触发击球
                    statusDisplay.textContent = '🖱️ 任意位置点击击球！';
                    statusDisplay.className = 'status-area highlight';
                    
                    // 清除发球轨迹
                    setTimeout(() => {
                        ctx.clearRect(0, 0, trailCanvas.width, trailCanvas.height);
                    }, 300);

                    isWaitingForClick = true;
                    clickTargetIndex = targetIndex;
                    isFiring = false;

                    // 设置点击超时
                    if (clickTimeout) {
                        clearTimeout(clickTimeout);
                    }
                    clickTimeout = setTimeout(() => {
                        if (isGameActive && isWaitingForClick) {
                            // 点击超时，不得击球分
                            isWaitingForClick = false;
                            const dot2 = dotElements[targetIndex];
                            if (dot2) {
                                dot2.className = 'target-dot timeout';
                                dot2.style.animation = 'none';
                            }
                            statusDisplay.textContent = '⏰ 未击球！';
                            statusDisplay.className = 'status-area fail';
                            playFail();
                            setTimeout(() => {
                                if (isGameActive && !isFiring && !isHitAnimating) {
                                    resetDots();
                                    scheduleNextLight(2000);
                                }
                            }, 1500);
                        }
                    }, CLICK_TIMEOUT);

                    ballAnimId = null;
                    return;
                }

                const idx = Math.floor(progress * steps);
                const p1 = points[idx] || points[0];
                const p2 = points[Math.min(idx+1, points.length-1)] || points[points.length-1];
                const mix = (progress * steps) % 1;
                const cx = p1.x + (p2.x - p1.x) * mix;
                const cy = p1.y + (p2.y - p1.y) * mix;
                tennisBall.style.left = cx + 'px';
                tennisBall.style.top = cy + 'px';
                ballAnimId = requestAnimationFrame(animateBall);
            }
            ballAnimId = requestAnimationFrame(animateBall);
        }

        // ================================================================
        // 悬停检测
        // ================================================================
        function startHoverDetection() {
            if (hoverCheckInterval) {
                clearInterval(hoverCheckInterval);
            }
            hoverCheckInterval = setInterval(() => {
                if (!isGameActive || isFiring || !isWaitingForHover || currentTargetIndex < 0) {
                    return;
                }
                const pos = DOT_POSITIONS[currentTargetIndex];
                const rect = scene.getBoundingClientRect();
                const scaleX = scene.clientWidth / rect.width;
                const scaleY = scene.clientHeight / rect.height;
                const sceneX = (cursorX - rect.left) * scaleX;
                const sceneY = (cursorY - rect.top) * scaleY;
                
                if (isCursorOverlappingDot(sceneX, sceneY, pos.x, pos.y, pos.r)) {
                    triggerArrived(currentTargetIndex);
                }
            }, 50);
        }

        function stopHoverDetection() {
            if (hoverCheckInterval) {
                clearInterval(hoverCheckInterval);
                hoverCheckInterval = null;
            }
        }

        // ================================================================
        // 超时处理 (悬停超时)
        // ================================================================
        function handleRoundTimeout() {
            if (!isGameActive || !isWaitingForHover || currentTargetIndex < 0) return;
            
            isWaitingForHover = false;
            stopHoverDetection();
            
            const dot = dotElements[currentTargetIndex];
            if (dot) {
                dot.className = 'target-dot timeout';
                dot.style.animation = 'none';
            }
            
            arrowIndicator.className = 'arrow-indicator';
            statusDisplay.textContent = '⏰ 超时！未到位';
            statusDisplay.className = 'status-area fail';
            playFail();
            
            if (roundTimeout) {
                clearTimeout(roundTimeout);
                roundTimeout = null;
            }
            
            setTimeout(() => {
                if (isGameActive && !isFiring && !isHitAnimating && !isWaitingForClick) {
                    resetDots();
                    scheduleNextLight(2000);
                }
            }, 1500);
        }

        // ================================================================
        // 触发到位 (悬停触发)
        // ================================================================
        function triggerArrived(index) {
            if (!isGameActive || isFiring || !isWaitingForHover) return;
            if (index !== currentTargetIndex) return;
            
            if (roundTimeout) {
                clearTimeout(roundTimeout);
                roundTimeout = null;
            }
            
            playClick();
            isWaitingForHover = false;
            stopHoverDetection();

            const dot = dotElements[index];
            dot.className = 'target-dot arrived';
            dot.style.animation = 'none';
            arrowIndicator.className = 'arrow-indicator';

            // 到位得分
            score += 2;
            scoreSpan.textContent = score;
            
            statusDisplay.textContent = '✅ 到位！发球...';
            statusDisplay.className = 'status-area highlight';

            // 开始发球
            isFiring = true;
            setTimeout(() => {
                if (isGameActive) {
                    fireBall(index);
                }
            }, 400);
        }

        // ================================================================
        // 灯光控制
        // ================================================================
        function lightRandomDot() {
            if (!isGameActive || isFiring || isWaitingForHover || isHitAnimating || isWaitingForClick) {
                scheduleNextLight(500);
                return;
            }

            resetDots();

            const allIndices = Array.from({ length: DOT_POSITIONS.length }, (_, i) => i);
            const randIdx = allIndices[Math.floor(Math.random() * allIndices.length)];
            currentTargetIndex = randIdx;

            const dot = dotElements[randIdx];
            dot.className = 'target-dot active';
            dot.style.animation = 'none';
            void dot.offsetHeight;
            dot.style.animation = 'pulse-red 0.8s infinite alternate';

            const pos = DOT_POSITIONS[randIdx];
            arrowIndicator.style.left = pos.x + 'px';
            arrowIndicator.style.top = (pos.y - 50) + 'px';
            arrowIndicator.className = 'arrow-indicator show';

            statusDisplay.textContent = '🎯 移动到此处';
            statusDisplay.className = 'status-area highlight';
            isWaitingForHover = true;

            startHoverDetection();
            playBeep(500, 60);

            if (roundTimeout) {
                clearTimeout(roundTimeout);
            }
            roundTimeout = setTimeout(() => {
                handleRoundTimeout();
            }, ROUND_TIMEOUT);
        }

        function scheduleNextLight(delay = 2000) {
            if (lightTimeout) {
                clearTimeout(lightTimeout);
                lightTimeout = null;
            }
            if (!isGameActive) return;
            lightTimeout = setTimeout(() => {
                lightTimeout = null;
                if (isGameActive && !isFiring && !isWaitingForHover && !isHitAnimating && !isWaitingForClick) {
                    lightRandomDot();
                } else {
                    scheduleNextLight(500);
                }
            }, delay);
        }

        // ================================================================
        // 游戏控制
        // ================================================================
        function startTimer() {
            timeLeft = 30;
            timerDisplay.textContent = '30s';
            timerInterval = setInterval(() => {
                timeLeft--;
                timerDisplay.textContent = timeLeft + 's';
                if (timeLeft <= 0) {
                    endGame();
                }
            }, 1000);
        }

        function startGame() {
            if (isGameActive) {
                resetGame();
                return;
            }

            initAudio();
            isGameActive = true;
            isFiring = false;
            isWaitingForHover = false;
            isHitAnimating = false;
            isWaitingForClick = false;
            clickTargetIndex = -1;
            score = 0;
            scoreSpan.textContent = '0';
            btnStart.textContent = '🔄 重置';
            statusDisplay.textContent = '⏳ 准备';
            statusDisplay.className = 'status-area';
            timerDisplay.textContent = '30s';

            setCursorMode(true);

            resetDots();
            buildDots();
            currentTargetIndex = -1;
            arrowIndicator.className = 'arrow-indicator';
            tennisBall.style.display = 'none';
            machine.classList.remove('firing');
            ctx.clearRect(0, 0, trailCanvas.width, trailCanvas.height);

            // 注册全局点击事件
            document.addEventListener('click', handleGlobalClick);

            startTimer();

            setTimeout(() => {
                if (isGameActive) {
                    lightRandomDot();
                }
            }, 500);
        }

        function endGame() {
            isGameActive = false;
            isFiring = false;
            isWaitingForHover = false;
            isHitAnimating = false;
            isWaitingForClick = false;
            stopHoverDetection();
            document.removeEventListener('click', handleGlobalClick);
            if (roundTimeout) {
                clearTimeout(roundTimeout);
                roundTimeout = null;
            }
            if (clickTimeout) {
                clearTimeout(clickTimeout);
                clickTimeout = null;
            }
            if (timerInterval) clearInterval(timerInterval);
            if (lightTimeout) {
                clearTimeout(lightTimeout);
                lightTimeout = null;
            }
            if (ballAnimId) cancelAnimationFrame(ballAnimId);
            if (hitAnimId) cancelAnimationFrame(hitAnimId);

            setCursorMode(false);

            btnStart.textContent = '▶ 开始训练';
            statusDisplay.textContent = '⏹ 训练结束';
            statusDisplay.className = 'status-area';
            timerDisplay.textContent = '0s';
            arrowIndicator.className = 'arrow-indicator';
            resetDots();
            machine.classList.remove('firing');
            tennisBall.style.display = 'none';
            ctx.clearRect(0, 0, trailCanvas.width, trailCanvas.height);

            playBeep(400, 300, 'square');
            setTimeout(() => playBeep(300, 300, 'square'), 300);
        }

        function resetGame() {
            isGameActive = false;
            isFiring = false;
            isWaitingForHover = false;
            isHitAnimating = false;
            isWaitingForClick = false;
            clickTargetIndex = -1;
            stopHoverDetection();
            document.removeEventListener('click', handleGlobalClick);
            if (roundTimeout) {
                clearTimeout(roundTimeout);
                roundTimeout = null;
            }
            if (clickTimeout) {
                clearTimeout(clickTimeout);
                clickTimeout = null;
            }
            if (timerInterval) clearInterval(timerInterval);
            if (lightTimeout) {
                clearTimeout(lightTimeout);
                lightTimeout = null;
            }
            if (ballAnimId) cancelAnimationFrame(ballAnimId);
            if (hitAnimId) cancelAnimationFrame(hitAnimId);

            setCursorMode(false);

            btnStart.textContent = '▶ 开始训练';
            statusDisplay.textContent = '⏸ 已暂停';
            statusDisplay.className = 'status-area';
            timerDisplay.textContent = '30s';
            arrowIndicator.className = 'arrow-indicator';
            resetDots();
            buildDots();
            tennisBall.style.display = 'none';
            machine.classList.remove('firing');
            ctx.clearRect(0, 0, trailCanvas.width, trailCanvas.height);
            score = 0;
            scoreSpan.textContent = '0';
            currentTargetIndex = -1;
        }

        function resizeCanvas() {
            trailCanvas.width = scene.clientWidth;
            trailCanvas.height = scene.clientHeight;
        }
        window.addEventListener('resize', resizeCanvas);

        buildDots();
        resizeCanvas();
        btnStart.addEventListener('click', startGame);
        document.addEventListener('click', () => {
            if (!audioCtx) {
                initAudio();
            }
        }, { once: true });
    })();
</script>

</body>
</html>
