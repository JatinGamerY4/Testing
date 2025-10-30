<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Jatin Gamer App</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        body {
            background: linear-gradient(135deg, #1a2a6c, #b21f1f, #fdbb2d);
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
        }
        
        .app-container {
            background-color: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 15px 35px rgba(0, 0, 0, 0.2);
            text-align: center;
            max-width: 90%;
            width: 400px;
            border: 1px solid rgba(255, 255, 255, 0.2);
            animation: fadeIn 1.5s ease;
        }
        
        .logo-container {
            margin-bottom: 25px;
        }
        
        .logo {
            width: 150px;
            height: 150px;
            border-radius: 50%;
            object-fit: cover;
            border: 3px solid white;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
        }
        
        .made-by {
            font-size: 18px;
            color: white;
            margin-bottom: 5px;
            font-weight: 500;
            letter-spacing: 1px;
        }
        
        .name {
            font-size: 24px;
            color: #FFD700;
            font-weight: bold;
            text-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
            letter-spacing: 1px;
        }
        
        .loading-container {
            margin-top: 25px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        
        .loading-text {
            color: rgba(255, 255, 255, 0.8);
            margin-bottom: 10px;
            font-size: 14px;
        }
        
        .spinner {
            width: 40px;
            height: 40px;
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-radius: 50%;
            border-top-color: #FFD700;
            animation: spin 1s linear infinite;
            margin-bottom: 15px;
        }
        
        .manual-button {
            background: rgba(255, 255, 255, 0.2);
            border: 2px solid white;
            border-radius: 8px;
            color: white;
            padding: 8px 16px;
            font-size: 14px;
            cursor: pointer;
            margin-top: 10px;
            transition: all 0.3s ease;
        }
        
        .manual-button:hover {
            background: rgba(255, 255, 255, 0.3);
        }
        
        .levels-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(135deg, #1a2a6c, #b21f1f, #fdbb2d);
            background-size: cover;
            background-position: center;
            display: none;
            flex-direction: column;
            align-items: center;
            padding: 20px;
            overflow-y: auto;
            z-index: 100;
        }
        
        .levels-title {
            font-size: 48px;
            color: white;
            text-shadow: 0 4px 8px rgba(0, 0, 0, 0.5);
            margin: 20px 0 30px;
            font-weight: bold;
            letter-spacing: 2px;
            text-align: center;
        }
        
        .buttons-container {
            display: grid;
            grid-template-columns: repeat(5, 1fr);
            gap: 15px;
            max-width: 800px;
            width: 100%;
            padding: 0 20px;
        }
        
        .level-button {
            background: linear-gradient(145deg, #6a11cb, #2575fc);
            border: none;
            border-radius: 12px;
            color: white;
            font-size: 20px;
            font-weight: bold;
            height: 60px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
            transition: all 0.3s ease;
            position: relative;
        }
        
        .level-button.unlocked:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.4);
            background: linear-gradient(145deg, #2575fc, #6a11cb);
        }
        
        .level-button:active {
            transform: translateY(2px);
            box-shadow: 0 3px 6px rgba(0, 0, 0, 0.3);
        }
        
        .level-button.locked {
            background: linear-gradient(145deg, #444, #666);
            cursor: not-allowed;
            opacity: 0.7;
        }
        
        .lock-icon {
            font-size: 24px;
        }
        
        .level-complete-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 200;
            color: white;
            text-align: center;
        }
        
        .complete-title {
            font-size: 48px;
            color: #FFD700;
            margin-bottom: 20px;
            text-shadow: 0 2px 10px rgba(0, 0, 0, 0.5);
        }
        
        .complete-message {
            font-size: 24px;
            margin-bottom: 30px;
        }
        
        .continue-button {
            background: linear-gradient(145deg, #6a11cb, #2575fc);
            border: none;
            border-radius: 12px;
            color: white;
            font-size: 20px;
            font-weight: bold;
            padding: 15px 30px;
            cursor: pointer;
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
            transition: all 0.3s ease;
        }
        
        .continue-button:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.4);
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        @media (max-width: 768px) {
            .buttons-container {
                grid-template-columns: repeat(3, 1fr);
            }
            
            .levels-title {
                font-size: 36px;
            }
        }
        
        @media (max-width: 480px) {
            .app-container {
                padding: 30px 20px;
            }
            
            .logo {
                width: 120px;
                height: 120px;
            }
            
            .name {
                font-size: 20px;
            }
            
            .made-by {
                font-size: 16px;
            }
            
            .buttons-container {
                grid-template-columns: repeat(2, 1fr);
            }
            
            .level-button {
                height: 50px;
                font-size: 18px;
            }
            
            .levels-title {
                font-size: 32px;
            }
        }
    </style>
</head>
<body>
    <div class="app-container" id="appContainer">
        <div class="logo-container">
            <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQbLZw-MSjxlZwgpehamid6kPvR-zDXp_nLC0oRHBDERe1S-IsLODtn88LA&s=10" 
                 alt="App Logo" class="logo">
        </div>
        <div class="text-container">
            <div class="made-by">MADE BY</div>
            <div class="name">JATIN GAMER Y4</div>
        </div>
        <div class="loading-container">
            <div class="loading-text">Loading App...</div>
            <div class="spinner"></div>
            <button class="manual-button" id="manualButton">Click if stuck on loading</button>
        </div>
    </div>
    
    <div class="levels-screen" id="levelsScreen">
        <h1 class="levels-title">LEVELS</h1>
        <div class="buttons-container" id="buttonsContainer">
            <!-- Buttons will be generated by JavaScript -->
        </div>
    </div>
    
    <div class="level-complete-screen" id="levelCompleteScreen">
        <h2 class="complete-title">LEVEL COMPLETED!</h2>
        <p class="complete-message" id="completeMessage">Congratulations! You've completed level 1.</p>
        <button class="continue-button" id="continueButton">Continue</button>
    </div>

    <script>
        // Simple player progress with fallback
        let playerProgress = { highestLevel: 1 };
        
        // Try to load from localStorage, but don't break if it fails
        try {
            const savedProgress = localStorage.getItem('jatinGamerProgress');
            if (savedProgress) {
                playerProgress = JSON.parse(savedProgress);
            }
        } catch (e) {
            // If localStorage fails, use default progress
            console.log("Using default progress");
        }
        
        // Set up manual transition button
        document.getElementById('manualButton').addEventListener('click', function() {
            transitionToLevels();
        });
        
        // Auto transition after 5-8 seconds
        const randomTime = Math.floor(Math.random() * 3000) + 5000;
        setTimeout(transitionToLevels, randomTime);
        
        function transitionToLevels() {
            const appContainer = document.getElementById('appContainer');
            const levelsScreen = document.getElementById('levelsScreen');
            
            // Fade out the loading screen
            appContainer.style.opacity = '0';
            
            // Show levels screen after fade out
            setTimeout(function() {
                appContainer.style.display = 'none';
                levelsScreen.style.display = 'flex';
                createLevelButtons();
            }, 1000);
        }
        
        function createLevelButtons() {
            const buttonsContainer = document.getElementById('buttonsContainer');
            buttonsContainer.innerHTML = '';
            
            for (let i = 1; i <= 50; i++) {
                const button = document.createElement('button');
                button.className = 'level-button';
                
                if (i <= playerProgress.highestLevel) {
                    // Unlocked level
                    button.classList.add('unlocked');
                    button.textContent = i;
                    button.addEventListener('click', function() {
                        startLevel(i);
                    });
                } else {
                    // Locked level
                    button.classList.add('locked');
                    button.innerHTML = '<span class="lock-icon">🔒</span>';
                    button.disabled = true;
                }
                
                buttonsContainer.appendChild(button);
            }
        }
        
        function startLevel(level) {
            // Simulate level completion after a short delay
            setTimeout(function() {
                // Show level complete screen
                document.getElementById('completeMessage').textContent = 
                    `Congratulations! You've completed level ${level}.`;
                document.getElementById('levelCompleteScreen').style.display = 'flex';
                
                // Update progress if this is the highest level completed
                if (level === playerProgress.highestLevel) {
                    playerProgress.highestLevel = level + 1;
                    
                    // Try to save to localStorage, but don't break if it fails
                    try {
                        localStorage.setItem('jatinGamerProgress', JSON.stringify(playerProgress));
                    } catch (e) {
                        console.log("Could not save progress to localStorage");
                    }
                }
            }, 1500);
        }
        
        // Continue button handler
        document.getElementById('continueButton').addEventListener('click', function() {
            document.getElementById('levelCompleteScreen').style.display = 'none';
            createLevelButtons(); // Refresh buttons to show newly unlocked levels
        });
    </script>
</body>
</html>
