
</head>

<body>

<h1>F00B</h1>

<p>Deine App läuft 🚀</p>

<button onclick="alert('F00B funktioniert!')">Test</button>

</body>
</html><!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>F00B V3.2</title>

<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js"></script>

<style>
body { background:black; color:white; font-family:sans-serif; text-align:center; }
h1 { color:#00aaff; }
#chat { height:300px; overflow:auto; border:1px solid #00aaff; margin:10px; padding:10px; }
input, button { padding:10px; margin:5px; }
button { background:#007bff; color:white; border:none; }
</style>
</head>

<body>

<h1>F00B</h1>

<button onclick="createRoom()">Raum erstellen 🔗</button>

<br><br>

<input id="room" placeholder="Raum Code">
<input id="name" placeholder="Name">
<input id="msg" placeholder="Nachricht">

<br>

<button onclick="join()">Raum betreten</button>
<button onclick="send()">Senden</button>
<button onclick="destroy()">💥 Selbstzerstörung</button>

<div id="chat"></div>

<script>

const supabaseUrl = "DEINE_URL";
const supabaseKey = "DEIN_KEY";
const client = supabase.createClient(supabaseUrl, supabaseKey);

let room = "";

// 🔗 Raum erstellen
function createRoom(){
  const id = "F00B-" + Math.floor(Math.random()*9999);
  room = id;

  document.getElementById("room").value = id;

  const link = window.location.href + "?room=" + id;

  alert("Raum erstellt:\n\n" + link);
}

// 🚪 Raum join
function join(){
  room = document.getElementById("room").value;
  load();
  listen();
}

// 💬 senden
async function send(){
  await client.from("messages").insert([
    {
      room,
      name: document.getElementById("name").value,
      msg: document.getElementById("msg").value
    }
  ]);
}

// 📥 laden
async function load(){
  const { data } = await client
    .from("messages")
    .select("*")
    .eq("room", room);

  document.getElementById("chat").innerHTML = "";

  data.forEach(m=>{
    const div = document.createElement("div");
    div.innerText = m.name + ": " + m.msg;
    document.getElementById("chat").appendChild(div);
  });
}

// 📡 realtime
function listen(){
  client
    .channel('messages')
    .on('postgres_changes',
      { event: 'INSERT', schema: 'public', table: 'messages' },
      payload => {
        if(payload.new.room === room){
          const div = document.createElement("div");
          div.innerText = payload.new.name + ": " + payload.new.msg;
          document.getElementById("chat").appendChild(div);
        }
      }
    )
    .subscribe();
}

// 💥 löschen
async function destroy(){
  await client.from("messages").delete().eq("room", room);
  document.getElementById("chat").innerHTML = "";
  alert("Selbstzerstörung aktiviert 💥");
}

// 🔗 Auto join via link
window.onload = function(){
  const url = new URLSearchParams(window.location.search);
  const r = url.get("room");

  if(r){
    document.getElementById("room").value = r;
    room = r;
    load();
    listen();
  }
}

</script>

</body>
</html>await client.from("messages").insert([
  {
    room,
    name,
    msg,
    created_at: new Date()
  }
]);async function scheduleDelete(id){
  setTimeout(async ()=>{
    await client.from("messages").delete().eq("id", id);
  }, 60000);
}function addMessage(m){
  const div = document.createElement("div");
  div.innerText = m.name + ": " + m.msg;
  document.getElementById("chat").appendChild(div);

  // 💣 nach 60 Sekunden entfernen (UI)
  setTimeout(()=>{
    div.remove();
  }, 60000);

  // 💥 Server löschen nach 60 Sekunden
  scheduleDelete(m.id);
}async function send(){
  const { data } = await client.from("messages")
  .insert([
    {
      room,
      name,
      msg,
      created_at: new Date()
    }
  ])
  .select(); // 👈 wichtig

  scheduleDelete(data[0].id);
}<label>
  <input type="checkbox" id="encrypt">
  🔐 Nachricht verschlüsseln
</label>function encrypt(text, key){
  return btoa(unescape(encodeURIComponent(text + "::" + key)));
}

function decrypt(data, key){
  try {
    const decoded = decodeURIComponent(escape(atob(data)));
    const [msg, k] = decoded.split("::");
    if(k === key) return msg;
    return "🔒 Verschlüsselte Nachricht";
  } catch(e){
    return "Fehler";
  }
}let room = "";async function send(){
  const text = document.getElementById("msg").value;
  const isEncrypted = document.getElementById("encrypt").checked;

  let finalMsg = text;

  if(isEncrypted){
    finalMsg = encrypt(text, room);
  }

  await client.from("messages").insert([
    {
      room,
      name,
      msg: finalMsg
    }
  ]);
}function addMessage(m){
  const div = document.createElement("div");

  const isEncrypted = m.msg.includes("::");

  let text = m.msg;

  if(isEncrypted){
    text = decrypt(m.msg, room);
  }

  div.innerText = m.name + ": " + text;

  document.getElementById("chat").appendChild(div);

  setTimeout(()=>div.remove(), 60000);
}async function generateKey(){
  return crypto.subtle.generateKey(
    { name: "AES-GCM", length: 256 },
    true,
    ["encrypt", "decrypt"]
  );
}async function exportKey(key){
  const raw = await crypto.subtle.exportKey("jwk", key);
  return btoa(JSON.stringify(raw));
}async function importKey(jwkString){
  const jwk = JSON.parse(atob(jwkString));

  return crypto.subtle.importKey(
    "jwk",
    jwk,
    { name: "AES-GCM" },
    true,
    ["encrypt", "decrypt"]
  );
}async function encryptMsg(key, text){
  const iv = crypto.getRandomValues(new Uint8Array(12));

  const encoded = new TextEncoder().encode(text);

  const cipher = await crypto.subtle.encrypt(
    { name: "AES-GCM", iv },
    key,
    encoded
  );

  return {
    data: btoa(String.fromCharCode(...new Uint8Array(cipher))),
    iv: btoa(String.fromCharCode(...iv))
  };
}

async function decryptMsg(key, data, iv){
  const cipher = Uint8Array.from(atob(data), c => c.charCodeAt(0));
  const ivArr = Uint8Array.from(atob(iv), c => c.charCodeAt(0));

  const plain = await crypto.subtle.decrypt(
    { name: "AES-GCM", iv: ivArr },
    key,
    cipher
  );

  return new TextDecoder().decode(plain);
}let roomKey;

async function createRoom(){
  roomKey = await generateKey();

  const exported = await exportKey(roomKey);

  const roomId = "F00B-" + Math.floor(Math.random()*9999);

  const link = window.location.origin + "?room=" + roomId + "&key=" + exported;

  alert("Raum erstellt:\n\n" + link);
}window.onload = async function(){
  const url = new URLSearchParams(window.location.search);

  const key = url.get("key");
  const r = url.get("room");

  if(key && r){
    room = r;
    roomKey = await importKey(key);

    load();
    listen();
  }
}async function send(){
  const text = document.getElementById("msg").value;

  const encrypted = await encryptMsg(roomKey, text);

  await client.from("messages").insert([
    {
      room,
      name,
      msg: encrypted.data,
      iv: encrypted.iv
    }
  ]);
}async function addMessage(m){
  const div = document.createElement("div");

  const text = await decryptMsg(roomKey, m.msg, m.iv);

  div.innerText = m.name + ": " + text;

  document.getElementById("chat").appendChild(div);

  setTimeout(()=>div.remove(), 60000);
}let channel;

function listen(){
  if(channel) supabase.removeChannel(channel);

  channel = client
    .channel('messages')
    .on('postgres_changes',
      { event: 'INSERT', schema: 'public', table: 'messages' },
      payload => {
        if(payload.new.room === room){
          addMessage(payload.new);
        }
      }
    )
    .subscribe();
}async function load(){
  const { data } = await client
    .from("messages")
    .select("*")
    .eq("room", room)
    .order("created_at", { ascending: true })
    .limit(50); // WICHTIG

  document.getElementById("chat").innerHTML = "";
  data.forEach(addMessage);
}let sending = false;

async function send(){
  if(sending) return;
  sending = true;

  await client.from("messages").insert([
    {
      room,
      name,
      msg
    }
  ]);

  sending = false;
}setInterval(async ()=>{
  if(!room) return;

  await client
    .from("messages")
    .delete()
    .lt("created_at", new Date(Date.now() - 60000));
}, 30000);function createRoom(){
  const id = "F00B-" + Math.floor(Math.random()*9999);
  room = id;

  document.getElementById("room").value = id;

  const base = window.location.origin + window.location.pathname;
  const link = base + "?room=" + id;

  alert("Raum erstellt:\n\n" + link);
}window.onload = function(){
  const url = new URLSearchParams(window.location.search);
  const r = url.get("room");

  if(r){
    room = r;
    document.getElementById("room").value = r;
    load();
    listen();
  }
}
