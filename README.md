<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KeenanAI - Asisten Cerdas Anda</title>
    <!-- Memuat Tailwind CSS untuk styling modern dan responsif -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- MEMUAT FONT INTER (FONT YANG MIRIP DENGAN ESTETIKA GEMINI/GOOGLE) -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        /* Mengaplikasikan font Inter ke seluruh body */
        body {
            font-family: 'Inter', sans-serif;
        }

        /* Gaya kustom untuk membatasi tinggi dan memastikan scrolling yang halus */
        .chat-container {
            height: 90vh;
            max-height: 900px;
            width: 100%;
            max-width: 500px;
        }
        .chat-box {
            overflow-y: auto;
            scroll-behavior: smooth;
        }
        /* Mengatur agar textarea menyesuaikan isi (max 5 baris) */
        #user-input {
            max-height: 120px; /* Sekitar 5 baris teks */
            resize: none;
            overflow-y: auto;
        }
        /* Icon bintang 6-sudut untuk status AI */
        .ai-icon {
            content: ' ';
            display: inline-block;
            width: 1rem;
            height: 1rem;
            background-color: #34D399; /* Hijau (Tetap terang untuk kontras status) */
            -webkit-mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath d='M12 2l2.4 6.7h7.2l-5.8 4.3 2.4 6.7-6.2-4.7-6.2 4.7 2.4-6.7-5.8-4.3h7.2z'/%3E%3C/svg>") no-repeat 50% 50%;
            mask: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath d='M12 2l2.4 6.7h7.2l-5.8 4.3 2.4 6.7-6.2-4.7-6.2 4.7 2.4-6.7-5.8-4.3h7.2z'/%3E%3C/svg>") no-repeat 50% 50%;
            -webkit-mask-size: cover;
            mask-size: cover;
        }
    </style>
