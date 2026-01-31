<!DOCTYPE html>
<html lang="mk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>GamerAI Pro Ultimate</title>
    <style>
        :root {
            --primary: #7d5fff;
            --bg: #050508;
            --panel: #11111a;
            --text: #f0f0f0;
        }

        body {
            margin: 0; background: var(--bg); color: var(--text);
            font-family: 'Segoe UI', sans-serif; display: flex; flex-direction: column; height: 100vh;
        }

        header {
            padding: 20px; background: var(--panel); border-bottom: 2px solid var(--primary);
            text-align: center; box-shadow: 0 0 20px rgba(125, 95, 255, 0.4);
        }

        header h1 { margin: 0; font-size: 1.2rem; letter-spacing: 4px; text-transform: uppercase; color: #fff; }

        #chat-window { flex: 1; overflow-y: auto; padding: 20px; display: flex; flex-direction: column; gap: 15px; }

        .msg {
            max-width: 85%; padding: 12px 18px; border-radius: 15px;
            font-size: 0.95rem; line-height: 1.5; animation: slideIn 0.3s ease;
        }

        @keyframes slideIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        .user { align-self: flex-end; background: var(--primary); color: white; border-bottom-right-radius: 2px; }
        
        .ai { 
            align-self: flex-start; background: #1a1a2e; border: 1px solid #333; 
            border-bottom-left-radius: 2px; position: relative;
        }

        .ai::before {
            content: ""; position: absolute; top: 0; left: 0; width: 3px; height: 100%;
            background: var(--primary); box-shadow: 0 0 10px var(--primary);
        }

        pre {
            background: #000; color: #50fa7b; padding: 15px; border-radius: 8px;
            font-size: 0.85rem; overflow-x: auto; border: 1px solid #222; margin-top: 10px;
        }

        .input-box { padding: 15px; background: var(--panel); display: flex; gap: 10px; border-top: 1px solid #222; }
        
        input {
            flex: 1; background: #000; border: 1px solid #333; padding: 12px;
            border-radius: 10px; color: white; outline: none; font-size: 16px;
        }

        button {
            background: var(--primary); border: none; padding: 0 20px;
            border-radius: 10px; color: white; font-weight: bold; cursor: pointer;
        }

        #loader { font-size: 0.7rem; color: var(--primary); text-align: center; display: none; padding: 5px; font-weight: bold; }
    </style>
</head>
<body>

<header><h1>GAMER AI <span style="color:var(--primary)">PRO</span></h1></header>

<div id="chat-window">
    <div class="msg ai">Твојот клуч е внесен! Сега сум целосно активен. Прашај ме нешто за Roblox или CS2!</div>
</div>

<div id="loader">AI СЕ ПОВРЗУВА...</div>

<div class="input-box">
    <input type="text" id="query" placeholder="Прашај нешто..." onkeypress="if(event.key==='Enter') send()">
    <button onclick="send()">SEND</button>
</div>

<script>
    // Твојот клуч е веќе тука и е исчистен од празни места
    const API_TOKEN = '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'.trim(); 

    async function send() {
        const input = document.getElementById('query');
        const win = document.getElementById('chat-window');
        const loader = document.getElementById('loader');
        const val = input.value.trim();

        if (!val) return;

        win.innerHTML += `<div class="msg user">${val}</div>`;
        input.value = '';
        win.scrollTop = win.scrollHeight;
        loader.style.display = 'block';

        try {
            // Користиме посигурен Proxy за да ја заобиколиме CORS грешката на телефон
            const proxy = "https://corsproxy.io/?";
            const target = "https://api-inference.huggingface.co/models/mistralai/Mistral-7B-Instruct-v0.3";

            const res = await fetch(proxy + encodeURIComponent(target), {
                method: "POST",
                headers: {
                    "Authorization": `Bearer ${API_TOKEN}`,
                    "Content-Type": "application/json"
                },
                body: JSON.stringify({
                    inputs: `[INST] You are a professional gaming assistant. Be short and helpful. User asks: ${val} [/INST]`,
                    parameters: { max_new_tokens: 500, wait_for_model: true }
                })
            });

            const data = await res.json();

            // Ако моделот уште се вчитува на серверот
            if (data.error && data.error.includes("currently loading")) {
                win.innerHTML += `<div class="msg ai">Серверот го вчитува моделот... Пробај пак за 10 секунди.</div>`;
            } else {
                let reply = data[0].generated_text.split('[/INST]').pop().trim();
                
                if (reply.includes('```')) {
                    reply = reply.replace(/```([\s\S]*?)```/g, '<pre>$1</pre>');
                }
                win.innerHTML += `<div class="msg ai">${reply}</div>`;
            }

        } catch (e) {
            win.innerHTML += `<div class="msg ai" style="border-color:red">Конекцијата е блокирана. Отвори го фајлот во Chrome Browser!</div>`;
        } finally {
            loader.style.display = 'none';
            win.scrollTop = win.scrollHeight;
        }
    }
</script>

</body>
</html>
