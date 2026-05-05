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
const supabaseUrl = "DEINE_URL";
const supabaseKey = "DEIN_KEY";
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
const supabaseKey = sb_publishable_wOmLn6M1amR8EPH8r9Osfw__aHAHe_I