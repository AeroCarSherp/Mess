mychat/
├── server.py          
├── index.html         
└── requirements.txt   
fastapi==0.111.0
uvicorn[standard]==0.30.1
jinja2==3.1.4
fastapi==0.111.0
uvicorn[standard]==0.30.1
jinja2==3.1.4
fastapi==0.111.0
uvicorn[standard]==0.30.1
jinja2==3.1.4
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.responses import HTMLResponse
from pathlib import Path

app = FastAPI(title="MiniChat")

TEMPLATE = Path(__file__).with_name("index.html").read_text(encoding="utf‑8")

class Hub:
    
    def __init__(self) -> None:
        self.clients: list[WebSocket] = []

    async def connect(self, ws: WebSocket) -> None:
        await ws.accept()
        self.clients.append(ws)

    def disconnect(self, ws: WebSocket) -> None:
        if ws in self.clients:
            self.clients.remove(ws)

    async def broadcast(self, message: str) -> None:
        for client in self.clients.copy():
            try:
                await client.send_text(message)
            except Exception:
                
                self.disconnect(client)

hub = Hub()

@app.get("/", response_class=HTMLResponse)
async def root() -> str:
    
    return TEMPLATE

@app.websocket("/ws/{username}")
async def chat_ws(ws: WebSocket, username: str) -> None:
    
    await hub.connect(ws)
    await hub.broadcast(f"🟢 <b>{username}</b> joined the chat")

    try:
        while True:
            data = await ws.receive_text()
            await hub.broadcast(f"<b>{username}:</b> {data}")
    except WebSocketDisconnect:
        hub.disconnect(ws)
        await hub.broadcast(f"🔴 <b>{username}</b> left the chat")
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>MiniChat</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    body{font-family:system-ui, sans-serif;display:flex;flex-direction:column;
         gap:1rem;max-width:600px;margin:2rem auto;padding:0 1rem}
    #log{height:300px;resize:none;width:100%;border:1px solid #ccc;
         padding:.5rem;border-radius:.5rem;overflow-y:auto}
    #controls{display:flex;gap:.5rem}
    input,button{padding:.5rem;font-size:1rem}
    button{cursor:pointer}
  </style>
</head>
<body>
  <h1>MiniChat</h1>

  <div id="join-block">
    <input id="username" placeholder="Enter a nickname" />
    <button id="join">Join</button>
  </div>

  <textarea id="log" readonly></textarea>

  <div id="controls" hidden>
    <input id="msg" placeholder="Type a message…" autocomplete="off" />
    <button id="send">Send</button>
  </div>

  <script>
    let ws;

   
    function appendLog(html){
      const log=document.getElementById("log");
      const atBottom = log.scrollTop + log.clientHeight >= log.scrollHeight - 5;
      log.insertAdjacentHTML("beforeend", html+"<br>");
      if(atBottom) log.scrollTop = log.scrollHeight;
    }

    document.getElementById("join").onclick = () => {
      const name = document.getElementById("username").value.trim();
      if(!name) return alert("Choose a nickname");
      ws = new WebSocket(`ws://${location.host}/ws/${encodeURIComponent(name)}`);

      ws.onopen = () => {
        document.getElementById("join-block").hidden = true;
        document.getElementById("controls").hidden     = false;
        document.getElementById("msg").focus();
      };
      ws.onmessage = (e) => appendLog(e.data);
      ws.onclose   = () => appendLog("<i>Connection closed.</i>");
    };

    document.getElementById("send").onclick = () => {
      const input = document.getElementById("msg");
      if(input.value && ws?.readyState===1){
        ws.send(input.value);
        input.value="";
      }
    };

    
    document.getElementById("msg").addEventListener("keydown", e=>{
      if(e.key==="Enter" && (e.ctrlKey||e.metaKey)){
        e.preventDefault(); document.getElementById("send").click();
      }
    });
  </script>
</body>
</html>

git clone https://github.com/your‑repo/mychat.git
cd mychat


python -m venv .venv && source .venv/bin/activate  # Windows: .venv\Scripts\activate


pip install -r requirements.txt


python -m uvicorn server:app --reload
