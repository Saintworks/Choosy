<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Universal Neon Thumb Chooser Game</title>

<style>
body {
    margin: 0;
    height: 100vh;
    overflow: hidden;
    font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
    /* Animated Gradient Backdrop */
    background: linear-gradient(125deg, #120c1f, #1a0b2e, #0b1b3a, #052c2e);
    background-size: 400% 400%;
    animation: fluidGradient 15s ease infinite;
    color: white;
    user-select: none;
    -webkit-user-select: none;
    touch-action: none; 
}

/* Background Fluid Animation */
@keyframes fluidGradient {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
}

/* Decorative Ambient Floating Orb 1 */
body::before {
    content: '';
    position: absolute;
    width: 300px;
    height: 300px;
    background: radial-gradient(circle, rgba(255, 0, 128, 0.2) 0%, transparent 70%);
    top: 10%;
    left: -5%;
    border-radius: 50%;
    z-index: 0;
    pointer-events: none;
    animation: floatOrb 12s ease-in-out infinite alternate;
}

/* Decorative Ambient Floating Orb 2 */
body::after {
    content: '';
    position: absolute;
    width: 400px;
    height: 400px;
    background: radial-gradient(circle, rgba(0, 234, 255, 0.15) 0%, transparent 70%);
    bottom: 5%;
    right: -10%;
    border-radius: 50%;
    z-index: 0;
    pointer-events: none;
    animation: floatOrb 18s ease-in-out infinite alternate-reverse;
}

@keyframes floatOrb {
    0% { transform: translate(0, 0) scale(1); }
    100% { transform: translate(40px, 60px) scale(1.2); }
}

/* Glassmorphism Status Banner */
#status {
    position: absolute;
    top: 30px;
    left: 5%;
    width: 90%;
    padding: 16px 0;
    text-align: center;
    font-size: 22px;
    font-weight: 800;
    z-index: 100;
    text-transform: uppercase;
    letter-spacing: 2px;
    background: rgba(255, 255, 255, 0.04);
    backdrop-filter: blur(12px);
    -webkit-backdrop-filter: blur(12px);
    border-radius: 16px;
    border: 1px solid rgba(255, 255, 255, 0.1);
    box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.3);
    text-shadow: 0 2px 10px rgba(255, 255, 255, 0.2);
    transition: all 0.3s ease;
}

/* Vibrantly Glowing Touch Points */
.touch-point {
    position: absolute;
    width: 95px;
    height: 95px;
    border-radius: 50%;
    transform: translate(-50%, -50%) scale(1);
    border: 3px solid rgba(255, 255, 255, 0.85);
    box-shadow: 0 0 15px currentColor, inset 0 0 15px currentColor;
    transition: transform 0.15s cubic-bezier(0.175, 0.885, 0.32, 1.275), opacity 0.3s ease;
    pointer-events: none; 
    z-index: 10;
}

.holding {
    animation: pulse 0.8s infinite alternate ease-in-out;
}

@keyframes pulse {
    0% { transform: translate(-50%, -50%) scale(1); box-shadow: 0 0 20px currentColor, inset 0 0 10px currentColor; }
    100% { transform: translate(-50%, -50%) scale(1.18); box-shadow: 0 0 35px currentColor, inset 0 0 20px currentColor; }
}

/* Dramatic Winner State Override */
.winner {
    background: #00ff87 !important;
    color: #00ff87 !important;
    transform: translate(-50%, -50%) scale(1.65) !important;
    border-color: #ffffff;
    box-shadow: 0 0 50px #00ff87, 0 0 20px #00ff87, inset 0 0 15px #ffffff !important;
    z-index: 99;
    animation: celebrate 0.5s ease infinite alternate !important;
}

@keyframes celebrate {
    0% { transform: translate(-50%, -50%) scale(1.65); }
    100% { transform: translate(-50%, -50%) scale(1.75); }
}

/* Smooth Fade-out for Losers */
.loser {
    opacity: 0.08;
    transform: translate(-50%, -50%) scale(0.65);
    filter: blur(2px);
}
</style>
</head>
<body>

<div id="status">Place 2+ fingers on screen</div>

<script>
const statusText = document.getElementById("status");

// Ultra-bright neon colors
const colors = [
    '#ff0055', // Neon Pink
    '#00eaff', // Cyan Blue
    '#ffaa00', // Neon Gold
    '#a020f0', // Electric Purple
    '#ff5500', // Neon Orange
    '#00ffaa', // Mint Spark
    '#ff00aa', // Magenta Magic
    '#0088ff'  // Deep Electric Blue
];

const activePoints = {};
let selectionTimer = null;
let isLocked = false; 

