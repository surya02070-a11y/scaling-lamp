
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Offline AI Assistant</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .container {
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
            max-width: 800px;
            width: 100%;
            padding: 40px;
        }

        h1 {
            text-align: center;
            color: #667eea;
            margin-bottom: 10px;
            font-size: 2.5em;
        }

        .subtitle {
            text-align: center;
            color: #666;
            margin-bottom: 30px;
            font-size: 0.9em;
        }

        .chat-container {
            height: 400px;
            overflow-y: auto;
            border: 2px solid #e0e0e0;
            border-radius: 10px;
            padding: 20px;
            margin-bottom: 20px;
            background: #f9f9f9;
        }

        .message {
            margin-bottom: 15px;
            padding: 12px 16px;
            border-radius: 10px;
            max-width: 80%;
            animation: fadeIn 0.3s;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .user-message {
            background: #667eea;
            color: white;
            margin-left: auto;
            text-align: right;
        }

        .assistant-message {
            background: #e8eaf6;
            color: #333;
        }

        .controls {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }

        input[type="text"] {
            flex: 1;
            padding: 15px;
            border: 2px solid #e0e0e0;
            border-radius: 10px;
            font-size: 16px;
            transition: border 0.3s;
        }

        input[type="text"]:focus {
            outline: none;
            border-color: #667eea;
        }

        button {
            padding: 15px 30px;
            border: none;
            border-radius: 10px;
            font-size: 16px;
            cursor: pointer;
            transition: all 0.3s;
            font-weight: 600;
        }

        .send-btn {
            background: #667eea;
            color: white;
        }

        .send-btn:hover {
            background: #5568d3;
            transform: translateY(-2px);
        }

        .voice-btn {
            background: #764ba2;
            color: white;
            min-width: 120px;
        }

        .voice-btn:hover {
            background: #653a8a;
            transform: translateY(-2px);
        }

        .voice-btn.listening {
            background: #e74c3c;
            animation: pulse 1.5s infinite;
        }

        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.05); }
        }

        .status {
            text-align: center;
            padding: 10px;
            border-radius: 5px;
            margin-bottom: 10px;
            font-size: 14px;
        }

        .status.info {
            background: #e3f2fd;
            color: #1976d2;
        }

        .status.error {
            background: #ffebee;
            color: #c62828;
        }

        .features {
            background: #f5f5f5;
            padding: 15px;
            border-radius: 10px;
            margin-top: 20px;
        }

        .features h3 {
            color: #667eea;
            margin-bottom: 10px;
        }

        .features ul {
            list-style-position: inside;
            color: #666;
        }

        .features li {
            margin: 5px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🤖 Offline AI Assistant</h1>
        <p class="subtitle">Your personal voice-enabled assistant (no internet required)</p>
        
        <div id="status" class="status info" style="display: none;"></div>
        
        <div class="chat-container" id="chatContainer">
            <div class="message assistant-message">
                Hello! I'm your offline AI assistant. I can help you with calculations, information, reminders, and more. How can I assist you today?
            </div>
        </div>

        <div class="controls">
            <input type="text" id="userInput" placeholder="Type your message or click the microphone to speak..." />
            <button class="voice-btn" id="voiceBtn" onclick="toggleVoice()">🎤 Voice</button>
            <button class="send-btn" onclick="sendMessage()">Send</button>
        </div>
        
        <div id="voiceIndicator" class="voice-indicator" style="display: none;">
            <div class="pulse-ring"></div>
            <div class="listening-text">Listening... Speak now</div>
        </div>

        <div class="features">
            <h3>Available Commands:</h3>
            <ul>
                <li><strong>Calculate:</strong> "What is 25 * 4?" or "Calculate 100 / 5"</li>
                <li><strong>Time:</strong> "What time is it?" or "What's the date?"</li>
                <li><strong>Timer:</strong> "Set timer for 5 minutes"</li>
                <li><strong>Reminder:</strong> "Remind me to call John"</li>
                <li><strong>Information:</strong> Ask general questions and get helpful responses</li>
            </ul>
        </div>
    </div>

    <script>
        let recognition;
        let isListening = false;
        const synth = window.speechSynthesis;

        // Initialize speech recognition
        if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            recognition = new SpeechRecognition();
            recognition.continuous = false;
            recognition.interimResults = false;
            recognition.lang = 'en-US';

            recognition.onresult = (event) => {
                const transcript = event.results[0][0].transcript;
                document.getElementById('userInput').value = transcript;
                sendMessage();
            };

            recognition.onerror = (event) => {
                showStatus('Voice recognition error: ' + event.error, 'error');
                isListening = false;
                updateVoiceButton();
            };

            recognition.onend = () => {
                isListening = false;
                updateVoiceButton();
            };
        }

        function toggleVoice() {
            if (!recognition) {
                showStatus('Speech recognition not supported in this browser', 'error');
                return;
            }

            if (isListening) {
                recognition.stop();
                isListening = false;
            } else {
                recognition.start();
                isListening = true;
                showStatus('Listening... Speak now', 'info');
            }
            updateVoiceButton();
        }

        function updateVoiceButton() {
            const btn = document.getElementById('voiceBtn');
            if (isListening) {
                btn.classList.add('listening');
                btn.textContent = '🔴 Listening...';
            } else {
                btn.classList.remove('listening');
                btn.textContent = '🎤 Speak';
            }
        }

        function sendMessage() {
            const input = document.getElementById('userInput');
            const message = input.value.trim();
            
            if (message === '') return;

            addMessage(message, 'user');
            input.value = '';

            setTimeout(() => {
                const response = processMessage(message);
                addMessage(response, 'assistant');
                speak(response);
            }, 500);
        }

        function addMessage(text, sender) {
            const chatContainer = document.getElementById('chatContainer');
            const messageDiv = document.createElement('div');
            messageDiv.className = `message ${sender}-message`;
            messageDiv.textContent = text;
            chatContainer.appendChild(messageDiv);
            chatContainer.scrollTop = chatContainer.scrollHeight;
        }

        function processMessage(msg) {
            const lower = msg.toLowerCase();

            // Math calculations
            if (lower.includes('calculate') || lower.includes('what is') || lower.match(/[\d+\-*/]/)) {
                return handleCalculation(msg);
            }

            // Time queries
            if (lower.includes('time')) {
                return `The current time is ${new Date().toLocaleTimeString()}`;
            }

            // Date queries
            if (lower.includes('date') || lower.includes('today')) {
                return `Today is ${new Date().toLocaleDateString('en-US', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })}`;
            }

            // Timer
            if (lower.includes('timer') || lower.includes('alarm')) {
                return handleTimer(msg);
            }

            // Reminders
            if (lower.includes('remind')) {
                return `I've noted your reminder: "${msg.replace(/remind me to/i, '').trim()}". However, as an offline assistant, I can't send notifications. Consider setting a timer on your device!`;
            }

            // Greetings
            if (lower.match(/^(hi|hello|hey|greetings)/)) {
                return "Hello! How can I help you today?";
            }

            // How are you
            if (lower.includes('how are you')) {
                return "I'm functioning perfectly! Thanks for asking. How can I assist you?";
            }

            // Help
            if (lower.includes('help') || lower === '?') {
                return "I can help with calculations, tell you the time/date, set timers, and answer general questions. Try asking me something!";
            }

            // Default response
            return generateResponse(msg);
        }

        function handleCalculation(msg) {
            try {
                const expression = msg.replace(/calculate|what is|equals|=/gi, '').trim();
                const result = eval(expression.replace(/[^0-9+\-*/().\s]/g, ''));
                return `The answer is ${result}`;
            } catch (e) {
                return "I couldn't calculate that. Please provide a valid mathematical expression.";
            }
        }

        function handleTimer(msg) {
            const minutes = msg.match(/(\d+)\s*(minute|min)/i);
            const seconds = msg.match(/(\d+)\s*(second|sec)/i);
            
            let totalSeconds = 0;
            if (minutes) totalSeconds += parseInt(minutes[1]) * 60;
            if (seconds) totalSeconds += parseInt(seconds[1]);

            if (totalSeconds >  0) {
                setTimeout(() => {
                    addMessage('⏰ Timer finished!', 'assistant');
                    speak('Timer finished!');
                }, totalSeconds * 1000);
                return `Timer set for ${totalSeconds} seconds. I'll notify you when it's done!`;
            }
            return "Please specify a valid time for the timer (e.g., '5 minutes' or '30 seconds').";
        }

        function generateResponse(msg) {
            const responses = [
                "That's interesting! Tell me more.",
                "I understand. Is there anything specific you'd like help with?",
                "I'm here to assist you with calculations, time, and general questions.",
                "That's a good question! While I'm an offline assistant with limited knowledge, I'll do my best to help.",
                "I appreciate your question. I can help with math, time, timers, and basic queries."
            ];
            return responses[Math.floor(Math.random() * responses.length)];
        }

        function speak(text) {
            if (synth.speaking) {
                synth.cancel();
            }
            const utterance = new SpeechSynthesisUtterance(text);
            utterance.rate = 1;
            utterance.pitch = 1;
            utterance.volume = 1;
            synth.speak(utterance);
        }

        function showStatus(message, type) {
            const statusDiv = document.getElementById('status');
            statusDiv.textContent = message;
            statusDiv.className = `status ${type}`;
            statusDiv.style.display = 'block';
            setTimeout(() => {
                statusDiv.style.display = 'none';
            }, 3000);
        }

        // Allow Enter key to send message
        document.getElementById('userInput').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                sendMessage();
            }
        });
    </script>
</body>
</html>
