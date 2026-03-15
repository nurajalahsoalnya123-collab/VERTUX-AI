<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>VERTUX AI v5.0 (SEPULUH ROBLOX EDITION)</title>
    <style>
        :root { --grad: linear-gradient(135deg, #ff00cc, #3333ff, #00ffcc); }
        body { background: #030307; color: white; font-family: sans-serif; margin: 0; display: flex; flex-direction: column; height: 100vh; }
        .header { text-align: center; padding: 15px; border-bottom: 2px solid #3333ff; }
        .header h1 { font-size: 1.5rem; margin: 0; background: var(--grad); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        #chat-box { flex: 1; overflow-y: auto; padding: 20px; display: flex; flex-direction: column; gap: 10px; }
        .msg { padding: 12px; border-radius: 15px; max-width: 80%; font-size: 14px; }
        .user { align-self: flex-end; background: var(--grad); }
        .ai { align-self: flex-start; background: #1e293b; }
        .input-area { padding: 20px; background: #080810; display: flex; gap: 10px; }
        input { flex: 1; padding: 12px; border-radius: 25px; border: 1px solid #3333ff; background: #151525; color: white; outline: none; }
        button { background: var(--grad); border: none; color: white; padding: 10px 20px; border-radius: 25px; font-weight: bold; }
    </style>
</head>
<body>
    <div class="header"><h1>VERTUX AI</h1></div>
    <div id="chat-box"><div class="msg ai">Ayo jagoan, jangan mau kalah sama API! Cek lagi! 🗿🔥</div></div>
    <div class="input-area">
        <input type="text" id="inp" placeholder="Tanya sesuatu...">
        <button onclick="ask()">KIRIM</button>
    </div>

    <script>
        // 🔑 GANTI API KEY LO DI SINI!
        const MY_KEY = "AIzaSyAjK0rvkyOmlgUSVL5MDYZAmmlsJu2JdKU";

        async function ask() {
            const inp = document.getElementById('inp');
            const box = document.getElementById('chat-box');
            const txt = inp.value.trim();
            if (!txt) return;

            // Validasi API key (jangan pake yang default)
            if (!MY_KEY || MY_KEY.includes("AIzaSyBB1e")) {
                addMsg('API Key masih default atau kosong. Ganti dulu dengan key asli lo!', 'user'); // biar keliatan
                return;
            }

            addMsg(txt, 'user');
            inp.value = '';

            const loading = addMsg('Mikir...', 'ai');

            // Daftar model yang mau dicoba (prioritas dari curl lo)
            const models = [
                "gemini-1.5-flash-latest",
                "gemini-1.5-flash",
                "gemini-flash-latest",
                "gemini-1.0-pro"
            ];

            let found = false;
            let lastError = '';

            for (let m of models) {
                try {
                    // ENDPOINT: pake v1beta, kirim API key via header X-goog-api-key
                    const url = `https://generativelanguage.googleapis.com/v1beta/models/${m}:generateContent`;

                    const res = await fetch(url, {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'X-goog-api-key': MY_KEY   // <-- header sesuai curl
                        },
                        body: JSON.stringify({
                            contents: [{
                                parts: [{ text: txt }]
                            }]
                        })
                    });

                    const data = await res.json();

                    // Cek apakah respons sukses
                    if (res.ok && data.candidates && data.candidates.length > 0) {
                        const reply = data.candidates[0].content.parts[0].text;
                        loading.remove();
                        addMsg(reply, 'ai');
                        found = true;
                        break;
                    } else {
                        // Simpan pesan error untuk ditampilkan nanti
                        lastError = data.error?.message || `HTTP error ${res.status}`;
                    }
                } catch (e) {
                    lastError = e.message;
                }
            }

            if (!found) {
                loading.innerText = `Gagal: ${lastError || 'Semua model tidak merespons.'}`;
                // Kasih saran kalo error karena key
                if (lastError.includes('API key') || lastError.includes('403') || lastError.includes('401')) {
                    addMsg('API Key bermasalah atau billing belum diaktifkan. Cek lagi di Google Cloud Console!', 'ai');
                }
            }
        }

        // Fungsi nambah pesan ke chat box
        function addMsg(text, sender) {
            const msgDiv = document.createElement('div');
            msgDiv.className = `msg ${sender}`;
            msgDiv.innerText = text;
            document.getElementById('chat-box').appendChild(msgDiv);
            document.getElementById('chat-box').scrollTop = document.getElementById('chat-box').scrollHeight;
            return msgDiv;
        }
    </script>
</body>
</html>