function updateGameState() {
    const count = Object.keys(activePoints).length;

    if (count === 0) {
        isLocked = false;
        if (selectionTimer) clearTimeout(selectionTimer);
        selectionTimer = null;
        statusText.style.borderColor = "rgba(255, 255, 255, 0.1)";
        statusText.style.boxShadow = "0 8px 32px 0 rgba(0, 0, 0, 0.3)";
        statusText.innerText = "Place 2+ fingers on screen";
        return;
    }

    if (isLocked) return; 

    if (count >= 2) {
        for (let id in activePoints) {
            activePoints[id].classList.add("holding");
        }

        if (!selectionTimer) {
            statusText.innerText = `Detected ${count} players. Hold still...`;
            startCountdown();
        } else {
            statusText.innerText = `Choosing from ${count} players...`;
        }
    } else {
        if (selectionTimer) {
            clearTimeout(selectionTimer);
            selectionTimer = null;
        }
        for (let id in activePoints) {
            activePoints[id].classList.remove("holding");
        }
        statusText.innerText = count === 1 ? "🔥 Need at least 2 players! 🔥" : "Place 2+ fingers on screen";
    }
}

function startCountdown() {
    selectionTimer = setTimeout(() => {
        const keys = Object.keys(activePoints);
        if (keys.length < 2) return; 

        const randomIndex = Math.floor(Math.random() * keys.length);
        const winnerId = keys[randomIndex];

        keys.forEach(id => {
            const dot = activePoints[id];
            if (!dot) return;
            
            dot.classList.remove("holding");
            if (id === winnerId) {
                dot.classList.add("winner");
            } else {
                dot.classList.add("loser");
            }
        });

        statusText.innerText = "🎉 Winner Selected! 🎉";
        statusText.style.borderColor = "#00ff87";
        statusText.style.boxShadow = "0 0 20px rgba(0, 255, 135, 0.4)";
        isLocked = true;
        selectionTimer = null;

    }, 2000); 
}

// --- OPTION A: NATIVE MOBILE TOUCH API ---
function handleTouch(event) {
    if (event.cancelable) event.preventDefault(); 
    
    if (!event.touches || event.touches.length === 0) {
        for (let id in activePoints) {
            if (activePoints[id]) activePoints[id].remove();
            delete activePoints[id];
        }
        updateGameState();
        return;
    }

    const liveIds = new Set();
    for (let i = 0; i < event.touches.length; i++) {
        const touch = event.touches[i];
        const stringId = String(touch.identifier);
        liveIds.add(stringId);

        if (!isLocked) {
            let dot = activePoints[stringId];
            if (!dot) {
                dot = document.createElement("div");
                dot.className = "touch-point";
                const choiceColor = colors[Object.keys(activePoints).length % colors.length];
                dot.style.color = choiceColor; // Dictates border/box-shadow color values via currentColor
                dot.style.backgroundColor = 'rgba(255, 255, 255, 0.03)';
                document.body.appendChild(dot);
                activePoints[stringId] = dot;
            }
            dot.style.left = touch.clientX + "px";
            dot.style.top = touch.clientY + "px";
        }
    }

    for (let id in activePoints) {
        if (!liveIds.has(String(id))) {
            if (activePoints[id]) activePoints[id].remove();
            delete activePoints[id];
        }
    }

    updateGameState();
}

// --- OPTION B: PC HYBRID POINTER API ---
const pointerMap = new Set();

function handlePointer(event) {
    if (event.pointerType === "mouse") return; 
    
    const pId = String(event.pointerId);

    if (event.type === "pointerdown" || event.type === "pointermove") {
        pointerMap.add(pId);
        
        if (!isLocked) {
            let dot = activePoints[pId];
            if (!dot) {
                dot = document.createElement("div");
                dot.className = "touch-point";
                const choiceColor = colors[Object.keys(activePoints).length % colors.length];
                dot.style.color = choiceColor;
                dot.style.backgroundColor = 'rgba(255, 255, 255, 0.03)';
                document.body.appendChild(dot);
                activePoints[pId] = dot;
            }
            dot.style.left = event.clientX + "px";
            dot.style.top = event.clientY + "px";
        }
    } else if (event.type === "pointerup" || event.type === "pointercancel") {
        pointerMap.delete(pId);
        if (activePoints[pId]) {
            activePoints[pId].remove();
            delete activePoints[pId];
        }
    }

    updateGameState();
}

// --- DEVICE DISPATCH HANDLER LINKER ---
if ('ontouchstart' in window) {
    document.addEventListener("touchstart", handleTouch, { passive: false });
    document.addEventListener("touchmove", handleTouch, { passive: false });
    document.addEventListener("touchend", handleTouch, { passive: false });
    document.addEventListener("touchcancel", handleTouch, { passive: false });
} else {
    document.addEventListener("pointerdown", handlePointer);
    document.addEventListener("pointermove", handlePointer);
    document.addEventListener("pointerup", handlePointer);
    document.addEventListener("pointercancel", handlePointer);
}

</script>
</body>
</html>
