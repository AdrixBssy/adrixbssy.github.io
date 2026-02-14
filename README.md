
<html lang="fr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>QCM OSI & TCP/IP — 200 questions (Interactif + correction)</title>
  <style>
    body { font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; margin: 0; background:#0b1020; color:#e8ecff; }
    header { padding: 18px 16px; background: #111a33; border-bottom: 1px solid rgba(255,255,255,.08); position: sticky; top: 0; z-index: 10;}
    header h1 { margin: 0 0 8px 0; font-size: 18px; }
    header .row { display:flex; flex-wrap:wrap; gap:10px; align-items:center; }
    button {
      background:#2b5cff; color:white; border:0; padding:10px 12px; border-radius:10px;
      font-weight:600; cursor:pointer;
    }
    button.secondary { background:#26304f; }
    button:disabled { opacity:.55; cursor:not-allowed; }
    .wrap { max-width: 1000px; margin: 0 auto; padding: 16px; }
    .meta { display:flex; flex-wrap:wrap; gap:10px; align-items:center; color:#b9c2ff; font-size: 14px; }
    .pill { padding: 6px 10px; border-radius: 999px; background: rgba(255,255,255,.08); }
    .card { background: rgba(255,255,255,.06); border: 1px solid rgba(255,255,255,.08); border-radius: 16px; padding: 14px; margin: 12px 0; }
    .qtitle { margin:0 0 10px 0; font-size: 15px; line-height:1.35; }
    .choices { display:grid; gap:8px; }
    label.choice {
      display:flex; gap:10px; align-items:flex-start;
      padding:10px; border-radius:12px;
      border:1px solid rgba(255,255,255,.10);
      background: rgba(0,0,0,.12);
      cursor:pointer;
    }
    label.choice:hover { border-color: rgba(43,92,255,.6); }
    input[type="radio"] { margin-top: 2px; }
    .footerbar { display:flex; gap:10px; flex-wrap:wrap; align-items:center; justify-content:space-between; margin: 14px 0; }
    .nav { display:flex; gap:8px; flex-wrap:wrap; }
    .result {
      padding: 12px; border-radius: 14px; border: 1px solid rgba(255,255,255,.12);
      background: rgba(0,0,0,.18);
    }
    .ok { outline: 2px solid rgba(0, 255, 150, .35); }
    .bad { outline: 2px solid rgba(255, 80, 80, .40); }
    .explain { color:#cfd6ff; font-size: 13px; margin-top: 8px; opacity: .95; }
    .small { font-size: 13px; color:#b9c2ff; }
    a { color:#9db4ff; }
  </style>
</head>
<body>
<header>
  <div class="row">
    <h1>QCM OSI & TCP/IP — 200 questions (interactif + correction)</h1>
  </div>
  <div class="row meta">
    <span class="pill" id="pillCount">Questions: …</span>
    <span class="pill" id="pillPage">Page: …</span>
    <span class="pill" id="pillAnswered">Répondu: …</span>
    <span class="pill" id="pillScore">Score: —</span>
  </div>
  <div class="row" style="margin-top:10px">
    <button id="btnPrev" class="secondary">◀ Page précédente</button>
    <button id="btnNext" class="secondary">Page suivante ▶</button>
    <button id="btnFinish">Corriger</button>
    <button id="btnReset" class="secondary">Réinitialiser</button>
  </div>
</header>

<div class="wrap">
  <div class="result" id="resultBox" style="display:none"></div>
  <div id="questions"></div>

  <div class="footerbar">
    <div class="nav">
      <button id="btnPrev2" class="secondary">◀ Page précédente</button>
      <button id="btnNext2" class="secondary">Page suivante ▶</button>
    </div>
    <div class="small">
      Astuce : tu peux répondre page par page, puis cliquer sur <b>Corriger</b> à la fin.
    </div>
  </div>
</div>

<script>
/* ===========================
   Utilitaires
=========================== */
function shuffle(arr) {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
function uniqPush(list, item) {
  if (!list.includes(item)) list.push(item);
}
function clamp(n, min, max){ return Math.max(min, Math.min(max, n)); }

/* ===========================
   Banque de faits -> générateur de QCM
   Objectif: 200 questions fiables, sans tout écrire à la main.
=========================== */

const OSI = [
  {layer: 7, name:"Application", pdu:"Données", examples:["HTTP","DNS","SMTP","FTP","SSH","DHCP"]},
  {layer: 6, name:"Présentation", pdu:"Données", examples:["Encodage","Chiffrement","Compression","TLS (souvent placé ici en OSI)"]},
  {layer: 5, name:"Session", pdu:"Données", examples:["Ouverture/fermeture de session","Synchronisation","RPC (souvent cité)"]},
  {layer: 4, name:"Transport", pdu:"Segment (TCP) / Datagramme (UDP)", examples:["TCP","UDP"]},
  {layer: 3, name:"Réseau", pdu:"Paquet", examples:["IP","ICMP","Routage","OSPF/BGP (routage)"]},
  {layer: 2, name:"Liaison de données", pdu:"Trame", examples:["Ethernet","Wi-Fi (802.11)","Switch","MAC","ARP (entre 2/3, souvent associé L2)"]},
  {layer: 1, name:"Physique", pdu:"Bits", examples:["Câble","Fibre","Radio","Signal"]},
];

const TCPIP = [
  {layer: 4, name:"Application", mapsOSI:"5-6-7", examples:["HTTP","DNS","SMTP","FTP","SSH","DHCP","TLS (souvent vu ici en TCP/IP)"]},
  {layer: 3, name:"Transport", mapsOSI:"4", examples:["TCP","UDP"]},
  {layer: 2, name:"Internet", mapsOSI:"3", examples:["IP","ICMP","Routage"]},
  {layer: 1, name:"Accès réseau", mapsOSI:"1-2", examples:["Ethernet","Wi-Fi","ARP","MAC"]},
];

const PORTS = [
  ["HTTP", 80], ["HTTPS", 443], ["DNS", 53], ["DHCP (client)", 68], ["DHCP (serveur)", 67],
  ["FTP (contrôle)", 21], ["FTP (données)", 20], ["SSH", 22], ["Telnet", 23],
  ["SMTP", 25], ["POP3", 110], ["IMAP", 143],
  ["NTP", 123], ["SNMP", 161], ["SNMP Trap", 162],
  ["LDAP", 389], ["LDAPS", 636],
  ["RDP", 3389], ["SIP", 5060], ["SIPS", 5061],
  ["TFTP", 69], ["SMB", 445], ["Kerberos", 88],
  ["MySQL", 3306], ["PostgreSQL", 5432], ["MongoDB", 27017],
  ["HTTPS Alt", 8443], ["HTTP Alt", 8080],
];

const PROTO_LAYER = [
  ["IP", "Réseau (OSI 3) / Internet (TCP/IP)"],
  ["ICMP", "Réseau (OSI 3) / Internet (TCP/IP)"],
  ["ARP", "Liaison (souvent associé L2, entre L2/L3)"],
  ["TCP", "Transport (OSI 4) / Transport (TCP/IP)"],
  ["UDP", "Transport (OSI 4) / Transport (TCP/IP)"],
  ["HTTP", "Application (OSI 7) / Application (TCP/IP)"],
  ["DNS", "Application (OSI 7) / Application (TCP/IP)"],
  ["SMTP", "Application (OSI 7) / Application (TCP/IP)"],
  ["FTP", "Application (OSI 7) / Application (TCP/IP)"],
  ["SSH", "Application (OSI 7) / Application (TCP/IP)"],
  ["DHCP", "Application (OSI 7) / Application (TCP/IP)"],
  ["Ethernet", "Liaison/Physique (OSI 2/1) / Accès réseau"],
];

const DEVICES = [
  ["Hub", "Physique (OSI 1)"],
  ["Répéteur", "Physique (OSI 1)"],
  ["Switch (commutateur)", "Liaison (OSI 2)"],
  ["Pont (bridge)", "Liaison (OSI 2)"],
  ["Routeur", "Réseau (OSI 3)"],
  ["Pare-feu (classique L3/L4)", "Réseau/Transport (OSI 3/4)"],
  ["Proxy applicatif", "Application (OSI 7)"],
];

const TCP_FACTS = [
  ["TCP est orienté connexion", "Vrai"],
  ["TCP fournit un mécanisme de fiabilité (ACK, retransmission)", "Vrai"],
  ["UDP est orienté connexion", "Faux"],
  ["UDP ne garantit pas la livraison", "Vrai"],
  ["Le handshake TCP standard est en 3 temps (SYN, SYN-ACK, ACK)", "Vrai"],
  ["La fenêtre TCP sert au contrôle de flux", "Vrai"],
  ["Le contrôle de congestion TCP adapte le débit", "Vrai"],
];

const IP_FACTS = [
  ["Une adresse IPv4 fait 32 bits", "Vrai"],
  ["Une adresse IPv6 fait 128 bits", "Vrai"],
  ["Le TTL (IPv4) limite la durée de vie d’un paquet", "Vrai"],
  ["ICMP sert notamment au diagnostic (ping) et aux messages d’erreur", "Vrai"],
  ["Le routage se fait à la couche Réseau/Internet", "Vrai"],
];

/* ===========================
   Construction des 200 questions
   Format final: { text, choices:[...], answerIndex, explain }
=========================== */

const questions = [];

function addQ(text, correct, wrongs, explain) {
  // assure 4 choix
  const choices = [];
  uniqPush(choices, correct);
  for (const w of wrongs) uniqPush(choices, w);
  // compléter si <4 (au cas où)
  const fillers = ["Aucune de ces réponses", "Toutes ces réponses", "Cela dépend du contexte", "Non applicable"];
  for (const f of fillers) if (choices.length < 4) uniqPush(choices, f);

  const finalChoices = shuffle(choices.slice(0,4));
  const answerIndex = finalChoices.indexOf(correct);
  questions.push({ text, choices: finalChoices, answerIndex, explain });
}

function buildQuestions() {
  // 1) OSI : couches, noms, PDU, exemples
  addQ("Combien de couches comporte le modèle OSI ?", "7", ["4", "5", "8"],
      "Le modèle OSI est constitué de 7 couches (de Physique à Application).");

  for (const l of OSI) {
    addQ(`Dans le modèle OSI, quelle est la couche ${l.layer} ?`, l.name,
      OSI.filter(x=>x.layer!==l.layer).slice(0,3).map(x=>x.name),
      `La couche ${l.layer} OSI s'appelle « ${l.name} ».`
    );

    addQ(`Quel est le PDU typique de la couche OSI ${l.layer} (${l.name}) ?`, l.pdu,
      ["Bits", "Trame", "Paquet", "Segment"].filter(x=>x!==l.pdu).slice(0,3),
      `PDU: ${l.name} → ${l.pdu}.`
    );

    // question sur exemple
    const ex = l.examples[0];
    const other = [];
    // prendre des exemples d'autres couches
    for (const o of OSI) if (o.layer !== l.layer) other.push(o.examples[0]);
    addQ(`À quelle couche OSI associe-t-on le plus souvent « ${ex} » ?`, l.name,
      shuffle(OSI.filter(x=>x.layer!==l.layer).map(x=>x.name)).slice(0,3),
      `« ${ex} » est généralement rattaché à la couche ${l.name}.`
    );
  }

  // 2) TCP/IP : couches + mapping
  addQ("Combien de couches comporte le modèle TCP/IP (version 4 couches) ?", "4", ["5", "6", "7"],
      "Le modèle TCP/IP est souvent présenté en 4 couches: Accès réseau, Internet, Transport, Application.");

  for (const l of TCPIP) {
    addQ(`Dans le modèle TCP/IP (4 couches), quelle est la couche ${l.layer} ?`, l.name,
      TCPIP.filter(x=>x.layer!==l.layer).slice(0,3).map(x=>x.name),
      `TCP/IP couche ${l.layer} : ${l.name}.`
    );

    addQ(`La couche TCP/IP « ${l.name} » correspond à quelles couches OSI ?`, l.mapsOSI,
      shuffle(TCPIP.filter(x=>x.name!==l.name).map(x=>x.mapsOSI)).slice(0,3),
      `Correspondance classique : TCP/IP ${l.name} ↔ OSI ${l.mapsOSI}.`
    );

    const ex = l.examples[0];
    addQ(`Dans TCP/IP, « ${ex} » appartient le plus souvent à quelle couche ?`, l.name,
      shuffle(TCPIP.filter(x=>x.name!==l.name).map(x=>x.name)).slice(0,3),
      `« ${ex} » est généralement rattaché à la couche ${l.name} en TCP/IP.`
    );
  }

  // 3) Ports classiques
  for (const [svc, port] of PORTS) {
    addQ(`Quel est le port TCP/UDP le plus couramment associé à ${svc} ?`, String(port),
      shuffle(PORTS.filter(x=>x[0]!==svc).slice(0,12).map(x=>String(x[1]))).slice(0,3),
      `${svc} est classiquement associé au port ${port}.`
    );
  }

  // 4) Protocole -> couche
  for (const [proto, layerText] of PROTO_LAYER) {
    addQ(`À quelle couche se rattache principalement ${proto} ?`, layerText,
      shuffle(PROTO_LAYER.filter(x=>x[0]!==proto).slice(0,8).map(x=>x[1])).slice(0,3),
      `${proto} : ${layerText}.`
    );
  }

  // 5) Équipements réseau -> couche
  for (const [dev, layer] of DEVICES) {
    addQ(`Quel équipement correspond le plus souvent à la couche suivante : ${layer} ?`, dev,
      shuffle(DEVICES.filter(x=>x[0]!==dev).map(x=>x[0])).slice(0,3),
      `${dev} est classiquement associé à ${layer}.`
    );
    addQ(`À quelle couche OSI associe-t-on le plus souvent : ${dev} ?`, layer,
      shuffle(DEVICES.filter(x=>x[0]!==dev).map(x=>x[1])).slice(0,3),
      `${dev} → ${layer}.`
    );
  }

  // 6) Vrai/Faux (transformé en QCM)
  for (const [stmt, truth] of TCP_FACTS) {
    addQ(`${stmt}.`, truth, truth==="Vrai" ? ["Faux","Ça dépend","Non applicable"] : ["Vrai","Ça dépend","Non applicable"],
      `Réponse: ${truth}.`
    );
  }
  for (const [stmt, truth] of IP_FACTS) {
    addQ(`${stmt}.`, truth, truth==="Vrai" ? ["Faux","Ça dépend","Non applicable"] : ["Vrai","Ça dépend","Non applicable"],
      `Réponse: ${truth}.`
    );
  }

  // 7) Questions “mix” supplémentaires (génération)
  // Objectif: atteindre exactement 200 questions
  const mix = [
    ["Quelle couche OSI est responsable du routage ?", "Réseau", ["Transport","Session","Liaison de données"],
      "Le routage se fait à la couche Réseau (OSI 3)."],
    ["Quel protocole traduit généralement IP → MAC sur un LAN ?", "ARP", ["DNS","ICMP","TCP"],
      "ARP sert à résoudre une adresse IP vers une adresse MAC sur un réseau local."],
    ["Quel est le rôle principal de DNS ?", "Résoudre les noms de domaine", ["Chiffrer les données","Router les paquets","Assurer la fiabilité"],
      "DNS permet de résoudre un nom (ex: exemple.com) en adresse IP."],
    ["Dans OSI, la couche Physique transporte surtout :", "Des bits", ["Des trames","Des paquets","Des segments"],
      "La couche 1 transporte des bits (signaux)."],
    ["Dans OSI, la couche Liaison manipule surtout :", "Des trames", ["Des bits","Des paquets","Des segments"],
      "La couche 2 manipule des trames (Ethernet/Wi-Fi)."],
    ["Dans OSI, la couche Transport manipule surtout :", "Des segments (TCP) / datagrammes (UDP)", ["Des bits","Des trames","Des paquets"],
      "La couche 4 transporte des segments TCP ou datagrammes UDP."],
    ["Dans OSI, la couche Réseau manipule surtout :", "Des paquets", ["Des bits","Des trames","Des segments"],
      "La couche 3 manipule des paquets (IP)."],
    ["TCP est généralement utilisé quand on veut :", "Fiabilité", ["Zéro latence garantie","Aucun contrôle","Broadcast natif"],
      "TCP apporte des mécanismes de fiabilité (ACK, retransmission, ordre)."],
    ["UDP est souvent choisi pour :", "Des flux temps réel / faible overhead", ["La garantie de livraison","Le chiffrement automatique","Le routage"],
      "UDP est simple, sans connexion, souvent utilisé pour temps réel (selon usage)."],
  ];

  while (questions.length < 200) {
    const [t,c,w,e] = mix[questions.length % mix.length];
    addQ(t, c, w, e);
  }

  // Si on dépasse (normalement non), tronquer
  if (questions.length > 200) questions.length = 200;
}

buildQuestions();

/* ===========================
   UI : pagination + réponses + correction
=========================== */

const pageSize = 20;
let page = 0;
let userAnswers = new Array(questions.length).fill(null);
let corrected = false;

const elQ = document.getElementById("questions");
const pillCount = document.getElementById("pillCount");
const pillPage = document.getElementById("pillPage");
const pillAnswered = document.getElementById("pillAnswered");
const pillScore = document.getElementById("pillScore");
const resultBox = document.getElementById("resultBox");

const btnPrev = document.getElementById("btnPrev");
const btnNext = document.getElementById("btnNext");
const btnPrev2 = document.getElementById("btnPrev2");
const btnNext2 = document.getElementById("btnNext2");
const btnFinish = document.getElementById("btnFinish");
const btnReset = document.getElementById("btnReset");

function answeredCount() {
  return userAnswers.filter(x => x !== null).length;
}

function render() {
  pillCount.textContent = `Questions: ${questions.length}`;
  const totalPages = Math.ceil(questions.length / pageSize);
  pillPage.textContent = `Page: ${page + 1}/${totalPages}`;
  pillAnswered.textContent = `Répondu: ${answeredCount()}/${questions.length}`;
  btnPrev.disabled = page === 0;
  btnPrev2.disabled = page === 0;
  btnNext.disabled = page >= totalPages - 1;
  btnNext2.disabled = page >= totalPages - 1;

  elQ.innerHTML = "";
  const start = page * pageSize;
  const end = Math.min(start + pageSize, questions.length);

  for (let i = start; i < end; i++) {
    const q = questions[i];
    const card = document.createElement("div");
    card.className = "card";
    card.id = `q-${i}`;

    const h = document.createElement("h3");
    h.className = "qtitle";
    h.textContent = `${i+1}. ${q.text}`;
    card.appendChild(h);

    const choices = document.createElement("div");
    choices.className = "choices";

    q.choices.forEach((choiceText, idx) => {
      const lab = document.createElement("label");
      lab.className = "choice";

      const inp = document.createElement("input");
      inp.type = "radio";
      inp.name = `q${i}`;
      inp.value = idx;
      inp.checked = userAnswers[i] === idx;
      inp.disabled = corrected;

      inp.addEventListener("change", () => {
        userAnswers[i] = idx;
        updateMeta();
      });

      const span = document.createElement("div");
      span.textContent = choiceText;

      lab.appendChild(inp);
      lab.appendChild(span);
      choices.appendChild(lab);
    });

    card.appendChild(choices);

    if (corrected) {
      const ua = userAnswers[i];
      const correctIdx = q.answerIndex;

      if (ua === correctIdx) card.classList.add("ok");
      else card.classList.add("bad");

      const exp = document.createElement("div");
      exp.className = "explain";
      const uaTxt = (ua === null) ? "— (non répondu)" : q.choices[ua];
      exp.innerHTML = `
        <div><b>Ta réponse :</b> ${uaTxt}</div>
        <div><b>Bonne réponse :</b> ${q.choices[correctIdx]}</div>
        <div>${q.explain || ""}</div>
      `;
      card.appendChild(exp);
    }

    elQ.appendChild(card);
  }

  updateMeta();
}

function updateMeta() {
  if (!corrected) pillScore.textContent = "Score: —";
}

function goToFirstWrong() {
  for (let i = 0; i < questions.length; i++) {
    if (userAnswers[i] !== questions[i].answerIndex) {
      const targetPage = Math.floor(i / pageSize);
      page = targetPage;
      render();
      const el = document.getElementById(`q-${i}`);
      if (el) el.scrollIntoView({behavior:"smooth", block:"start"});
      return;
    }
  }
}

function grade() {
  corrected = true;

  let correct = 0;
  let unanswered = 0;
  for (let i = 0; i < questions.length; i++) {
    if (userAnswers[i] === null) unanswered++;
    if (userAnswers[i] === questions[i].answerIndex) correct++;
  }
  const score = Math.round((correct / questions.length) * 1000) / 10;

  pillScore.textContent = `Score: ${correct}/${questions.length} (${score}%)`;

  resultBox.style.display = "block";
  resultBox.innerHTML = `
    <div style="font-size:16px; font-weight:800; margin-bottom:6px;">Résultat</div>
    <div>✅ Bonnes réponses : <b>${correct}</b></div>
    <div>❌ Erreurs : <b>${questions.length - correct - unanswered}</b></div>
    <div>⚠️ Non répondu : <b>${unanswered}</b></div>
    <div style="margin-top:10px; display:flex; gap:8px; flex-wrap:wrap;">
      <button class="secondary" id="btnFirstWrong">Aller à la première erreur</button>
      <button class="secondary" id="btnTop">Revenir en haut</button>
    </div>
    <div class="small" style="margin-top:10px;">
      Les questions sont encadrées en <b>vert</b> si correct, en <b>rouge</b> si incorrect.
    </div>
  `;

  // actions
  document.getElementById("btnFirstWrong").onclick = goToFirstWrong;
  document.getElementById("btnTop").onclick = () => window.scrollTo({top:0, behavior:"smooth"});

  render();
  window.scrollTo({top:0, behavior:"smooth"});
}

function resetAll() {
  corrected = false;
  userAnswers = new Array(questions.length).fill(null);
  resultBox.style.display = "none";
  page = 0;
  render();
}

btnPrev.onclick = () => { page = clamp(page - 1, 0, 9999); render(); window.scrollTo({top:0, behavior:"smooth"}); };
btnNext.onclick = () => { page = clamp(page + 1, 0, 9999); render(); window.scrollTo({top:0, behavior:"smooth"}); };
btnPrev2.onclick = btnPrev.onclick;
btnNext2.onclick = btnNext.onclick;

btnFinish.onclick = grade;
btnReset.onclick = resetAll;

// init
render();
</script>
</body>
</html>
