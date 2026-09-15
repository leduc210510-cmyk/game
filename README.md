const HEROES = {
  ren: {
    id: "ren",
    name: "Kazuma Ren",
    role: "Kiếm sĩ Huyết Ảnh",
    img: "img/ren.jpg",
    blurb: "Nhát kiếm chảy máu. Burst xé cả hàng địch.",
    maxHp: 980, maxMp: 70, atk: 92, def: 38, spd: 76, crit: 0.18,
    color: "#ff2d55",
    skills: [
      { id: "slash", name: "Huyết Trảm", cost: 0, power: 1, target: "single", type: "phys", desc: "Chém thường." },
      { id: "crimson", name: "Crimson Moon", cost: 16, power: 1.7, target: "single", type: "phys", bleed: 0.7, desc: "Sát thương lớn + chảy máu." },
      { id: "garden", name: "Blood Garden", cost: 0, power: 2.15, target: "all", type: "burst", desc: "Bộc phát: chém cả sân." }
    ]
  },
  yuki: {
    id: "yuki",
    name: "Shiro Yuki",
    role: "Pháp sư Băng Nguyệt",
    img: "img/yuki.jpg",
    blurb: "Đóng băng địch, hồi máu, nổ diện rộng.",
    maxHp: 820, maxMp: 120, atk: 86, def: 28, spd: 68, crit: 0.12,
    color: "#8ecbff",
    skills: [
      { id: "bolt", name: "Băng Tiễn", cost: 0, power: 0.95, target: "single", type: "magic", desc: "Đòn phép cơ bản." },
      { id: "nova", name: "Frost Nova", cost: 22, power: 1.25, target: "all", type: "magic", freeze: 0.55, desc: "AOE + đóng băng." },
      { id: "veil", name: "Nguyệt Liêm", cost: 18, power: 0, target: "self", type: "heal", heal: 0.28, desc: "Hồi 28% máu." },
      { id: "domain", name: "Frozen Domain", cost: 0, power: 2.0, target: "all", type: "burst", freeze: 0.8, desc: "Bộc phát đóng băng cả sân." }
    ]
  },
  akari: {
    id: "akari",
    name: "Kuro Akari",
    role: "Sát thủ Tử Anh",
    img: "img/akari.jpg",
    blurb: "Nhanh, chí mạng, đánh 3 nhát.",
    maxHp: 740, maxMp: 80, atk: 84, def: 24, spd: 96, crit: 0.32,
    color: "#b57cff",
    skills: [
      { id: "cut", name: "Ảnh Trảm", cost: 0, power: 0.9, target: "single", type: "phys", desc: "Chém nhanh." },
      { id: "triple", name: "Tam Liên", cost: 14, power: 0.72, hits: 3, target: "single", type: "phys", desc: "Đánh 3 lần." },
      { id: "shadow", name: "Tử Ảnh", cost: 0, power: 2.4, target: "single", type: "burst", desc: "Bộc phát chí mạng đơn mục tiêu." }
    ]
  }
};

const ENEMIES = {
  soldier: {
    id: "soldier", name: "Quỷ Lính", img: "img/soldier.jpg",
    maxHp: 520, maxMp: 0, atk: 58, def: 30, spd: 50, crit: 0.08
  },
  archer: {
    id: "archer", name: "Cung Hồ Ly", img: "img/archer.jpg",
    maxHp: 430, maxMp: 0, atk: 72, def: 18, spd: 64, crit: 0.16
  },
  boss: {
    id: "boss", name: "Oni Kurogami", img: "img/boss.jpg",
    maxHp: 1680, maxMp: 0, atk: 86, def: 36, spd: 54, crit: 0.12, isBoss: true
  }
};

const WAVES = [
  {
    tag: "Trận 1 / 3",
    storyTag: "HỒI I",
    story: "Trăng huyết treo trên đền Sakura. Cổng địa ngục vừa nứt. Hai quỷ lính chặn lối — nếu không chém chúng, cả ngôi làng phía sau sẽ thành tro.",
    enemies: ["soldier", "soldier"]
  },
  {
    tag: "Trận 2 / 3",
    storyTag: "HỒI II",
    story: "Lính gục. Từ sương đen bước ra cung thủ đeo mặt nạ cáo. Mũi tên linh hồn rít lên giữa cánh hoa anh đào.",
    enemies: ["soldier", "archer"]
  },
  {
    tag: "BOSS",
    storyTag: "HỒI III",
    story: "Đất nứt. Oni Vương Kurogami đội sừng bước ra, chùy gai kéo lửa. Hắn cười: «Linh kiếm của ngươi cũng chỉ là sắt.»",
    enemies: ["boss"]
  }
];

const state = {
  heroId: null,
  player: null,
  enemies: [],
  wave: 0,
  target: 0,
  busy: false,
  potions: 2,
  showSkills: false
};

