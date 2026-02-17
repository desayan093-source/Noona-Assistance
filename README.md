<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Noona AI</title>

<!-- PWA Settings -->
<meta name="theme-color" content="#6c63ff">
<link rel="manifest" href="data:application/json,{
  &quot;name&quot;:&quot;Noona AI&quot;,
  &quot;short_name&quot;:&quot;Noona&quot;,
  &quot;start_url&quot;:&quot;.&quot;,
  &quot;display&quot;:&quot;standalone&quot;,
  &quot;background_color&quot;:&quot;#6c63ff&quot;,
  &quot;theme_color&quot;:&quot;#6c63ff&quot;
}">

<style>
body{
  margin:0;
  font-family:Arial, sans-serif;
  background:linear-gradient(135deg,#6c63ff,#4834d4);
  display:flex;
  flex-direction:column;
  height:100vh;
  color:white;
}
header{
  text-align:center;
  padding:15px;
  font-size:20px;
  font-weight:bold;
}
#chat{
  flex:1;
  padding:10px;
  overflow-y:auto;
}
.message{
  margin:10px 0;
  padding:10px;
  border-radius:12px;
  max-width:80%;
}
.user{
  background:#ffffff;
  color:black;
  align-self:flex-end;
}
.bot{
  background:#2f2fa2;
}
#inputArea{
  display:flex;
  padding:10px;
  background:rgba(0,0,0,0.2);
}
input{
  flex:1;
  padding:10px;
  border-radius:8px;
  border:none;
  outline:none;
}
button{
  margin-left:5px;
  padding:10px 15px;
  border:none;
  border-radius:8px;
  background:white;
  cursor:pointer;
}
</style>
</head>
<body>

<header>Noona AI</header>

<div id="chat"></div>

<div id="inputArea">
  <input type="text" id="userInput" placeholder="Type something...">
  <button onclick="sendMessage()">Send</button>
  <button onclick="startVoice()">🎤</button>
</div>

<script>
const chat = document.getElementById("chat");
const input = document.getElementById("userInput");

// 🔑 Put your Gemini API key here
const API_KEY = "YOUR_GEMINI_API_KEY";

function addMessage(text, sender){
  const msg = document.createElement("div");
  msg.classList.add("message", sender);
  msg.innerText = text;
  chat.appendChild(msg);
  chat.scrollTop = chat.scrollHeight;
}

async function sendMessage(){
  const text = input.value.trim();
  if(!text) return;

  addMessage(text, "user");
  input.value = "";

  addMessage("Thinking...", "bot");

  const response = await fetch(
    "https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=" + API_KEY,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        contents: [{ parts: [{ text: text }] }]
      })
    }
  );

  const data = await response.json();
  const reply = data.candidates?.[0]?.content?.parts?.[0]?.text || "Error";

  chat.lastChild.remove();
  addMessage(reply, "bot");

  speak(reply);
}

function speak(text){
  const utterance = new SpeechSynthesisUtterance(text);
  speechSynthesis.speak(utterance);
}

function startVoice(){
  const recognition = new(window.SpeechRecognition || window.webkitSpeechRecognition)();
  recognition.lang = "en-US";
  recognition.start();
  recognition.onresult = function(event){
    input.value = event.results[0][0].transcript;
    sendMessage();
  };
}
</script>

</body>
</html>