</head>
<!-- TEMA LEBIH HITAM: Mengubah latar belakang body menjadi abu-abu super gelap (950) -->
<body class="bg-gray-950 flex items-center justify-center min-h-screen p-4 text-white">

    <!-- Container Utama Chatbot: Mengubah latar belakang menjadi abu-abu gelap (900) -->
    <div class="chat-container bg-gray-900 rounded-xl shadow-2xl flex flex-col transition-all duration-300">
        
        <!-- Header Chatbot: Mempertahankan aksen biru tua -->
        <header class="chat-header bg-indigo-700 text-white p-4 rounded-t-xl flex items-center justify-between">
            <div class="flex items-center space-x-2">
                <span class="ai-icon bg-white"></span>
                <h1 class="text-xl font-bold">KeenanAI</h1>
            </div>
            <span class="text-sm opacity-80">Online</span>
        </header>

        <!-- Kotak Pesan (Tempat Menampilkan Riwayat) -->
        <div id="chat-box" class="chat-box flex-grow p-4 space-y-4">
            <!-- Pesan Sambutan Awal dari KeenanAI -->
            <div class="flex justify-start">
                <!-- Pesan AI: Latar belakang abu-abu tua (800), teks putih -->
                <div class="bg-gray-800 text-white p-3 rounded-xl rounded-tl-none max-w-xs md:max-w-md shadow-md">
                    Halo! Saya KeenanAI. Tanyakan apa pun pada saya.
                </div>
            </div>
        </div>

        <!-- Area Input -->
        <div class="input-area p-4 border-t border-gray-800 flex items-end bg-gray-900 rounded-b-xl">
            <textarea id="user-input" 
                      placeholder="Ketik pesan" 
                      rows="1" 
                      class="flex-grow p-3 text-white bg-gray-800 border border-gray-700 rounded-2xl focus:outline-none focus:border-blue-500 transition duration-150"></textarea>
            
            <button id="send-button" 
                    class="ml-3 bg-blue-600 hover:bg-blue-700 text-white p-3 rounded-full shadow-lg transition duration-150 ease-in-out disabled:opacity-50"
                    disabled>
                <!-- Ikon Kirim (Send Icon) dari Lucide -->
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="w-5 h-5">
                    <path d="m22 2-7 20-4-9-9-4 20-7Z"></path>
                    <path d="M22 2 11 13"></path>
                </svg>
            </button>
        </div>
    </div>

    <script>
        // DOM Elements
        const chatBox = document.getElementById('chat-box');
        const userInput = document.getElementById('user-input');
        const sendButton = document.getElementById('send-button');

        // --- Konfigurasi API ---
        const TEXT_MODEL = 'gemini-2.5-flash-preview-09-2025';
        // !!! Kunci API yang Disediakan Pengguna !!!
        const apiKey = "AIzaSyChjewPz_As5SEV0LwTCHUgkg1aSSd1dYY"; 
        
        const textApiUrl = `https://generativelanguage.googleapis.com/v1beta/models/${TEXT_MODEL}:generateContent?key=${apiKey}`;

        // Riwayat obrolan untuk mempertahankan konteks
        let chatHistory = [
            {
                role: "model",
                parts: [{ text: "Halo! Saya KeenanAI. Tanyakan apa pun pada saya." }]
            }
        ];

        // --- Fungsi Utilitas UI ---

        const toggleSendButton = () => {
            sendButton.disabled = userInput.value.trim() === '';
        };

        const adjustTextareaHeight = () => {
            userInput.style.height = 'auto'; 
            userInput.style.height = userInput.scrollHeight + 'px';
        };

        const scrollToBottom = () => {
            chatBox.scrollTop = chatBox.scrollHeight;
        };

        userInput.addEventListener('input', () => {
            adjustTextareaHeight();
            toggleSendButton();
        });

        const addMessage = (text, sender) => {
            const messageWrapper = document.createElement('div');
            messageWrapper.className = sender === 'user' ? 'flex justify-end' : 'flex justify-start';

            const messageContent = document.createElement('div');
            messageContent.className = sender === 'user'
                ? 'bg-blue-600 text-white p-3 rounded-xl rounded-br-none max-w-xs md:max-w-md shadow-lg whitespace-pre-wrap'
                : 'bg-gray-800 text-white p-3 rounded-xl rounded-tl-none max-w-xs md:max-w-md shadow-md whitespace-pre-wrap';
            
            messageContent.textContent = text;
            messageWrapper.appendChild(messageContent);
            chatBox.appendChild(messageWrapper);
            
            scrollToBottom();
        };

        const addTypingIndicator = (loadingText = 'KeenanAI sedang mengetik...') => {
            const indicatorWrapper = document.createElement('div');
            indicatorWrapper.className = 'flex justify-start indicator-wrapper';
            indicatorWrapper.innerHTML = `
                <div class="bg-gray-800 text-gray-400 p-3 rounded-xl rounded-tl-none shadow-md">
                    <span class="animate-pulse">${loadingText}</span>
                </div>
            `;
            chatBox.appendChild(indicatorWrapper);
            scrollToBottom();
            return indicatorWrapper;
        };

        // --- Logika API dengan Exponential Backoff ---

        const generateContentWithBackoff = async (payload, maxRetries = 5, url) => {
            for (let i = 0; i < maxRetries; i++) {
                try {
                    const response = await fetch(url, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (response.ok) {
                        return await response.json();
                    } else if (response.status === 429 || response.status >= 500) {
                        const delay = Math.pow(2, i) * 1000 + Math.random() * 1000;
                        await new Promise(resolve => setTimeout(resolve, delay));
                    } else {
                        // Jika status bukan 429/5xx, anggap sebagai error non-retryable
                        const errorData = await response.json();
                        throw new Error(`API Error ${response.status}: ${errorData.error?.message || 'Unknown error'}`);
                    }
                } catch (error) {
                    if (i === maxRetries - 1) throw error; 
                }
            }
        };

        // Fungsi untuk memanggil Gemini API (Teks)
        const getAiResponse = async () => {
            userInput.disabled = true;
            sendButton.disabled = true;

            const typingIndicator = addTypingIndicator();

            try {
                const payload = {
                    contents: chatHistory,
                    systemInstruction: {
                         parts: [{ text: "Anda adalah KeenanAI, asisten AI yang ramah, sopan, dan profesional. Selalu jawab dalam Bahasa Indonesia." }]
                    },
                    tools: [{ "google_search": {} }]
                };

                const result = await generateContentWithBackoff(payload, 5, textApiUrl);
                
                const candidate = result.candidates?.[0];
                const responseText = candidate?.content?.parts?.[0]?.text;
                
                if (responseText) {
                    chatHistory.push({ role: "model", parts: [{ text: responseText }] });
                    typingIndicator.remove(); 
                    addMessage(responseText, 'ai');
                } else {
                    throw new Error("Respons API kosong atau tidak valid.");
                }

            } catch (error) {
                console.error("Kesalahan dalam panggilan API Teks:", error);
                if (typingIndicator) typingIndicator.remove();
                
                // --- PERBAIKAN: Hapus pesan pengguna terakhir jika terjadi error ---
                addMessage("Maaf, KeenanAI sedang mengalami kesulitan teknis. (" + error.message.substring(0, 100) + "...) Silakan coba lagi. Riwayat pesan terakhir Anda telah dihapus.", 'ai');
                
                if (chatHistory.length > 0 && chatHistory[chatHistory.length - 1].role === 'user') {
                    // Hapus pesan pengguna terakhir yang menyebabkan error dari riwayat
                    chatHistory.pop();
                }
                // --------------------------------------------------------------------

            } finally {
                userInput.disabled = false;
                toggleSendButton();
                userInput.focus();
            }
        };

        // Fungsi utama pengiriman pesan
        const sendMessage = () => {
            const userText = userInput.value.trim();
            if (userText === "") return;

            // Tambahkan pesan pengguna ke UI
            addMessage(userText, 'user');
            
            // Panggilan untuk respons Teks (TIDAK ADA LOGIKA GAMBAR)
            chatHistory.push({ role: "user", parts: [{ text: userText }] });
            getAiResponse(); 

            // Kosongkan input dan reset UI
            userInput.value = ''; 
            adjustTextareaHeight(); 
            toggleSendButton();
        };

        // Event listener untuk tombol Kirim
        sendButton.addEventListener('click', sendMessage);

        // Event listener untuk menekan Enter pada textarea
        userInput.addEventListener('keydown', function(event) {
            if (event.key === 'Enter' && !event.shiftKey) {
                event.preventDefault(); 
                if (!sendButton.disabled) {
                    sendMessage();
                }
            }
        });

        // Inisialisasi awal
        window.onload = () => {
            toggleSendButton();
            userInput.focus();
        };

    </script>

</body>
</html>