const $ = (id) => document.getElementById(id);

function show(id) {
  document.querySelectorAll(".screen").forEach((s) => s.classList.remove("active"));
  $(id).classList.add("active");
}

function rand(a, b) { return a + Math.random() * (b - a); }
function clamp(n, a, b) { return Math.max(a, Math.min(b, n)); }

function spawnPetals() {
  const box = $("petals");
  setInterval(() => {
    const p = document.createElement("div");
    p.className = "petal";
    p.style.left = Math.random() * 100 + "%";
    p.style.animationDuration = 6 + Math.random() * 6 + "s";
    p.style.opacity = 0.25 + Math.random() * 0.4;
    p.style.transform = `scale(${0.6 + Math.random()})`;
    box.appendChild(p);
    setTimeout(() => p.remove(), 12000);
  }, 420);
}

function beep(freq, dur, type = "square", vol = 0.03) {
  try {
    const ctx = beep.ctx || (beep.ctx = new (window.AudioContext || window.webkitAudioContext)());
    const o = ctx.createOscillator();
    const g = ctx.createGain();
    o.type = type; o.frequency.value = freq;
    g.gain.value = vol;
    o.connect(g); g.connect(ctx.destination);
    o.start();
    g.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + dur);
    o.stop(ctx.currentTime + dur);
  } catch (e) {}
}

function makeUnit(tpl) {
  return {
    ...tpl,
    hp: tpl.maxHp,
    mp: tpl.maxMp,
    burst: 0,
    freeze: 0,
    bleed: 0,
    dead: false
  };
}

function renderSelect() {
  const list = $("char-list");
  list.innerHTML = Object.values(HEROES).map((h) => `
    <button class="char-pick ${state.heroId === h.id ? "selected" : ""}" data-id="${h.id}">
      <img src="${h.img}" alt="${h.name}" />
      <div class="char-meta">
        <h3>${h.name}</h3>
        <div class="role">${h.role}</div>
        <p>${h.blurb}</p>
        <div class="stats">HP ${h.maxHp} · ATK ${h.atk} · SPD ${h.spd}</div>
      </div>
    </button>
  `).join("");
  list.querySelectorAll(".char-pick").forEach((btn) => {
    btn.onclick = () => {
      state.heroId = btn.dataset.id;
      $("btn-confirm").disabled = false;
      beep(520, 0.08);
      renderSelect();
    };
  });
}

function startRun() {
  const hero = HEROES[state.heroId];
  state.player = makeUnit(hero);
  state.wave = 0;
  state.potions = 2;
  openStory();
}

function openStory() {
  const w = WAVES[state.wave];
  $("story-tag").textContent = w.storyTag;
  $("story-text").textContent = w.story;
  show("screen-story");
}

function startWave() {
  const w = WAVES[state.wave];
  state.enemies = w.enemies.map((id, i) => {
    const u = makeUnit(ENEMIES[id]);
    u.uid = id + "-" + i;
    return u;
  });
  state.target = 0;
  state.busy = false;
  state.showSkills = false;
  $("wave-tag").textContent = w.tag;
  show("screen-battle");
  log("Trận chiến bắt đầu.");
  renderBattle();
}

function livingEnemies() {
  return state.enemies.filter((e) => !e.dead);
}

function hpPct(u) { return clamp((u.hp / u.maxHp) * 100, 0, 100); }
function mpPct(u) { return u.maxMp ? clamp((u.mp / u.maxMp) * 100, 0, 100) : 0; }

