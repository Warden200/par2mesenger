<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Par2Messenger</title>
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-database-compat.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
        }

        body {
            background: linear-gradient(135deg, #0a0c12 0%, #121624 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 16px;
        }

        .chat {
            width: 100%;
            max-width: 900px;
            height: 90vh;
            background: #0f1119e6;
            backdrop-filter: blur(10px);
            border-radius: 2rem;
            box-shadow: 0 25px 45px rgba(0,0,0,0.5), 0 0 0 1px rgba(255,255,255,0.05);
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        .chat-header {
            background: #080b10;
            padding: 1rem 1.5rem;
            border-bottom: 1px solid #242a3a;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 10px;
        }

        .logo {
            display: flex;
            align-items: center;
            gap: 8px;
            font-weight: 600;
            color: #ffbc6e;
        }

        .user-id {
            background: #1a1e2c;
            padding: 6px 14px;
            border-radius: 40px;
            font-size: 0.7rem;
            font-family: monospace;
            color: #b3c7ff;
        }

        .user-id span {
            color: #ffbc6e;
            font-weight: bold;
            background: #2a1f2c;
            padding: 2px 8px;
            border-radius: 30px;
            margin-left: 6px;
        }

        .online-status {
            background: #00ff8844;
            padding: 5px 12px;
            border-radius: 20px;
            font-size: 0.7rem;
            color: #7effb3;
        }

        .messages {
            flex: 1;
            overflow-y: auto;
            padding: 1.2rem;
            display: flex;
            flex-direction: column;
            gap: 0.8rem;
            background: #0b0e16;
        }

        .message {
            display: flex;
            flex-direction: column;
            max-width: 80%;
            animation: fadeIn 0.2s ease;
        }

        .message.own {
            align-self: flex-end;
        }

        .message.other {
            align-self: flex-start;
        }

        .bubble {
            padding: 0.7rem 1.1rem;
            border-radius: 1.2rem;
            font-size: 0.95rem;
            line-height: 1.4;
            word-break: break-word;
        }

        .own .bubble {
            background: linear-gradient(135deg, #2c6285, #1f4a6e);
            color: white;
            border-bottom-right-radius: 0.3rem;
        }

        .other .bubble {
            background: #1f263b;
            color: #eef2ff;
            border: 1px solid #2f354b;
            border-bottom-left-radius: 0.3rem;
        }

        .message-meta {
            font-size: 0.6rem;
            margin-top: 0.25rem;
            padding: 0 0.5rem;
            color: #6f78a3;
            display: flex;
            gap: 0.8rem;
        }

        .input-area {
            background: #080b10;
            padding: 1rem 1.2rem 1.3rem;
            border-top: 1px solid #232838;
        }

        .input-wrapper {
            display: flex;
            gap: 0.7rem;
            background: #121624;
            border-radius: 60px;
            padding: 0.2rem 0.2rem 0.2rem 1.2rem;
            align-items: center;
            border: 1px solid #2f354a;
        }

        .input-wrapper input {
            flex: 1;
            background: transparent;
            border: none;
            padding: 0.8rem 0;
            color: #eef3ff;
            font-size: 0.95rem;
            outline: none;
        }

        .input-wrapper input::placeholder {
            color: #5d6588;
        }

        .input-wrapper button {
            background: #3f618b;
            border: none;
            padding: 0.55rem 1.3rem;
            border-radius: 40px;
            color: white;
            font-weight: bold;
            cursor: pointer;
            transition: 0.1s;
        }

        .input-wrapper button:hover {
            background: #5a7da8;
            transform: scale(0.97);
        }

        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(8px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .messages::-webkit-scrollbar {
            width: 5px;
        }
        .messages::-webkit-scrollbar-track {
            background: #0c0f18;
        }
        .messages::-webkit-scrollbar-thumb {
            background: #3c4465;
            border-radius: 10px;
        }

        .status-message {
            text-align: center;
            color: #5f77aa;
            font-size: 0.8rem;
            padding: 1rem;
        }
        
        /* Кнопка разрешить уведомления */
        .notify-btn {
            background: #3f618b;
            border: none;
            padding: 4px 10px;
            border-radius: 20px;
            color: white;
            font-size: 0.65rem;
            cursor: pointer;
            margin-left: 8px;
        }
        
        .notify-btn:hover {
            background: #5a7da8;
        }
    </style>
</head>
<body>

<div class="chat">
    <div class="chat-header">
        <div class="logo">
            🔥 Par2Messenger
        </div>
        <div class="user-id">
            🕶️ Ваш ID: <span id="userIdSpan">загрузка...</span>
            <button id="enableNotificationsBtn" class="notify-btn" style="display:none;">🔔 Вкл. уведомления</button>
        </div>
        <div class="online-status" id="statusSpan">
            🔄 подключение...
        </div>
    </div>

    <div class="messages" id="messagesContainer">
        <div class="status-message">💬 Загрузка сообщений...</div>
    </div>

    <div class="input-area">
        <div class="input-wrapper">
            <input type="text" id="messageInput" placeholder="✍️ Анонимное сообщение... увидят ВСЕ" autocomplete="off">
            <button id="sendBtn">➤ ОТПРАВИТЬ</button>
        </div>
        <div style="font-size: 0.65rem; text-align: center; margin-top: 8px; color: #4b5376;">
            💡 При новом сообщении приходит уведомление (если чат не в фокусе)
        </div>
    </div>
</div>

<script>
    // ========== ВАШИ ДАННЫЕ ИЗ FIREBASE ==========
    const firebaseConfig = {
        apiKey: "AIzaSyAZQXfQFN7kQynQ4RfAz1h64giZiH78hpg",
        authDomain: "par2masenger.firebaseapp.com",
        projectId: "par2masenger",
        storageBucket: "par2masenger.firebasestorage.app",
        messagingSenderId: "84650108216",
        appId: "1:84650108216:web:587059b247c92ea01ccdcd"
    };
    // =============================================

    // Инициализация Firebase
    firebase.initializeApp(firebaseConfig);
    const database = firebase.database();
    const messagesRef = database.ref('messages');

    // --- Генерация анонимного ID (сохраняется в браузере навсегда)
    let currentUserId = localStorage.getItem('par2messenger_userId');
    if (!currentUserId) {
        const randomPart = Math.random().toString(36).substring(2, 10) + Date.now().toString(36).slice(-4);
        currentUserId = 'anon_' + randomPart;
        localStorage.setItem('par2messenger_userId', currentUserId);
    }
    document.getElementById('userIdSpan').innerHTML = currentUserId + ' <button id="enableNotificationsBtn" class="notify-btn">🔔 Уведомления</button>';
    
    // --- НАСТРОЙКА УВЕДОМЛЕНИЙ ---
    let notificationsEnabled = false;
    let lastMessageCount = 0;
    let isPageActive = true;
    
    // Отслеживаем, активна ли страница
    document.addEventListener('visibilitychange', () => {
        isPageActive = !document.hidden;
        if (isPageActive) {
            // Когда возвращаемся на страницу, сбрасываем счётчик
            lastMessageCount = 0;
        }
    });
    
    // Запрос разрешения на уведомления
    function requestNotificationPermission() {
        if (!('Notification' in window)) {
            console.log('Браузер не поддерживает уведомления');
            alert('Ваш браузер не поддерживает уведомления');
            return;
        }
        
        if (Notification.permission === 'granted') {
            notificationsEnabled = true;
            alert('✅ Уведомления включены!');
        } else if (Notification.permission !== 'denied') {
            Notification.requestPermission().then(permission => {
                if (permission === 'granted') {
                    notificationsEnabled = true;
                    alert('✅ Уведомления включены! Теперь вы будете получать оповещения о новых сообщениях.');
                } else {
                    alert('❌ Уведомления отклонены. Вы можете включить их в настройках браузера.');
                }
            });
        } else {
            alert('❌ Уведомления заблокированы. Включите их в настройках браузера (🔒 рядом с адресной строкой)');
        }
    }
    
    // Показать уведомление
    function showNotification(senderId, messageText, senderShortId) {
        if (!notificationsEnabled && Notification.permission === 'granted') {
            notificationsEnabled = true;
        }
        
        if (notificationsEnabled && !isPageActive) {
            // Показываем уведомление только если страница НЕ активна
            new Notification(`📨 Новое сообщение от ${senderShortId || senderId.slice(-8)}`, {
                body: messageText.length > 100 ? messageText.slice(0, 100) + '...' : messageText,
                icon: 'data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="%232c6285"%3E%3Cpath d="M20 2H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h14l4 4V4c0-1.1-.9-2-2-2z"/%3E%3C/svg%3E',
                silent: false
            });
        }
    }
    
    // --- Статус подключения ---
    const statusSpan = document.getElementById('statusSpan');
    const connectedRef = database.ref('.info/connected');
    connectedRef.on('value', (snap) => {
        if (snap.val() === true) {
            statusSpan.innerHTML = '🟢 онлайн | реальное время';
            statusSpan.style.color = '#7effb3';
        } else {
            statusSpan.innerHTML = '🔴 офлайн';
            statusSpan.style.color = '#ff8888';
        }
    });
    
    // --- ОТПРАВКА СООБЩЕНИЯ ---
    function sendMessage(text) {
        if (!text || text.trim() === '') return false;
        
        const newMessage = {
            userId: currentUserId,
            text: text.trim(),
            timestamp: firebase.database.ServerValue.TIMESTAMP
        };
        
        messagesRef.push(newMessage).catch((error) => {
            console.error('Ошибка:', error);
            alert('Ошибка соединения. Проверьте интернет.');
        });
        return true;
    }
    
    // --- ПОЛУЧЕНИЕ СООБЩЕНИЙ В РЕАЛЬНОМ ВРЕМЕНИ С УВЕДОМЛЕНИЯМИ ---
    const container = document.getElementById('messagesContainer');
    let isFirstMessage = true;
    let lastMessageId = null;
    
    messagesRef.orderByChild('timestamp').on('child_added', (snapshot) => {
        const msg = snapshot.val();
        const msgId = snapshot.key;
        if (!msg) return;
        
        // Убираем заглушку "Загрузка..." при первом сообщении
        if (isFirstMessage && container.children.length === 1) {
            container.innerHTML = '';
            isFirstMessage = false;
        }
        
        const isOwn = (msg.userId === currentUserId);
        
        // ========== УВЕДОМЛЕНИЕ ДЛЯ ЧУЖИХ СООБЩЕНИЙ ==========
        if (!isOwn && msgId !== lastMessageId) {
            lastMessageId = msgId;
            const shortId = msg.userId.length > 12 ? msg.userId.substring(0, 10) + '..' : msg.userId;
            showNotification(msg.userId, msg.text, shortId);
        }
        
        const messageDiv = document.createElement('div');
        messageDiv.className = `message ${isOwn ? 'own' : 'other'}`;
        
        // Форматируем время
        let timeStr = 'только что';
        if (msg.timestamp && typeof msg.timestamp === 'number') {
            const date = new Date(msg.timestamp);
            timeStr = date.toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
        }
        
        // Короткий ID для отображения
        const shortId = msg.userId.length > 12 ? msg.userId.substring(0, 10) + '..' : msg.userId;
        
        messageDiv.innerHTML = `
            <div class="bubble">
                ${escapeHtml(msg.text)}
            </div>
            <div class="message-meta">
                <span>🔹 ${escapeHtml(shortId)}</span>
                <span>🕘 ${timeStr}</span>
            </div>
        `;
        
        container.appendChild(messageDiv);
        container.scrollTop = container.scrollHeight;
    });
    
    // Защита от XSS
    function escapeHtml(str) {
        if (!str) return '';
        return str
            .replace(/&/g, '&amp;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/"/g, '&quot;')
            .replace(/'/g, '&#39;');
    }
    
    // --- Обработчики кнопок ---
    const sendBtn = document.getElementById('sendBtn');
    const inputEl = document.getElementById('messageInput');
    
    sendBtn.addEventListener('click', () => {
        if (sendMessage(inputEl.value)) {
            inputEl.value = '';
            inputEl.focus();
        } else {
            inputEl.placeholder = '❌ Нельзя отправить пустое сообщение';
            setTimeout(() => {
                inputEl.placeholder = '✍️ Анонимное сообщение... увидят ВСЕ';
            }, 1500);
        }
    });
    
    inputEl.addEventListener('keypress', (e) => {
        if (e.key === 'Enter') {
            e.preventDefault();
            sendBtn.click();
        }
    });
    
    // Кнопка включения уведомлений (динамическая)
    setTimeout(() => {
        const notifyBtn = document.getElementById('enableNotificationsBtn');
        if (notifyBtn) {
            notifyBtn.addEventListener('click', requestNotificationPermission);
            
            // Если уже разрешено, показываем другой текст
            if (Notification.permission === 'granted') {
                notifyBtn.textContent = '✅ Уведомления вкл';
                notificationsEnabled = true;
            } else {
                notifyBtn.style.display = 'inline-block';
            }
        }
    }, 100);
    
    inputEl.focus();
    
    // Проверяем разрешение при загрузке
    if (Notification.permission === 'granted') {
        notificationsEnabled = true;
    }
    
    console.log('✅ Par2Messenger запущен! Уведомления добавлены.');
</script>
</body>
</html>
