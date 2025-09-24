<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Landscape Background & Cursor</title>
<style>
  html, body {
    margin: 0;
    padding: 0;
    height: 100vh;
    width: 100vw;
    overflow: hidden;
    cursor: none; /* hide default cursor */
  }

  body {
    /* Landscape fit */
    background: url('https://winblogs.thesourcemediaassets.com/sites/2/2021/10/Windows-11-Bloom-Screensaver-Dark-scaled.jpg') no-repeat center center;
    background-size: cover;
  }

  #imageCursor {
    width: 40px;
    height: 40px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    pointer-events: none;
    background: url('https://www.pngmart.com/files/3/Cursor-Arrow-Transparent-Background.png') no-repeat center center;
    background-size: contain;
  }
</style>
</head>
<body>

<div id="imageCursor"></div>

<script>
const cursor = document.getElementById('imageCursor');

let cursorX = window.innerWidth / 2;
let cursorY = window.innerHeight / 2;

let lastMouseX = null;
let lastMouseY = null;

const speedFactor = 0.6; // balanced speed

// Mouse movement
document.addEventListener('mousemove', (e) => {
    if (lastMouseX !== null && lastMouseY !== null) {
        let deltaX = (e.clientX - lastMouseX) * speedFactor;
        let deltaY = (e.clientY - lastMouseY) * speedFactor;
        cursorX += deltaX;
        cursorY += deltaY;
        cursor.style.left = cursorX + 'px';
        cursor.style.top = cursorY + 'px';
    }
    lastMouseX = e.clientX;
    lastMouseY = e.clientY;
});

// Touch movement
document.addEventListener('touchmove', (e) => {
    e.preventDefault();
    const touch = e.touches[0];
    if (lastMouseX !== null && lastMouseY !== null) {
        let deltaX = (touch.clientX - lastMouseX) * speedFactor;
        let deltaY = (touch.clientY - lastMouseY) * speedFactor;
        cursorX += deltaX;
        cursorY += deltaY;
        cursor.style.left = cursorX + 'px';
        cursor.style.top = cursorY + 'px';
    }
    lastMouseX = touch.clientX;
    lastMouseY = touch.clientY;
}, {passive: false});

// Reset last positions on mouseup/touchend
document.addEventListener('mouseup', () => { lastMouseX = null; lastMouseY = null; });
document.addEventListener('touchend', () => { lastMouseX = null; lastMouseY = null; });
</script>

</body>
</html>
