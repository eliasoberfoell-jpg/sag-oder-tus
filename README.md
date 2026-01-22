 <!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Teenkreis Game Loop</title>
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700&display=swap" rel="stylesheet">
<style>
* { box-sizing: border-box; }
body { font-family: 'Montserrat', sans-serif; margin:0; padding:0; min-height:100vh;
       display:flex; flex-direction:column; justify-content:center; align-items:center;
       background: linear-gradient(135deg, #ff9a9e 0%, #fad0c4 100%);
       color:#fff; text-align:center; }
h1 { font-size:2.5rem; margin-bottom:1rem; text-shadow: 2px 2px 6px rgba(0,0,0,0.2); }
#idea-box { background: rgba(255,255,255,0.15); backdrop-filter: blur(10px);
            border-radius:25px; padding:40px 30px; margin:20px; min-width:280px;
            max-width:90%; font-size:1.3rem; font-weight:500;
            box-shadow:0 8px 25px rgba(0,0,0,0.2); transition: all 0.3s ease; }
.button-container { display:flex; gap:20px; flex-wrap:wrap; justify-content:center; }
button { background:#fff; color:#ff6f91; border:none; padding:15px 35px; border-radius:50px;
         font-size:1.2rem; font-weight:600; cursor:pointer; transition: all 0.2s ease;
         box-shadow: 0 6px 20px rgba(0,0,0,0.15);}
button:hover { transform: translateY(-5px) scale(1.05); background:#ffe3e6; }
#mode-menu { position:absolute; top:20px; right:20px; background: rgba(255,255,255,0.85);
             color:#ff6f91; padding:10px 15px; border-radius:15px; font-weight:bold;
             cursor:pointer; display:flex; gap:8px; flex-wrap:wrap;
             box-shadow:0 4px 15px rgba(0,0,0,0.2); }
.mode-btn { padding:5px 10px; border-radius:12px; border:2px solid transparent;
            background: rgba(255,255,255,0.6); color:#ff6f91; font-weight:bold;
            cursor:pointer; transition: all 0.2s ease; }
.mode-btn.selected { background:#fff; border:2px solid #ff6f91; font-weight:bold; }
@media (max-width:500px){ h1{font-size:2rem;} #idea-box{font-size:1.1rem; padding:30px 20px;}
button{padding:12px 25px; font-size:1rem;} #mode-menu{top:15px; right:10px;} }
</style>
</head>
<body>
<h1>Teenkreis Game Loop</h1>
<div id="idea-box">Wähle einen Modus, um zu starten!</div>

<div class="button-container">
  <button onclick="showIdea('tus')">TU’S</button>
  <button onclick="showIdea('sags')">SAGS</button>
</div>

<div id="mode-menu">
  <div class="mode-btn selected" onclick="toggleMode('Hot')">Hot</div>
  <div class="mode-btn selected" onclick="toggleMode('Church')">Church</div>
  <div class="mode-btn selected" onclick="toggleMode('Friends')">Friends</div>
</div>

<script>
let startTime = Date.now();
let selectedModes = ["Hot","Church","Friends"];
let tusList = [];
let sagsList = [];

// -------------------- 120 Aufgaben pro Kategorie --------------------
const ideas = {
  Hot: {
    tus: [
      "Flirte 1 Minute mit jemandem.",
      "Mache eine Grimasse, die jemand erraten muss.",
      "Schicke eine freche Nachricht an einen Freund.",
      "Imitiere eine Liebesszene aus einem Film.",
      "Tanze 30 Sekunden wie dein Lieblingsschauspieler beim Flirten.",
      "Kreiere einen Spitznamen für jemanden und sage ihn laut.",
      "Balanciere 20 Sekunden und tue dabei flirtend.",
      "Mach ein freches Selfie.",
      "Zeige dein wildestes Tanz-Move-Flirt.",
      "Stelle eine Mini-Romanze pantomimisch dar.",
      "Küsse imaginär eine Hand.",
      "Tue so, als würdest du jemanden verführen, ohne Worte.",
      "Mach 10 Hampelmänner mit Liebesruf.",
      "Erzähle pantomimisch ein Date-Erlebnis.",
      "Sprich 30 Sekunden in Liebesreimen.",
      "Führe eine Mini-Romanze vor.",
      "Tue so, als würdest du ein geheimes Date planen.",
      "Zeige deine „verführerische Pose“ für 30 Sekunden.",
      "Irritiere jemanden kurz spielerisch durch Flirtgesten.",
      "Stelle ein romantisches Emoji-Spiel dar.",
      // ... füge hier bis 120 Aufgaben ein, ähnlich wie oben ...
    ],
    sags: [
      "Erzähle von deinem ersten Crush.",
      "Was war dein peinlichster Flirt?",
      "Mit wem hättest du am liebsten ein Date?",
      "Beschreibe dein perfektes Liebesgeständnis.",
      "Wer ist dein heimlicher Schwarm?",
      "Was würdest du tun, wenn du für 24h begehrenswert wärst?",
      "Erzähle dein wildestes Date-Erlebnis.",
      "Nenne dein Lieblings-Kompliment, das du je bekommen hast.",
      "Hast du schon mal jemanden heimlich beobachtet?",
      "Was war dein verrücktester Liebeswunsch?",
      // ... 110 weitere Aufgaben ergänzen ...
    ]
  },
  Church: {
    tus: [
      "Führe eine kurze Gebetspose aus.",
      "Lies einen Bibelvers laut vor.",
      "Zeige mit Gesten, was Dankbarkeit bedeutet.",
      "Tue so, als würdest du ein Bibelzitat darstellen.",
      "Sprich 30 Sekunden über etwas, wofür du Gott dankst.",
      "Mache eine kurze inspirierende Pose.",
      "Zeige, wie man Liebe im Alltag lebt.",
      "Sprich eine kurze Bitte oder Fürbitte aus.",
      "Pantomimiere eine bekannte Bibelszene.",
      "Zeige mit Handbewegungen Vertrauen.",
      // ... 110 weitere Aufgaben ergänzen ...
    ],
    sags: [
      "Welcher Bibelvers inspiriert dich am meisten?",
      "Wie würdest du deinen Glauben jemandem erklären?",
      "Welche gute Tat hast du heute getan?",
      "Erzähle von einem Moment, in dem du Gott gespürt hast.",
      "Was bedeutet Vertrauen für dich?",
      "Welche Bibelgeschichte magst du am liebsten?",
      "Wie lebst du Nächstenliebe?",
      "Welche Gebete sind dir wichtig?",
      "Erzähle von deinem ersten Gottesdiensterlebnis.",
      "Welche Charaktereigenschaft Gottes bewunderst du am meisten?",
      // ... 110 weitere Aufgaben ergänzen ...
    ]
  },
  Friends: {
    tus: [
      "Mach ein Selfie mit einem Freund.",
      "Erzähle einen Insider-Witz.",
      "Führe eine kleine Freundschafts-Challenge durch.",
      "Mache eine Pose, die eure Freundschaft zeigt.",
      "Organisiere ein kurzes lustiges Spiel.",
      "Imitiere eine witzige Szene aus eurer Freundschaft.",
      "Tanze mit einem Freund 30 Sekunden.",
      "Führe ein Mini-Rollenspiel über ein Abenteuer durch.",
      "Male ein Bild für einen Freund.",
      "Mach 10 Hampelmänner mit deinem Freund.",
      // ... 110 weitere Aufgaben ergänzen ...
    ],
    sags: [
      "Wer ist dein bester Freund und warum?",
      "Was war euer verrücktestes Abenteuer?",
      "Welche Erinnerung aus eurer Kindheit verbindet euch am meisten?",
      "Was macht eure Freundschaft besonders?",
      "Welche Freundschaftsregel ist dir wichtig?",
      "Was war euer peinlichster Moment zusammen?",
      "Wer ist der witzigste in eurer Gruppe?",
      "Erzähle von einer Situation, in der ihr euch gegenseitig geholfen habt.",
      "Was würdest du an eurer Freundschaft ändern?",
      "Welche gemeinsame Aktivität macht euch am meisten Spaß?",
      // ... 110 weitere Aufgaben ergänzen ...
    ]
  }
};

// Shuffle-Funktion
function shuffle(array){ let a=[...array]; for(let i=a.length-1;i>0;i--){let j=Math.floor(Math.random()*(i+1));[a[i],a[j]]=[a[j],a[i]];} return a; }

// Toggle Modus ein/aus
function toggleMode(mode){
  const index = selectedModes.indexOf(mode);
  if(index>-1) selectedModes.splice(index,1);
  else selectedModes.push(mode);

  document.querySelectorAll('.mode-btn').forEach(btn=>{
    if(selectedModes.includes(btn.innerText)) btn.classList.add('selected');
    else btn.classList.remove('selected');
  });
  rebuildLists();
}

// Listen für gewählte Modi zusammenstellen
function rebuildLists(){
  tusList = [];
  sagsList = [];
  selectedModes.forEach(m=>{
    tusList = tusList.concat(shuffle([...ideas[m].tus]));
    sagsList = sagsList.concat(shuffle([...ideas[m].sags]));
  });
}

// initial Listen erstellen
rebuildLists();

function showIdea(type){
  if(Date.now()-startTime>21600000){
    document.getElementById('idea-box').innerText="Die 6 Stunden sind vorbei! Spiel neu starten.";
    return;
  }
  let list = type==='tus'?tusList:sagsList;
  if(list.length===0){
    document.getElementById('idea-box').innerText="Keine Modi ausgewählt oder alle Aufgaben durch! Wähle Modi neu.";
    return;
  }
  const box=document.getElementById('idea-box');
  const selected=list.shift();
  box.style.transform='scale(0.95)';
  setTimeout(()=>{box.innerText=selected;box.style.transform='scale(1)';},150);
}
</script>
</body>
</html>
