<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="douglas" content="Douglas">
<SOUFELIZ>

<title>Desafio de Matemática</title>
<style>
  :root { --bg:#eef3fb; --card:#fff; --txt:#1c2a44; --main:#2f6fed; --ok:#1f9d55; --bad:#d64545; }
  * { box-sizing:border-box; }
  body { margin:0; font-family:Arial, Helvetica, sans-serif; background:var(--bg); color:var(--txt); display:flex; justify-content:center; padding:20px; }
  .card { background:var(--card); border-radius:16px; padding:24px; max-width:620px; width:100%; box-shadow:0 4px 18px rgba(0,0,0,.12); text-align:center; }
  h1 { font-size:1.5rem; margin:0 0 4px; }
  .sub { color:#5a6b8c; margin-bottom:16px; }
  .names { display:grid; grid-template-columns:repeat(auto-fit,minmax(110px,1fr)); gap:10px; margin:16px 0; }
  .names button { margin:0; font-size:1.1rem; }
  .info { display:flex; justify-content:space-between; font-weight:bold; margin-bottom:12px; }
  .clocks { display:flex; justify-content:space-around; align-items:center; gap:8px; margin:12px 0; }
  .clock p { margin:4px 0 0; font-size:1.4rem; font-weight:bold; }
  .clock small { color:#5a6b8c; display:block; }
  .arrow { font-size:2rem; color:var(--main); }
  input { font-size:1.4rem; width:120px; padding:8px; text-align:center; border:2px solid #b8c6e6; border-radius:10px; }
  button { font-size:1.1rem; padding:10px 20px; border:0; border-radius:10px; background:var(--main); color:#fff; cursor:pointer; margin:8px 4px; }
  button:disabled { background:#9db6ee; cursor:default; }
  button.sec { background:#6b7a99; }
  button.green { background:var(--ok); }
  button.red { background:var(--bad); font-size:.9rem; padding:6px 12px; }
  #msg { min-height:70px; font-size:1.05rem; margin-top:8px; text-align:left; }
  .ok { color:var(--ok); font-weight:bold; }
  .bad { color:var(--bad); font-weight:bold; }
  .bar { height:10px; background:#dbe4f7; border-radius:6px; overflow:hidden; margin-bottom:12px; }
  .bar div { height:100%; width:0; background:var(--main); transition:width .3s; }
  table { width:100%; border-collapse:collapse; margin-top:12px; font-size:.95rem; }
  th, td { border:1px solid #c9d5ef; padding:6px; }
  th { background:#dbe4f7; }
  .wrap { overflow-x:auto; }
  .frac { font-size:2.2rem; font-weight:bold; display:inline-block; margin:8px 0; }
  .frac span { display:block; padding:0 10px; }
  .frac span:first-child { border-bottom:3px solid #1c2a44; }
  .conta { font-size:2rem; font-weight:bold; margin:10px 0; }
  footer { margin-top:18px; font-size:.8rem; color:#6b7a99; }
</style>
</head>
<body>
<div class="card">
  <h1>🧮 Desafio de Matemática</h1>
  <div class="sub">Relógio, plano cartesiano, operações e frações</div>
 
  <!-- TELA 1: escolher o nome -->
  <div id="pick">
    <p><b>Escolha o seu nome para começar:</b></p>
    <div class="names" id="names"></div>
    <div id="rank"></div>
    <button id="dl" class="green">📥 Baixar planilha para o Excel</button>
    <button id="clear" class="red">Apagar resultados</button>
  </div>
 
  <!-- TELA 2: jogo -->
  <div id="game" style="display:none">
    <div class="info"><span id="who"></span><span id="qn"></span><span id="score"></span></div>
    <div class="bar"><div id="prog"></div></div>
    <div id="enun" style="font-size:1.15rem;margin:10px 0"></div>
    <div id="visual"></div>
    <input id="ans" type="text" inputmode="decimal" autocomplete="off" placeholder="resposta">
    <div>
      <button id="check">Conferir</button>
      <button id="next" class="sec" disabled>Próxima</button>
    </div>
    <div id="msg"></div>
  </div>
 
  <!-- TELA 3: resultado -->
  <div id="end" style="display:none">
    <h2 id="final"></h2>
    <p id="finalmsg"></p>
    <p id="sync" style="font-size:.9rem;color:#5a6b8c"></p>
    <button id="again">Voltar para os nomes</button>
    <button id="dl2" class="green">📥 Baixar planilha para o Excel</button>
  </div>
 
  <footer>Autoria: Douglas,MATEMATICA</footer>
</div>
 
<script>
const NOMES = ["Alex","Agatha","Julia","Yan","Laila","Lilia","Maria","Clara","Ana","Paula","Nicolas"];
const TOTAL = 11;
const KEY = "atividade_minutos_resultados";
// COLE AQUI o endereço do Google Apps Script (termina em /exec). Se ficar vazio, salva só neste aparelho.
const ENDPOINT = "";
let nome = "", q = 0, acertos = 0, start = 0, end = 0, answered = false, detalhe = [];
 
const $ = id => document.getElementById(id);
const pad = n => String(n).padStart(2, "0");
const fmt = m => pad(Math.floor(m / 60) % 24) + ":" + pad(m % 60);
const show = id => { ["pick","game","end"].forEach(x => $(x).style.display = x === id ? "block" : "none"); };
 
/* ---------- armazenamento ---------- */
function lerResultados() { try { return JSON.parse(localStorage.getItem(KEY)) || []; } catch (e) { return []; } }
function salvarResultado(r) { try { const l = lerResultados(); l.push(r); localStorage.setItem(KEY, JSON.stringify(l)); } catch (e) {} }
 
/* ---------- tela de nomes e ranking ---------- */
function montarNomes() {
  $("names").innerHTML = "";
  NOMES.forEach(n => {
    const b = document.createElement("button");
    b.textContent = n;
    b.onclick = () => iniciar(n);
    $("names").appendChild(b);
  });
  const l = lerResultados();
  if (!l.length) { $("rank").innerHTML = ""; return; }
  let h = '<div class="wrap"><table><tr><th>Nome</th><th>Acertos</th><th>Erros</th><th>Pontos</th><th>Data/Hora</th></tr>';
  l.slice().reverse().slice(0, 15).forEach(r => {
    h += `<tr><td>${r.nome}</td><td>${r.acertos}</td><td>${TOTAL - r.acertos}</td><td>${r.acertos * 10}</td><td>${r.data}</td></tr>`;
  });
  $("rank").innerHTML = h + "</table></div>";
}
 
/* ---------- planilha (CSV que abre direto no Excel) ---------- */
function baixarPlanilha() {
  const l = lerResultados();
  if (!l.length) { alert("Ainda não há resultados para baixar."); return; }
  const cab = ["Nome","Acertos","Erros","Pontos","Data/Hora"];
  TIPOS.forEach((t, i) => cab.push("Q" + (i + 1) + " " + t.nome));
  const linhas = [cab.join(";")];
  l.forEach(r => linhas.push([r.nome, r.acertos, TOTAL - r.acertos, r.acertos * 10, r.data, ...r.detalhe].join(";")));
  const blob = new Blob(["\uFEFF" + linhas.join("\r\n")], { type: "text/csv;charset=utf-8;" });
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "resultados_atividade_minutos.csv";
  document.body.appendChild(a); a.click(); a.remove();
}
 
/* ---------- recursos visuais ---------- */
const rnd = (a, b) => a + Math.floor(Math.random() * (b - a + 1));
 
function relogioSVG(minutes) {
  const h = Math.floor(minutes / 60) % 12, m = minutes % 60;
  const hand = (ang, len, w, col) => {
    const r = (ang - 90) * Math.PI / 180;
    return `<line x1="55" y1="55" x2="${55 + len * Math.cos(r)}" y2="${55 + len * Math.sin(r)}" stroke="${col}" stroke-width="${w}" stroke-linecap="round"/>`;
  };
  let ticks = "";
  for (let i = 0; i < 12; i++) {
    const r = i * 30 * Math.PI / 180;
    ticks += `<line x1="${55 + 44 * Math.sin(r)}" y1="${55 - 44 * Math.cos(r)}" x2="${55 + 50 * Math.sin(r)}" y2="${55 - 50 * Math.cos(r)}" stroke="#1c2a44" stroke-width="2"/>`;
  }
  return `<svg width="110" height="110" viewBox="0 0 110 110"><circle cx="55" cy="55" r="52" fill="#fff" stroke="#1c2a44" stroke-width="3"/>` + ticks +
    hand(h * 30 + m / 2, 28, 4, "#1c2a44") + hand(m * 6, 40, 2.5, "#2f6fed") + `<circle cx="55" cy="55" r="3" fill="#1c2a44"/></svg>`;
}
 
function planoSVG(pontos) {
  const ox = 34, oy = 266, p = 28;
  let s = `<svg width="290" height="290" viewBox="0 0 290 290" style="max-width:100%">`;
  for (let i = 0; i <= 8; i++) {
    s += `<line x1="${ox + i * p}" y1="${oy}" x2="${ox + i * p}" y2="${oy - 8 * p}" stroke="#c9d5ef"/>`;
    s += `<line x1="${ox}" y1="${oy - i * p}" x2="${ox + 8 * p}" y2="${oy - i * p}" stroke="#c9d5ef"/>`;
    s += `<text x="${ox + i * p}" y="${oy + 16}" font-size="12" text-anchor="middle" fill="#1c2a44">${i}</text>`;
    s += `<text x="${ox - 10}" y="${oy - i * p + 4}" font-size="12" text-anchor="end" fill="#1c2a44">${i}</text>`;
  }
  s += `<line x1="${ox}" y1="${oy}" x2="${ox + 8 * p + 8}" y2="${oy}" stroke="#1c2a44" stroke-width="2"/>`;
  s += `<line x1="${ox}" y1="${oy}" x2="${ox}" y2="${oy - 8 * p - 8}" stroke="#1c2a44" stroke-width="2"/>`;
  pontos.forEach(pt => {
    s += `<circle cx="${ox + pt.x * p}" cy="${oy - pt.y * p}" r="6" fill="#d64545"/>`;
    s += `<text x="${ox + pt.x * p + 9}" y="${oy - pt.y * p - 8}" font-size="15" font-weight="bold" fill="#1c2a44">${pt.l}</text>`;
  });
  return s + `</svg>`;
}
 
/* ---------- geradores de questões ---------- */
function genRelogio() {
  const s = (6 + rnd(0, 13)) * 60 + 5 * rnd(0, 11), d = 5 * rnd(6, 30), e = s + d;
  const prox = Math.ceil(s / 60) * 60;
  const exp = (s % 60 !== 0 && e >= prox)
    ? `de ${fmt(s)} até ${fmt(prox)} são ${prox - s} min; de ${fmt(prox)} até ${fmt(e)} são ${e - prox} min. Total: ${prox - s} + ${e - prox} = <b>${d} minutos</b>.`
    : `${fmt(e)} − ${fmt(s)} = <b>${d} minutos</b>.`;
  return { tipo: "num", resp: d, ph: "minutos", exp,
    enun: "Do horário inicial até o horário final, quantos <b>minutos</b> passaram?",
    visual: `<div class="clocks"><div class="clock"><small>Início</small>${relogioSVG(s)}<p>${fmt(s)}</p></div><div class="arrow">➜</div><div class="clock"><small>Fim</small>${relogioSVG(e)}<p>${fmt(e)}</p></div></div>` };
}
 
function genPlano1() {
  const x = rnd(1, 8), y = rnd(1, 8);
  return { tipo: "coord", resp: [x, y], ph: "x, y",
    enun: "Quais são as <b>coordenadas</b> do ponto A? <small>(escreva assim: 3, 5)</small>",
    visual: planoSVG([{ x, y, l: "A" }]),
    exp: `Do zero, ande ${x} para a direita e ${y} para cima: A = (<b>${x}, ${y}</b>).` };
}
 
function genPlano2() {
  const pts = [];
  while (pts.length < 3) {
    const x = rnd(1, 8), y = rnd(1, 8);
    if (!pts.some(p => p.x === x || p.y === y)) pts.push({ x, y, l: "ABC"[pts.length] });
  }
  const alvo = pts[rnd(0, 2)];
  return { tipo: "letra", resp: alvo.l, ph: "A, B ou C",
    enun: `Qual ponto está na posição <b>(${alvo.x}, ${alvo.y})</b>? <small>(responda A, B ou C)</small>`,
    visual: planoSVG(pts),
    exp: `Ande ${alvo.x} para a direita e ${alvo.y} para cima: é o ponto <b>${alvo.l}</b>.` };
}
 
function genConta(op, a, b) {
  const r = op === "×" ? a * b : op === "÷" ? a / b : a + b;
  return { tipo: "num", resp: r, ph: "resultado",
    enun: "Resolva a conta:",
    visual: `<div class="conta">${a.toLocaleString("pt-BR")} ${op} ${b.toLocaleString("pt-BR")} = ?</div>`,
    exp: `${a.toLocaleString("pt-BR")} ${op} ${b.toLocaleString("pt-BR")} = <b>${r.toLocaleString("pt-BR")}</b>.` };
}
const genMult1 = () => genConta("×", rnd(12, 48), rnd(3, 9));
const genMult2 = () => genConta("×", rnd(13, 35), rnd(11, 15));
const genDiv1 = () => { const b = rnd(3, 9); return genConta("÷", b * rnd(12, 60), b); };
const genDiv2 = () => { const b = rnd(4, 12); return genConta("÷", b * rnd(25, 95), b); };
const genSoma1 = () => genConta("+", rnd(120, 480), rnd(120, 480));
const genSoma2 = () => genConta("+", rnd(1000, 4999), rnd(1000, 4999));
 
function genFrac(den, dec, casas) {
  return { tipo: "num", resp: dec, ph: "decimal (ex.: 1,5)",
    enun: `Escreva a fração <b>36/${den}</b> na forma de <b>número decimal</b>:`,
    visual: `<div class="frac"><span>36</span><span>${den}</span></div>`,
    exp: `36 ÷ ${den} = <b>${String(dec).replace(".", ",")}</b>. Como o denominador é ${den}, a vírgula anda ${casas} casa(s) para a esquerda.` };
}
const genFrac10 = () => genFrac(10, 3.6, 1);
const genFrac100 = () => genFrac(100, 0.36, 2);
 
const TIPOS = [
  { nome: "Relógio", gera: genRelogio },
  { nome: "Plano cartesiano", gera: genPlano1 },
  { nome: "Plano cartesiano", gera: genPlano2 },
  { nome: "Multiplicação", gera: genMult1 },
  { nome: "Multiplicação", gera: genMult2 },
  { nome: "Divisão", gera: genDiv1 },
  { nome: "Divisão", gera: genDiv2 },
  { nome: "Adição", gera: genSoma1 },
  { nome: "Adição", gera: genSoma2 },
  { nome: "Fração 36/10", gera: genFrac10 },
  { nome: "Fração 36/100", gera: genFrac100 }
];
 
/* ---------- jogo ---------- */
let cur = null;
 
function iniciar(n) {
  nome = n; q = 0; acertos = 0; detalhe = [];
  $("who").textContent = "👤 " + nome;
  show("game");
  novaQuestao();
}
 
function novaQuestao() {
  cur = TIPOS[q].gera();
  $("qn").textContent = `Questão ${q + 1}/${TOTAL} · ${TIPOS[q].nome}`;
  $("score").textContent = `Acertos: ${acertos}`;
  $("prog").style.width = (q / TOTAL * 100) + "%";
  $("enun").innerHTML = cur.enun;
  $("visual").innerHTML = cur.visual;
  $("ans").placeholder = cur.ph; $("ans").style.width = "190px";
  $("ans").value = ""; $("ans").disabled = false; $("ans").focus();
  $("msg").innerHTML = ""; $("check").disabled = false; $("next").disabled = true;
  answered = false;
}
 
function confere(v) {
  const t = v.trim();
  if (cur.tipo === "coord") { const m = t.match(/-?\d+/g); return !!m && m.length === 2 && Number(m[0]) === cur.resp[0] && Number(m[1]) === cur.resp[1]; }
  if (cur.tipo === "letra") return t.toUpperCase() === cur.resp;
  return Number(t.replace(/\./g, "").replace(",", ".")) === cur.resp || Number(t.replace(",", ".")) === cur.resp;
}
 
function conferir() {
  if (answered) return;
  const v = $("ans").value;
  if (v.trim() === "") { $("msg").innerHTML = "Digite uma resposta primeiro."; return; }
  answered = true;
  const certo = confere(v);
  if (certo) { acertos++; $("msg").innerHTML = `<span class="ok">✔ Muito bem! Resposta correta.</span><br>${cur.exp}`; }
  else { $("msg").innerHTML = `<span class="bad">✘ Não foi dessa vez.</span><br>${cur.exp}`; }
  detalhe.push(certo ? "Certo" : "Errado");
  $("score").textContent = `Acertos: ${acertos}`;
  $("check").disabled = true; $("ans").disabled = true; $("next").disabled = false;
  $("next").textContent = q === TOTAL - 1 ? "Ver resultado" : "Próxima";
}
 
function proxima() {
  q++;
  if (q >= TOTAL) return terminar();
  novaQuestao();
}
 
function terminar() {
  const res = { nome, acertos, detalhe, data: new Date().toLocaleString("pt-BR") };
  salvarResultado(res);
  enviarNuvem(res);
  $("final").textContent = `${nome}, você acertou ${acertos} de ${TOTAL} (${acertos * 10} pontos)!`;
  $("finalmsg").textContent = acertos >= 9 ? "🏆 Excelente! Você mandou muito bem!"
    : acertos >= 6 ? "👍 Bom trabalho! Treine mais um pouco." : "💪 Continue praticando, você vai melhorar!";
  show("end");
}
 
function enviarNuvem(r) {
  if (!ENDPOINT) { $("sync").textContent = ""; return; }
  $("sync").textContent = "Enviando resultado para o professor...";
  fetch(ENDPOINT, { method: "POST", mode: "no-cors", headers: { "Content-Type": "text/plain" }, body: JSON.stringify(r) })
    .then(() => { $("sync").textContent = "✅ Resultado enviado para o professor!"; })
    .catch(() => { $("sync").textContent = "⚠️ Não foi possível enviar. Avise o professor."; });
}
 
function voltar() { montarNomes(); show("pick"); }
 
$("check").onclick = conferir;
$("next").onclick = proxima;
$("again").onclick = voltar;
$("dl").onclick = baixarPlanilha;
$("dl2").onclick = baixarPlanilha;
$("clear").onclick = () => { if (confirm("Apagar TODOS os resultados salvos?")) { try { localStorage.removeItem(KEY); } catch (e) {} montarNomes(); } };
$("ans").addEventListener("keydown", e => { if (e.key === "Enter") conferir(); });
montarNomes();
</script>
</body>
</html>
 