function renderBattle() {
  const p = state.player;
  $("burst-fill").style.width = clamp(p.burst, 0, 100) + "%";

  $("enemies").innerHTML = state.enemies.map((e, i) => `
    <div class="enemy ${e.dead ? "dead" : ""} ${state.target === i ? "selected" : ""}" data-i="${i}">
      <div class="status">${e.freeze > 0 ? "❄️" : ""}${e.bleed > 0 ? "🩸" : ""}</div>
      <img src="${e.img}" alt="${e.name}" />
      <div class="name">${e.name}</div>
      <div class="bar hp"><i class="${hpPct(e) < 30 ? "low" : ""}" style="width:${hpPct(e)}%"></i></div>
    </div>
  `).join("");

  $("enemies").querySelectorAll(".enemy").forEach((el) => {
    el.onclick = () => {
      const i = +el.dataset.i;
      if (state.enemies[i].dead || state.busy) return;
      state.target = i;
      renderBattle();
    };
  });

  $("player-card").innerHTML = `
    <img src="${p.img}" alt="${p.name}" />
    <div>
      <div class="name">${p.name}</div>
      <div class="nums">HP ${Math.ceil(p.hp)}/${p.maxHp} · MP ${Math.ceil(p.mp)}/${p.maxMp} · Bình ${state.potions}</div>
      <div class="bar hp"><i class="${hpPct(p) < 30 ? "low" : ""}" style="width:${hpPct(p)}%"></i></div>
      <div class="bar mp"><i style="width:${mpPct(p)}%"></i></div>
    </div>
  `;

  const box = $("actions");
  if (state.showSkills) {
    const skills = p.skills.filter((s) => s.type !== "burst");
    box.className = "actions skill-menu";
    box.innerHTML = skills.map((s) =>
      `<button class="btn skill" data-sid="${s.id}">${s.name} <small>(${s.cost} MP)</small><br><span style="font-weight:400;color:#c9b3bd;font-size:11px">${s.desc}</span></button>`
    ).join("") + `<button class="btn ghost" id="btn-cancel">Huỷ</button>`;
    box.querySelectorAll("[data-sid]").forEach((b) => b.onclick = () => useSkill(b.dataset.sid));
    $("btn-cancel").onclick = () => { state.showSkills = false; renderBattle(); };
    return;
  }

  box.className = "actions";
  box.innerHTML = `
    <button class="btn primary" id="btn-atk" ${state.busy ? "disabled" : ""}>Tấn công</button>
    <button class="btn skill" id="btn-sk" ${state.busy ? "disabled" : ""}>Kỹ năng</button>
    <button class="btn skill" id="btn-burst" ${state.busy || p.burst < 100 ? "disabled" : ""}>Bộc phát</button>
    <button class="btn ghost" id="btn-pot" ${state.busy || state.potions <= 0 ? "disabled" : ""}>Bình máu</button>
  `;
  $("btn-atk").onclick = () => useSkill(p.skills[0].id);
  $("btn-sk").onclick = () => { state.showSkills = true; renderBattle(); };
  $("btn-burst").onclick = () => {
    const ult = p.skills.find((s) => s.type === "burst");
    useSkill(ult.id);
  };
  $("btn-pot").onclick = usePotion;
}

function log(msg) { $("battle-log").textContent = msg; }

function floatText(text, kind) {
  const el = document.createElement("div");
  el.className = "float-num " + kind;
  el.textContent = text;
  $("fx-layer").appendChild(el);
  setTimeout(() => el.remove(), 900);
}

function shake() {
  $("screen-battle").classList.remove("shake");
  void $("screen-battle").offsetWidth;
  $("screen-battle").classList.add("shake");
}

function flash() {
  const f = document.createElement("div");
  f.className = "flash";
  $("screen-battle").appendChild(f);
  setTimeout(() => f.remove(), 250);
}

function dmgOf(atk, def, power) {
  const raw = atk * power * rand(0.9, 1.12);
  const mit = def / (def + 90);
  return Math.max(8, Math.round(raw * (1 - mit)));
}

function ensureTarget() {
  if (!state.enemies[state.target] || state.enemies[state.target].dead) {
    const i = state.enemies.findIndex((e) => !e.dead);
    state.target = i < 0 ? 0 : i;
  }
}

function applyDamage(unit, amount, crit) {
  unit.hp = clamp(unit.hp - amount, 0, unit.maxHp);
  if (unit.hp <= 0) {
    unit.hp = 0;
    unit.dead = true;
  }
  floatText((crit ? "CRIT " : "") + amount, crit ? "crit" : "dmg");
}

function applyHeal(unit, amount) {
  unit.hp = clamp(unit.hp + amount, 0, unit.maxHp);
  floatText("+" + amount, "heal");
}

async function useSkill(sid) {
  if (state.busy || state.player.dead) return;
  const p = state.player;
  const skill = p.skills.find((s) => s.id === sid);
  if (!skill) return;
  if (skill.type !== "burst" && p.mp < skill.cost) {
    log("Không đủ MP.");
    beep(180, 0.1);
    return;
  }
  if (skill.type === "burst" && p.burst < 100) return;

  state.busy = true;
  state.showSkills = false;
  ensureTarget();
  renderBattle();

  if (skill.type !== "burst") p.mp -= skill.cost;
  if (skill.type === "burst") p.burst = 0;

  if (skill.type === "heal") {
    const heal = Math.round(p.maxHp * skill.heal);
    applyHeal(p, heal);
    log(`${p.name} dùng ${skill.name}, hồi ${heal} HP.`);
    beep(640, 0.12, "sine", 0.04);
    await wait(500);
    return afterPlayer();
  }

  const targets = skill.target === "all" ? livingEnemies() : [state.enemies[state.target]];
  const hits = skill.hits || 1;
  let total = 0;

  for (const t of targets) {
    for (let h = 0; h < hits; h++) {
      if (t.dead) break;
      const crit = Math.random() < p.crit || skill.type === "burst" && Math.random() < 0.35;
      let dmg = dmgOf(p.atk, t.def, skill.power);
      if (crit) dmg = Math.round(dmg * 1.55);
      applyDamage(t, dmg, crit);
      total += dmg;
      const card = document.querySelector(`.enemy[data-i="${state.enemies.indexOf(t)}"]`);
      if (card) {
        card.classList.remove("hit");
        void card.offsetWidth;
        card.classList.add("hit");
      }
      beep(crit ? 880 : 320, 0.07);
      await wait(hits > 1 ? 180 : 80);
    }
    if (skill.bleed && !t.dead && Math.random() < skill.bleed) t.bleed = 3;
    if (skill.freeze && !t.dead && Math.random() < skill.freeze) t.freeze = 1;
  }

  p.burst = clamp(p.burst + (skill.type === "burst" ? 0 : 28), 0, 100);
  log(`${p.name} dùng ${skill.name} · ${total} sát thương.`);
  shake();
  if (skill.type === "burst") flash();
  await wait(420);
  afterPlayer();
}

