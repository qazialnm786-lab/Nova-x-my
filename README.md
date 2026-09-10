<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>NOVA-X AI</title>

<style>
*{box-sizing:border-box}

body{
  margin:0;
  font-family:Arial,sans-serif;
  background:#070b16;
  color:white;
}

header{
  padding:18px;
  text-align:center;
  font-size:25px;
  font-weight:bold;
  background:#11182b;
  border-bottom:1px solid #263452;
}

#chat{
  height:calc(100vh - 145px);
  overflow-y:auto;
  padding:15px;
}

.msg{
  max-width:88%;
  margin:10px 0;
  padding:12px 14px;
  border-radius:15px;
  line-height:1.5;
  word-wrap:break-word;
}

.user{
  margin-left:auto;
  background:#245cff;
}

.ai{
  margin-right:auto;
  background:#182238;
}

.buttons{
  margin-top:9px;
  display:flex;
  gap:7px;
  flex-wrap:wrap;
}

button{
  border:0;
  border-radius:10px;
  padding:9px 12px;
  background:#293653;
  color:white;
  cursor:pointer;
}

button:active{
  transform:scale(.96);
}

.listen{
  background:#146c43;
}

.stop{
  background:#8b2635;
}

.copy{
  background:#6246a5;
}

#bottom{
  position:fixed;
  bottom:0;
  left:0;
  right:0;
  padding:10px;
  background:#0e1526;
  display:flex;
  gap:7px;
}

#input{
  flex:1;
  border:0;
  outline:0;
  border-radius:12px;
  padding:12px;
  font-size:16px;
}

.send{
  background:#245cff;
  font-size:18px;
}

.topButtons{
  display:flex;
  justify-content:center;
  gap:8px;
  padding:8px;
  background:#0e1526;
}
</style>
</head>

<body>

<header>🤖 NOVA-X AI</header>

<div class="topButtons">
  <button onclick="newChat()">🆕 New Chat</button>
  <button onclick="clearChat()">🗑️ Clear</button>
  <button onclick="testVoice()">🔊 Test Voice</button>
</div>

<div id="chat"></div>

<div id="bottom">
  <input id="input" placeholder="NOVA-X से कुछ पूछो..." autocomplete="off">
  <button class="send" onclick="sendMessage()">➤</button>
</div>

<script>

/* =========================
   NOVA-X MEMORY
========================= */

let chatBox = document.getElementById("chat");

let history = JSON.parse(
  localStorage.getItem("nova_history") || "[]"
);


/* =========================
   DIFFERENT REPLIES
========================= */

const replies = {

hello: [
  "नमस्ते! 👋 मैं NOVA-X हूँ। बताओ, आज क्या करना है?",
  "हेलो! 😎 NOVA-X तैयार है। अपना सवाल पूछो।",
  "अरे वाह! 👋 मैं यहाँ हूँ। बताओ क्या मदद चाहिए?",
  "नमस्ते दोस्त! 🤖 आज किस चीज़ पर काम करें?",
  "Hello! 🚀 NOVA-X online है। क्या पूछना है?"
],

how: [
  "मैं बिल्कुल बढ़िया हूँ 😎 तुम बताओ कैसे हो?",
  "मैं तैयार हूँ! 🚀 जो पूछना है पूछो।",
  "NOVA-X पूरी तरह तैयार है 🤖। बोलो क्या करना है?",
  "सब बढ़िया! 😄 अब तुम्हारा सवाल बताओ।"
],

name: [
  "मेरा नाम NOVA-X है 🤖।",
  "मैं NOVA-X AI हूँ 🚀।",
  "तुम मुझे NOVA-X कह सकते हो। 😎",
  "NOVA-X — तुम्हारा AI assistant!"
],

thanks: [
  "कोई बात नहीं! 😄",
  "Welcome! 🤖",
  "खुशी हुई मदद करके! 🚀",
  "Anytime! 😎"
],

bye: [
  "ठीक है! 👋 फिर मिलते हैं।",
  "Bye! 🚀 जब जरूरत हो वापस आना।",
  "अलविदा दोस्त! 😄",
  "फिर मिलेंगे! 🤖"
],

help: [
  "मैं सवालों के जवाब देने, ideas देने, coding में मदद करने और बहुत सारी चीज़ों में सहायता कर सकता हूँ। 🤖",
  "बोलो क्या चाहिए—coding, ideas, explanation या सामान्य सवाल। 🚀",
  "अपना सवाल सीधे लिखो। NOVA-X उसे समझने की कोशिश करेगा। 😎"
],

default: [
  "अच्छा सवाल है! 🤔 इसके बारे में मैं तुम्हें समझाकर बता सकता हूँ।",
  "समझ गया 👍 चलो इसे आसान तरीके से देखते हैं।",
  "ठीक है! 🚀 मैं तुम्हारे सवाल के हिसाब से जवाब देता हूँ।",
  "Interesting! 😎 इस पर थोड़ा detail में बात करते हैं।",
  "हाँ, समझ गया। 🤖 तुम्हें इसका आसान जवाब चाहिए।",
  "बिल्कुल! 👍 चलो step-by-step देखते हैं।",
  "समझ गया दोस्त! 🚀 अब इसे आसान भाषा में समझते हैं।",
  "यह अच्छा topic है। 😄 इसके कई तरीके हो सकते हैं।",
  "ठीक है! मैं इसे simple तरीके से explain करता हूँ।",
  "Got it! 🤖 चलो शुरू करते हैं।"
]};


