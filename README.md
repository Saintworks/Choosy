# Choosy
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Thumb Chooser Game</title>

<style>
body {
    margin: 0;
    height: 100vh;
    overflow: hidden;
    font-family: Arial, sans-serif;
    background: #111;
    color: white;
    user-select: none;
    -webkit-user-select: none;
    touch-action: none; /* Prevents default browser panning/zooming entirely */
}

#status {
    position: absolute;
    top: 40px;
    width: 100%;
    text-align: center;
    font-size: 28px;
    font-weight: bold;
    z-index: 100;
    text-transform: uppercase;
    letter-spacing: 1px;
}

.touch-point {
    position: absolute;
    width: 90px;
    height: 90px;
    border-radius: 50%;
    transform: translate(-50%, -50%) scale(1);
    border: 4px solid white;
    box-shadow: 0 0 20px rgba(255,255,255,0.2);
    transition: transform 0.1s ease, opacity 0.3s ease;
}

.holding {
    animation: pulse 1s infinite alternate;
}

@keyframes pulse {
    0% { transform: translate(-50%, -50%) scale(1); }
    100% { transform: translate(-50%, -50%) scale(1.15); }
}

.winner {
    background: #2ecc71 !important;
    transform: translate(-50%, -50%) scale(1.6) !important;
    border-color: #fff;
    box-shadow: 0 0 40px #2ecc71;
    z-index: 99;
    animation: none !important;
}

.loser {
    opacity: 0.15;
    transform: translate(-50%, -50%) scale(0.7);
}
</style>
</head>
<body>

<div id="status">Place 2+ fingers on screen</div>

<script>
const statusText = document.getElementById("status");
const colors = ['#3498db', '#e74c3c', '#f1c40f', '#9b59b6', '#e67e22', '#1abc9c', '#00eaff', '#ff00bb'];

// Active tracking map for active fingers { identifier: DOMElement }
const activePoints = {};
let selectionTimer = null;
let isLocked = false; 

function handleTouch(event) {
    event.preventDefault(); 
    
    // 1. Reset state entirely when all fingers leave the screen
    if (event.touches.length === 0) {
        isLocked = false;
        if (selectionTimer) clearTimeout(selectionTimer);
        selectionTimer = null;
        
        // Remove left-over visual nodes
        for (let id in activePoints) {
            activePoints[id].remove();
            delete activePoints[id];
        }
        statusText.innerText = "Place 2+ fingers on screen";
        return;
    }

    // 2. Map all live fingers currently touching the hardware glass
    const liveIds = new Set();
    for (let i = 0; i < event.touches.length; i++) {
        const touch = event.touches[i];
        liveIds.add(touch.identifier.toString());

        // Only build and move points if the game isn't frozen on a winner screen
        if (!isLocked) {
            let dot = activePoints[touch.identifier];

            // If it's a new finger, build it
            if (!dot) {
                dot = document.createElement("div");
                dot.className = "touch-point";
                dot.style.backgroundColor = colors[Object.keys(activePoints).length % colors.length];
                document.body.appendChild(dot);
                activePoints[touch.identifier] = dot;
            }

            // Track position coordinates
            dot.style.left = touch.clientX + "px";
            dot.style.top = touch.clientY + "px";
        }
    }

    // 3. Clean up abandoned finger nodes immediately (even if locked!)
    for (let id in activePoints) {
        if (!liveIds.has(id.toString())) {
            activePoints[id].remove();
            delete activePoints[id];
        }
    }

    // If a winner was chosen and the screen is frozen, stop running countdown sequences
    if (isLocked) return; 

    // 4. Process Countdown Logic State
    const count = Object.keys(activePoints).length;

    if (count >= 2) {
        // Apply pulsing visual cues to indicate processing state
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
        // Kill timer if players drop below minimum
        if (selectionTimer) {
            clearTimeout(selectionTimer);
            selectionTimer = null;
        }
        for (let id in activePoints) {
            activePoints[id].classList.remove("holding");
        }
        statusText.innerText = count === 1 ? "Need at least 2 players!" : "Place 2+ fingers on screen";
    }
}

function startCountdown() {
    selectionTimer = setTimeout(() => {
        const keys = Object.keys(activePoints);
        if (keys.length < 2) return; // Edge case abort

        const randomIndex = Math.floor(Math.random() * keys.length);
        const winnerId = keys[randomIndex];

        // Apply distinct endpoint styling states
        keys.forEach(id => {
            const dot = activePoints[id];
            dot.classList.remove("holding");
            if (id === winnerId) {
                dot.classList.add("winner");
            } else {
                dot.classList.add("loser");
            }
        });

        statusText.innerText = "🎉 Winner Selected! 🎉";
        isLocked = true;
        selectionTimer = null;

    }, 2000); // 2 second countdown sequence
}

// Global Event Handlers
document.addEventListener("touchstart", handleTouch, { passive: false });
document.addEventListener("touchmove", handleTouch, { passive: false });
document.addEventListener("touchend", handleTouch, { passive: false });
document.addEventListener("touchcancel", handleTouch, { passive: false });

</script>
</body>
</html>