async function usePotion() {
  if (state.busy || state.potions <= 0) return;
  state.busy = true;
  state.potions -= 1;
  const heal = Math.round(state.player.maxHp * 0.4);
  applyHeal(state.player, heal);
  log(`Uống bình máu, hồi ${heal} HP.`);
  beep(700, 0.12, "sine", 0.04);
  renderBattle();
  await wait(400);
  afterPlayer();
}

async function afterPlayer() {
  tickStatus(state.player);
  if (livingEnemies().length === 0) {
    renderBattle();
    await wait(500);
    return nextWaveOrWin();
  }
  await enemyTurns();
  if (state.player.dead) {
    renderBattle();
    return finish(false);
  }
  state.player.mp = clamp(state.player.mp + 6, 0, state.player.maxMp);
  state.busy = false;
  renderBattle();
}

function tickStatus(u) {
  if (u.bleed > 0 && !u.dead) {
    const d = Math.round(u.maxHp * 0.07);
    applyDamage(u, d, false);
    u.bleed -= 1;
    log(`${u.name} chảy máu -${d}`);
  }
  if (u.freeze > 0) u.freeze -= 1;
}

async function enemyTurns() {
  for (const e of state.enemies) {
    if (e.dead || state.player.dead) continue;
    tickStatus(e);
    if (e.dead) continue;
    if (e.freeze > 0) {
      log(`${e.name} bị đóng băng.`);
      await wait(380);
      continue;
    }
    let power = 1;
    let label = "tấn công";
    if (e.isBoss && e.hp < e.maxHp * 0.45) {
      power = 1.45;
      label = "Nộ Chùy";
    } else if (e.id === "archer") {
      power = 1.15;
      label = "Linh Tiễn";
    }
    const crit = Math.random() < e.crit;
    let dmg = dmgOf(e.atk, state.player.def, power);
    if (crit) dmg = Math.round(dmg * 1.5);
    applyDamage(state.player, dmg, crit);
    log(`${e.name} ${label} · ${dmg} sát thương.`);
    shake();
    beep(200, 0.1);
    renderBattle();
    await wait(520);
  }
}

function nextWaveOrWin() {
  if (state.wave >= WAVES.length - 1) return finish(true);
  state.wave += 1;
  const bonus = Math.round(state.player.maxHp * 0.18);
  state.player.hp = clamp(state.player.hp + bonus, 0, state.player.maxHp);
  state.player.mp = clamp(state.player.mp + 25, 0, state.player.maxMp);
  state.player.burst = clamp(state.player.burst + 20, 0, 100);
  openStory();
}

function finish(win) {
  state.busy = true;
  if (win) {
    $("result-tag").textContent = "CHIẾN THẮNG";
    $("result-title").textContent = "Trăng huyết tan";
    $("result-text").textContent = `${state.player.name} hạ Kurogami. Linh kiếm còn run trong tay — đây mới chỉ là cánh cổng đầu tiên.`;
  } else {
    $("result-tag").textContent = "THẤT BẠI";
    $("result-title").textContent = "Kiếm gãy";
    $("result-text").textContent = "Hoa anh đào rơi trên xác. Thử nhân vật khác, giữ Burst cho boss.";
  }
  show("screen-result");
}

function wait(ms) { return new Promise((r) => setTimeout(r, ms)); }

function bind() {
  $("btn-start").onclick = () => { beep(440, 0.08); renderSelect(); show("screen-select"); };
  $("btn-how").onclick = () => show("screen-how");
  $("btn-how-back").onclick = () => show("screen-title");
  $("btn-confirm").onclick = startRun;
  $("btn-story").onclick = startWave;
  $("btn-again").onclick = () => {
    state.heroId = null;
    $("btn-confirm").disabled = true;
    renderSelect();
    show("screen-title");
  };
}

bind();
spawnPetals();
renderSelect();