/* =========================
   RANDOM REPLY
   SAME REPLY REPEAT नहीं
========================= */

let lastReply = "";

function randomReply(list){

  if(list.length === 1) return list[0];

  let result;

  do{
    result = list[Math.floor(Math.random()*list.length)];
  }while(result === lastReply);

  lastReply = result;

  return result;
}


/* =========================
   GET REPLY
========================= */

function getReply(text){

  let t = text.toLowerCase().trim();

  if(
    t.includes("hello") ||
    t.includes("hi") ||
    t.includes("हेलो") ||
    t.includes("नमस्ते") ||
    t === "hey"
  ){
    return randomReply(replies.hello);
  }

  if(
    t.includes("कैसे हो") ||
    t.includes("how are you")
  ){
    return randomReply(replies.how);
  }

  if(
    t.includes("नाम क्या") ||
    t.includes("your name") ||
    t.includes("नाम")
  ){
    return randomReply(replies.name);
  }

  if(
    t.includes("thank") ||
    t.includes("धन्यवाद") ||
    t.includes("शुक्रिया")
  ){
    return randomReply(replies.thanks);
  }

  if(
    t.includes("bye") ||
    t.includes("बाय")
  ){
    return randomReply(replies.bye);
  }

  if(
    t.includes("help") ||
    t.includes("मदद")
  ){
    return randomReply(replies.help);
  }

  return randomReply(replies.default);
}


/* =========================
   SHOW MESSAGE
========================= */

function addMessage(text,type){

  let div = document.createElement("div");

  div.className = "msg " + type;

  if(type === "user"){

    div.textContent = text;

  }else{

    let safeText = text.replace(/</g,"&lt;").replace(/>/g,"&gt;");

    div.innerHTML = `
      <div>${safeText}</div>

      <div class="buttons">

        <button class="listen"
          onclick="speakText(this.parentElement.parentElement.querySelector('div').textContent)">
          🔊 सुनें
        </button>

        <button class="stop"
          onclick="stopVoice()">
          ⏹ रोकें
        </button>

        <button class="copy"
          onclick="copyText(this.parentElement.parentElement.querySelector('div').textContent)">
          📋 Copy
        </button>

      </div>
    `;
  }

  chatBox.appendChild(div);

  chatBox.scrollTop = chatBox.scrollHeight;
}


/* =========================
   SEND MESSAGE
========================= */

function sendMessage(){

  let input = document.getElementById("input");

  let text = input.value.trim();

  if(!text) return;

  addMessage(text,"user");

  history.push({
    type:"user",
    text:text
  });

  localStorage.setItem(
    "nova_history",
    JSON.stringify(history)
  );

  input.value = "";

  setTimeout(()=>{

    let reply = getReply(text);

    addMessage(reply,"ai");

    history.push({
      type:"ai",
      text:reply
    });

    localStorage.setItem(
      "nova_history",
      JSON.stringify(history)
    );

  },500);
}


/* =========================
   ENTER KEY
========================= */

document.getElementById("input").addEventListener(
  "keydown",
  function(e){

    if(e.key === "Enter"){
      sendMessage();
    }

  }
);


/* =========================
   VOICE
========================= */

function speakText(text){

  if(!("speechSynthesis" in window)){

    alert("इस browser में Text-to-Speech उपलब्ध नहीं है। Chrome में खोलकर देखें।");

    return;
  }

  speechSynthesis.cancel();

  speechSynthesis.resume();

  let words = text.match(/.{1,180}(\s|$)/g) || [text];

  let index = 0;

  function speakNext(){

    if(index >= words.length) return;

    let utterance =
      new SpeechSynthesisUtterance(words[index]);

    utterance.lang = /[अ-ह]/.test(words[index])
      ? "hi-IN"
      : "en-US";

    utterance.rate = 0.95;
    utterance.pitch = 1;

    utterance.onend = function(){
      index++;
      speakNext();
    };

    speechSynthesis.speak(utterance);
  }

  speakNext();
}


/* =========================
   STOP VOICE
========================= */

function stopVoice(){

  if("speechSynthesis" in window){

    speechSynthesis.cancel();

  }
}


/* =========================
   TEST VOICE
========================= */

function testVoice(){

  speakText(
    "नमस्ते! मैं NOVA-X हूँ। मेरी आवाज़ अब काम कर रही है।"
  );

}


/* =========================
   COPY
========================= */

function copyText(text){

  navigator.clipboard.writeText(text)
    .then(()=>{
      alert("कॉपी हो गया ✅");
    })
    .catch(()=>{
      alert("Copy नहीं हो पाया।");
    });

}


/* =========================
   NEW CHAT
========================= */

function newChat(){

  chatBox.innerHTML = "";

  history = [];

  localStorage.removeItem("nova_history");

  lastReply = "";

  addMessage(
    "नया चैट शुरू हो गया 🚀 अब अपना सवाल पूछो।",
    "ai"
  );
}


/* =========================
   CLEAR CHAT
========================= */

function clearChat(){

  chatBox.innerHTML = "";

  history = [];

  localStorage.removeItem("nova_history");

}


/* =========================
   LOAD OLD CHAT
========================= */

function loadHistory(){

  history.forEach(item=>{

    addMessage(
      item.text,
      item.type
    );

  });

}


/* =========================
   START
========================= */

if(history.length > 0){

  loadHistory();

}else{

  addMessage(
    "नमस्ते! 👋 मैं NOVA-X हूँ। मुझसे कुछ भी पूछो।",
    "ai"
  );

}

</script>

</body>
</html>
