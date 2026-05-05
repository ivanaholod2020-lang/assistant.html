<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>🧠 Автономный ИИ Ассистент</title>
    <style>
        body {
            margin: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background: linear-gradient(135deg, #1A2A4C, #263F5F); /* Темный градиент */
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .chat-container {
            width: 100%;
            max-width: 700px;
            height: 90vh;
            background: #0D1B2A; /* Очень темный фон */
            border-radius: 20px;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5);
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        #chat-history {
            flex-grow: 1;
            padding: 25px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .message {
            max-width: 80%;
            padding: 14px 20px;
            border-radius: 22px;
            line-height: 1.6;
            position: relative;
        }

        .user {
             align-self: flex-end;
             background-color: #4CAF50; /* Зеленый для пользователя */
             color: white;
             border-bottom-right-radius: 6px; 
        }

        .assistant {
             align-self: flex-start;
             background-color: #2196F3; /* Синий для ИИ */
             color: white;
             border-bottom-left-radius: 6px; 
        }

        /* Анимация печати */
        .typing::before {
             content: '';
             display: inline-block;
             width: 0%;
             height: 1.2em; 
             background-color: rgba(255,255,255,0.3);
             animation: typing 2s steps(40, end) infinite, blink-caret .75s step-end infinite;
        }
        @keyframes typing { from { width: 0%; } to { width: 100%; } }
        @keyframes blink-caret { from, to { border-color: transparent; } 50% { border-color: rgba(255,255,255,0.3); } }

        .input-area {
             padding: 15px;
             border-top: 1px solid #1A2A4C;
             display: flex;
             gap: 10px;
             align-items: center;
             background-color: #0D1B2A;
        }

        #user-input {
             flex-grow: 1;
             padding: 14px 20px;
             border-radius: 25px;
             border: none;
             background-color: #1E232C;
             color: white;
             font-size: 16px;
             outline: none;
        }
        #user-input::placeholder { color: #666; }

        button {
             padding: 14px 30px;
             border-radius: 25px;
             border: none;
             font-size: 16px;
             font-weight: bold;
             cursor: pointer;
             background-color: #FFC107; /* Желтая кнопка */
             color: black;
             transition: background-color 0.2s;
        }
        button:hover { background-color: #FFA000; }
    </style>
</head>
<body>

<div class="chat-container">
    <div id="chat-history">
         <div class="message assistant">Привет! Я автономный ИИ. Я думаю прямо здесь. Это может занять несколько секунд при первом запуске...</div>
    </div>

    <div class="input-area">
         <input type="text" id="user-input" placeholder="Введите ваш вопрос...">
         <button onclick="sendMessage()">Спросить</button>
    </div>
</div>

<!-- Подключаем библиотеку ИИ -->
<script src="https://cdn.jsdelivr.net/npm/@xenova/transformers@2.6.0"></script>
<script>
let model = null; // Модель будет загружена позже
let isGenerating = false; // Флаг, чтобы не отправлять новый запрос, пока думает

const chatHistory = document.getElementById('chat-history');
const userInput = document.getElementById('user-input');

// Функция для добавления сообщений в чат
function addMessage(text, sender) {
    const div = document.createElement('div');
    div.className = 'message ' + sender;
    div.textContent = text; // Текст нужен для определения ширины

    if (sender === 'assistant') {
         div.classList.add('typing'); // Добавляем класс анимации
         // Убираем анимацию через время (когда "напечатается")
         // Длительность зависит от длины текста
         setTimeout(() => div.classList.remove('typing'), text.length * 45 + 800); 
    }
    
    chatHistory.appendChild(div);
    chatHistory.scrollTop = chatHistory.scrollHeight; // Прокрутка вниз
}

// Функция отправки сообщения
async function sendMessage() {
    if (isGenerating || !model) return; // Не работаем, пока не загружена модель или идет генерация

    const query = userInput.value.trim();
    if (!query) return; // Не отправляем пустые сообщения

    addMessage(query, 'user');
    userInput.value = '';
    isGenerating = true;

    try {
        // Генерируем ответ с помощью локальной модели
        const response = await model.generate(query, {
            maxNewTokens: 150,
            doSample: true,
            temperature: 0.7,
        });
        
        const answer = response.output.trim();
        
        if (answer) {
            addMessage(answer, 'assistant');
        } else {
            addMessage("Хм... Что-то пошло не так с ответом.", 'assistant');
        }
        
    } catch (error) {
         console.error(error);
         addMessage("Ой, произошла ошибка. Попробуйте еще раз.", 'assistant');
    } finally {
         isGenerating = false; // Разрешаем новые запросы
    }
}

userInput.addEventListener('keydown', e => {
     if (e.key === 'Enter') sendMessage();
});


// --- ЗАГРУЗКА МОДЕЛИ ПРИ ОТКРЫТИИ СТРАНИЦЫ ---
async function loadModel() {
     try {
          // Загружаем маленькую и быструю модель (Gemma или MBLab)
          // MBLab (mblab/tiny-llama-fp16) очень быстрый для первого запуска.
          model = await window.transformers.pipeline('text-generation', 'mblab/tiny-llama-fp16', { quantized: true });
          addMessage("✅ Готов к работе! Мозг загружен.", 'assistant');
     } catch (e) {
          console.error("Ошибка загрузки модели:", e);
          addMessage("❌ Не удалось загрузить искусственный интеллект.", 'assistant');
     }
}

// Запускаем загрузку сразу
loadModel();
</script>
</body>
</html>
