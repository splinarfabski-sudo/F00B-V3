# F00B-V3
Public 
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">

<title>F00B V3</title>

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

<h1>F00B V3</h1>

<input id="room" placeholder="Room Code">
<br>
<input id="name" placeholder="Name">
<input id="msg" placeholder="Nachricht">
<br>

<button onclick="join()">Join</button>
<button onclick="send()">Senden</button>
<button onclick="selfDestruct()">💥 Self Destruct</button>

<div id="chat"></div>

<script>
const supabaseUrl = https://jhpmqieqjfgshiskcwea.supabase.co/rest/v1/
const supabaseKey = sb_publishable_wOmLn6M1amR8EPH8r9Osfw__aHAHe_I
const client = supabase.createClient(supabaseUrl, supabaseKey);

let room = "";

function join(){
  room = document.getElementById("room").value;
  load();
}

async function send(){
  await client.from("messages").insert([
    {
      room,
      name: document.getElementById("name").value,
      msg: document.getElementById("msg").value,
      created_at: new Date()
    }
  ]);
}

async function load(){
  const { data } = await client
    .from("messages")
    .select("*")
    .eq("room", room);

  const chat = document.getElementById("chat");
  chat.innerHTML = "";

  data.forEach(m=>{
    const div = document.createElement("div");
    div.innerText = m.name + ": " + m.msg;
    chat.appendChild(div);

    setTimeout(async ()=>{
      await client.from("messages").delete().eq("id", m.id);
    }, 60000);
  });
}

async function selfDestruct(){
  await client.from("messages").delete().eq("room", room);
  chat.innerHTML = "";
  alert("💥 zerstört");
}

setInterval(load, 2000);
</script>

</body>
</html>const supabaseUrl = https://jhpmqieqjfgshiskcwea.supabase.co/rest/v1/
const supabaseKey = sb_publishable_wOmLn6M1amR8EPH8r9Osfw__aHAHe_Iindex.html<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>F00B</title>

<style>
body {
  background:black;
  color:white;
  font-family:sans-serif;
  text-align:center;
}
h1 { color:#00aaff; }
button { padding:10px; background:#007bff; color:white; border:none; }
</style>
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
</html>