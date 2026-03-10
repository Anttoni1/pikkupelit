# Retro Arcade — Design & Development Guide

Tämä ohje määrittelee yhtenäisen tyylin ja tekniset käytännöt retro-mobiilipeleille. Jokainen peli on yksittäinen HTML-tiedosto (vanilla HTML/CSS/JS, ei riippuvuuksia). Pelit jaetaan yhteisen valikon kautta.

---

## 1. Visuaalinen identiteetti

### CRT-retro-teema
Kaikki pelit ja valikko noudattavat yhtenäistä CRT-fosfori-estetiikkaa:

- **Tausta:** `radial-gradient(ellipse at center, #0d0d0d 0%, #000 80%)`
- **Scanline-efekti:** `::before` pseudo-elementti — `repeating-linear-gradient` 2px välein, `rgba(0,0,0,0.08)`
- **Vinjetti:** `::after` pseudo-elementti — `radial-gradient(ellipse at center, transparent 55%, rgba(0,0,0,0.65) 100%)`
- Molemmat `position: fixed; inset: 0; pointer-events: none; z-index: 1000+`

### Väripaletti (CSS-muuttujat)
```css
:root {
  --g: #33ff33;          /* Pääväri — vihreä fosfori */
  --gd: #1a8c1a;         /* Himmennetty vihreä — reunat, inaktiivinen */
  --bg: #070707;         /* Pelialueen tausta */
  --glow: rgba(51,255,51,0.12);  /* Hehku-efekti */
  --amber: #ffaa00;      /* Korostusväri — pisteet, numerot, aktiiviset valinnat */
}
```

Värit pysyvät samoina kaikissa peleissä. Pelialueella voi käyttää vihreän sävyjä vaihteluun:
`#33ff33, #33dd33, #00ffaa, #66ff66, #00cc66, #22ff88, #44ffcc`

### Typografia
- **Fontti:** `'Press Start 2P', monospace` (Google Fonts)
- **Otsikot:** 14–22px, `letter-spacing: 4–5px`, titleGlow-animaatio
- **Pisteet/arvot:** `--amber` värillä, `text-shadow: 0 0 4–5px var(--amber)`
- **Labelit:** 5–7px, `opacity: 0.6`, `letter-spacing: 1–2px`
- **Hintit:** 5px, `opacity: 0.3`

### Animaatiot
```css
@keyframes titleGlow {
  0%, 100% { text-shadow: 0 0 2px #fff, 0 0 8px var(--g), 0 0 20px rgba(51,255,51,0.25) }
  50% { text-shadow: 0 0 4px #fff, 0 0 12px var(--g), 0 0 30px rgba(51,255,51,0.35) }
}
@keyframes blink {
  0%, 100% { opacity: 1 } 50% { opacity: 0 }
}
```
- Pelin nimi: `titleGlow 3s ease-in-out infinite`
- "Kosketa aloittaaksesi" -tyyppiset hintit: `blink 1.2s step-end infinite`
- **Overlay h1:** `color: #d0ffd0` — hieman vaaleampi kuin perusvihreä, yhdessä valkoisen sisäisen glown kanssa erottaa kirjaimet taustahehkusta

**Huom:** Pelkkä `var(--g)` text-shadowna saman värisen tekstin päällä tekee tekstistä epäselvän hehkuvan möykyn. Valkoinen lähiglow (`0 0 2px #fff`) terävöittää kirjainmuodot, ja ulommat tasot käyttävät läpinäkyvää vihreää.

### Elementtien tyylit
- **Canvas/pelialue:** `border: 2px solid var(--gd)`, `box-shadow: 0 0 15px var(--glow), inset 0 0 25px rgba(0,0,0,0.4)`, `image-rendering: pixelated`
- **Pienet paneelit (preview yms.):** `border: 1px solid var(--gd)`, `box-shadow: 0 0 4px var(--glow)`
- **Overlay-ruudut:** `background: rgba(0,0,0,0.88)`, keskitetty flexbox, `z-index: 500`
- **Pause-nappi:** Topbarin keskellä (vasemman ja oikean välissä), teksti aina `PAUSE` (ei `| |` tai muuta), `color: var(--gd)`, `border: 1px solid var(--gd)`, ei opacity-himmennetty

### Piirto-tyyli (canvas-blokit)
Kun peli piirtää ruudukkoblokkeja canvasille:
```
- Täytetty blokki: 1px padding joka reunalla
- Vaaleat highlight-viivat vasempaan yläkulmaan (2px leveitä)
- Tummat varjo-viivat oikeaan alakulmaan
- Väri pelin kontekstin mukaan, vihreän sävyistä
```

---

## 2. Layout & mobiilioptimointi

### iPhone-ensisijainen suunnittelu
Jokainen peli suunnitellaan täyttämään koko näyttö ilman scrollausta.

**Pakolliset meta-tagit:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
```

**Pakolliset CSS-säännöt:**
```css
html, body {
  height: 100%; height: 100dvh; overflow: hidden;
  touch-action: none;
  -webkit-user-select: none; user-select: none;
  -webkit-touch-callout: none;
  -webkit-tap-highlight-color: transparent;
}
body {
  padding: env(safe-area-inset-top) env(safe-area-inset-right)
           env(safe-area-inset-bottom) env(safe-area-inset-left);
}
```

**Scrollauksen esto:**
```javascript
document.addEventListener('touchmove', e => e.preventDefault(), { passive: false });
```

### Sivun rakenne (pystysuunnassa)
```
┌──────────────────────┐
│  TOPBAR              │  ← flex-shrink: 0, padding: 6px 12px
│  Nimi  PAUSE  Stats  │     Vasemmalla: pelin nimi + score
│                      │     Keskellä: PAUSE-nappi
│                      │     Oikealla: pelikohtainen info (level, next jne.)
├──────────────────────┤
│                      │
│                      │
│   PELIALUE           │  ← flex: 1, täyttää kaiken jäljellä olevan tilan
│   (canvas)           │     Canvas skaalataan dynaamisesti (CSS width/height)
│                      │     Looginen resoluutio pysyy vakiona
│                      │
│                      │
├──────────────────────┤
│  TOUCH HINT          │  ← flex-shrink: 0, 5px teksti, opacity: 0.3
└──────────────────────┘
```

### Pelialueen skaalaus
Canvas piirretään aina kiinteällä loogisella resoluutiolla. CSS-koko skaalataan täyttämään tila:
```javascript
function resize() {
  const wrap = document.getElementById('boardWrap');
  const aH = wrap.clientHeight - 8;
  const aW = wrap.clientWidth - 8;
  // Sovita aspektisuhteen mukaan — pelin oma logiikka
  let h = aH, w = h / ASPECT;
  if (w > aW) { w = aW; h = w * ASPECT; }
  canvas.style.width = Math.floor(w) + 'px';
  canvas.style.height = Math.floor(h) + 'px';
}
window.addEventListener('resize', resize);
setTimeout(resize, 50);
setTimeout(resize, 200);
resize();
```

### EI nappeja
Pelejä ohjataan swipe-eleillä ja tapeilla. Alareunan nappeja EI käytetä — ne vievät arvokasta tilaa mobiilissa. Poikkeus: pieni pause-nappi.

---

## 3. Ohjaus (touch + keyboard)

### Touch-ohjaus (koko pelialue)
Ohjaus sidotaan pelialueen wrapper-elementtiin (`boardWrap`). Koko pelialue on kosketusaluetta.

**Periaatteet:**
- **Swipe vasen/oikea:** liikuta/ohjaa (toistuva — pidempi veto = enemmän siirtoja)
- **Swipe alas:** soft drop / nopea toiminto
- **Nopea swipe alas:** hard drop / vahva toiminto (tunnistetaan nopeudesta, ei pelkästä etäisyydestä)
- **Tap (lyhyt kosketus, <250ms, vähäinen liike):** käännä / ensisijainen toiminto
- Kynnysarvot: horisontaalinen ~24px per askel, vertikaalinen ~30px per askel

**Vakiokoodi:**
```javascript
const bw = document.getElementById('boardWrap');
let tx, ty, tt, moved, lastMX, totalDY;
const SH = 24, SV = 30, HDSPD = 1.5; // px/ms hard drop -nopeus

bw.addEventListener('touchstart', e => {
  if (!gameRunning || paused || e.target.id === 'pauseBtn') return;
  e.preventDefault();
  const t = e.touches[0];
  tx = t.clientX; ty = t.clientY; tt = Date.now();
  moved = false; lastMX = tx; totalDY = 0;
}, { passive: false });

bw.addEventListener('touchmove', e => {
  if (!gameRunning || paused) return;
  e.preventDefault();
  const t = e.touches[0];
  const dx = t.clientX - lastMX, dy = t.clientY - ty;
  if (Math.abs(dx) > SH) {
    const s = Math.floor(Math.abs(dx) / SH);
    for (let i = 0; i < s; i++) dx > 0 ? moveRight() : moveLeft();
    lastMX += Math.sign(dx) * s * SH;
    moved = true;
  }
  if (dy > SV) {
    const s = Math.floor(dy / SV);
    for (let i = 0; i < s; i++) softDropAction();
    ty += s * SV; totalDY += s * SV;
    moved = true;
  }
}, { passive: false });

bw.addEventListener('touchend', e => {
  if (!gameRunning || paused) return;
  e.preventDefault();
  const el = Date.now() - tt;
  const ct = e.changedTouches[0];
  const fdx = ct.clientX - tx, fdy = ct.clientY - ty + totalDY;
  if (moved && fdy > 50 && el > 0 && (fdy / el) > HDSPD) { hardDropAction(); return; }
  if (!moved && el < 250 && Math.abs(fdx) < 12 && Math.abs(ct.clientY - ty) < 12) tapAction();
}, { passive: false });
```

Pelikohtaiset funktiot (`moveLeft`, `moveRight`, `softDropAction`, `hardDropAction`, `tapAction`) vaihtelevat pelin mukaan.

### Näppäimistö (PC-tuki)
Jokaisella pelillä on myös näppäimistöohjaus. Vakio:
- **Nuolet:** liikkuminen
- **Ylänuoli / Välilyönti:** ensisijainen toiminto
- **P / Escape:** pause ja jatka

```javascript
document.addEventListener('keydown', e => {
  if (!gameRunning || paused) return;
  switch (e.key) {
    case 'ArrowLeft': e.preventDefault(); moveLeft(); break;
    case 'ArrowRight': e.preventDefault(); moveRight(); break;
    case 'ArrowDown': e.preventDefault(); softDropAction(); break;
    case 'ArrowUp': e.preventDefault(); tapAction(); break;
    case ' ': e.preventDefault(); hardDropAction(); break;
    case 'p': case 'P': e.preventDefault(); togglePause(); break;
  }
});
```

---

## 4. Pelirakenne (vakioarkkitehtuuri)

Jokainen peli noudattaa samaa perusrakennetta:

### HTML-runko
```
overlay#startScreen   — Pelin nimi + ohjeet + "TAP TO START"
overlay#gameOver      — "GAME OVER" + lopputulokset + "TAP TO RETRY"
div.game-container    — Topbar + boardWrap + touchHint
div#pauseOverlay      — "PAUSE" teksti
```

### JavaScript-rakenne
```
1. Vakiot (ruudukon koko, värit, muodot)
2. Pelitilamuuttujat
3. Ääni (beep-funktio Web Audio API:lla)
4. Pelilogiikka (grid, collision, spawn, clear jne.)
5. Piirtofunktiot (draw, drawPreview)
6. UI-päivitykset (updateUI, showGameOver)
7. Input-funktiot (move, rotate, drop jne.)
8. Näppäimistökuuntelija
9. Touch-kuuntelijat (boardWrap)
10. Pause-toiminnallisuus
11. Game loop (requestAnimationFrame)
12. startGame()
13. Overlay-kuuntelijat (start/retry)
14. resize()
```

### Ääniefektit
Kaikki äänet tuotetaan Web Audio API:lla — square wave -tyyppiset retro-biipit:
```javascript
let audioCtx;
function beep(freq, duration, volume = 0.08) {
  try {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    if (audioCtx.state === 'suspended') audioCtx.resume();
    const o = audioCtx.createOscillator();
    const g = audioCtx.createGain();
    o.type = 'square';
    o.frequency.value = freq;
    g.gain.value = volume;
    o.connect(g); g.connect(audioCtx.destination);
    o.start();
    g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
    o.stop(audioCtx.currentTime + duration);
  } catch(e) {}
}
```
**Huom:** `audioCtx.resume()` on pakollinen — iOS Safari ja monet mobiiliselaimet luovat AudioContextin `suspended`-tilassa. Ilman resumea ääntä ei kuulu.
Tyypilliset äänet: liike 220Hz/50ms, käännös 440Hz/60ms, pudotus 160Hz/100ms, rivin poisto 660Hz/150ms, game over 100Hz/400ms.

### High score
Tallennetaan `localStorage`:een pelikohtaisella avaimella:
```javascript
try { localStorage.setItem('PELINIMI_hi', hiScore); } catch(e) {}
```

### Game loop
```javascript
function gameLoop(time) {
  if (!gameRunning) { draw(); return; }
  if (paused) return;
  const dt = time - (lastTime || time);
  lastTime = time;
  // Pelilogiikka dt:n perusteella
  draw();
  requestAnimationFrame(gameLoop);
}
```

---

## 5. Overlay-ruudut

### Aloitusruutu
- Pelin nimi isolla (`<h1>`, titleGlow)
- Lyhyet ohjeet (swipe/tap -kuvaukset pelin kontekstissa)
- "PRESS START" vilkkuvalla animaatiolla (tai "TAP TO START")
- Kaikki tekstit englanniksi
- **Käynnistys:** click, touchend JA `Enter`/`Space`-näppäin

### Game over -ruutu
- "GAME OVER" amber-värillä
- Lopputulos (pisteet, taso, erityistiedot pelistä riippuen)
- "PRESS RETRY" (tai "TAP TO RETRY")
- **Uudelleen:** click, touchend JA `Enter`/`Space`-näppäin

### Pause
- Erillinen overlay `z-index: 400`
- "PAUSE" vilkkuvalla animaatiolla
- Overlay sisältää "RESUME" ja "EXIT TO MENU" -napit
- `P`/`Escape` pausettaa ja jatkaa peliä (paitsi EXIT TO MENU -napista)
- Näppäimistönavigointi: ↑↓ vaihtaa RESUME / EXIT TO MENU välillä, Enter vahvistaa

**Pause-napin kuuntelijat — TÄRKEÄÄ:**
PauseBtn tarvitsee sekä `click` että `touchend` — boardWrap kutsuu `e.preventDefault()` touchend:ssä, mikä estää `click`-tapahtuman syntymisen mobiilissa:
```javascript
document.getElementById('pauseBtn').addEventListener('click', e => { e.stopPropagation(); togglePause(); });
document.getElementById('pauseBtn').addEventListener('touchend', e => { e.stopPropagation(); e.preventDefault(); togglePause(); });
pauseOv.addEventListener('click', e => { if (!['resumeBtn','menuBtn','confirmYes','confirmNo'].includes(e.target.id) && !e.target.closest('#confirmBox')) togglePause(); });
pauseOv.addEventListener('touchend', e => { if (!['resumeBtn','menuBtn','confirmYes','confirmNo'].includes(e.target.id) && !e.target.closest('#confirmBox')) { e.preventDefault(); togglePause(); } });
['click', 'touchend'].forEach(ev => document.getElementById('resumeBtn').addEventListener(ev, e => { e.stopPropagation(); e.preventDefault(); togglePause(); }));
['click', 'touchend'].forEach(ev => document.getElementById('menuBtn').addEventListener(ev, e => { e.stopPropagation(); e.preventDefault(); showConfirm(); }));
```

### Overlay-kuuntelijoiden vakiokoodi

Kaikki overlayt tukevat sekä hiirtä, touchia että näppäimistöä — peli on pelattavissa kokonaan ilman hiirtä:

```javascript
// Click & touch
['click', 'touchend'].forEach(ev => {
  document.getElementById('startScreen').addEventListener(ev, e => {
    e.preventDefault();
    document.getElementById('startScreen').classList.add('hidden');
    startGame();
  });
  document.getElementById('gameOver').addEventListener(ev, e => {
    e.preventDefault();
    document.getElementById('gameOver').classList.add('hidden');
    startGame();
  });
});

// Näppäimistö — Enter tai Space käynnistää/yrittää uudelleen
document.addEventListener('keydown', e => {
  if (e.key === 'Enter' || e.key === ' ') {
    const start = document.getElementById('startScreen');
    const over = document.getElementById('gameOver');
    if (!start.classList.contains('hidden')) {
      e.preventDefault(); start.classList.add('hidden'); startGame();
    } else if (!over.classList.contains('hidden')) {
      e.preventDefault(); over.classList.add('hidden'); startGame();
    }
  }
});
```

**Huom:** Jos pelissä on muitakin overlay-ruutuja (esim. "ALL CLEAR"), lisää ne samaan keydown-kuuntelijaan.

---

## 6. Tiedostorakenne

```
games/
├── index.html          ← Päävalikko
├── tetris.html         ← Tetris
├── snake.html          ← Snake
├── breakout.html       ← Breakout
├── pong.html           ← Pong
└── ...                 ← Lisää pelejä
```

Jokainen peli on itsenäinen HTML-tiedosto — ei jaettuja JS/CSS-tiedostoja. Tämä pitää asiat yksinkertaisina ja jokaisen pelin helposti testattavana ja jaettavana erikseen.

---

## 7. Päävalikko (index.html)

Valikko noudattaa samaa CRT-tyyliä. Rakenne:

```
┌──────────────────────┐
│     RETRO ARCADE     │  ← titleGlow
├──────────────────────┤
│                      │
│  ► TETRIS      1250  │  ← Pelilistaus, hi-score oikealla
│  ► SNAKE        890  │     Valittu/hover: --amber korostus
│  ► BREAKOUT     640  │     Tap/click avaa pelin
│  ► PONG         ---  │     "---" jos ei hi-scorea
│                      │
└──────────────────────┘
│     SWIPE ↑↓ SELECT
      TAP TO PLAY
```

- Pelilista scrollattavissa jos useita pelejä
- Hi-scoret luetaan `localStorage`:sta (avaimet: `tetris_hi`, `snake_hi` jne.)
- Navigointi: `window.location.href = 'tetris.html'`
- Jokaisessa pelissä "takaisin valikkoon" -tapa (esim. pelin otsikkoa painamalla tai pieni ← ikoni)

---

## 8. Kieli

- Kaikki pelien tekstit **englanniksi** — ohjeet, hintit, overlayt, pelitermit
- Esimerkkejä: "TAP TO START", "TAP TO RETRY", "GAME OVER", "SWIPE ←→ MOVE", "TAP ROTATE", "FAST↓ DROP", "SCORE", "LEVEL", "LINES", "HI", "NEXT", "PAUSE"
- Englanti on luonteva kieli retro-arcade-kontekstissa ja tekee peleistä universaalisti pelattavia

---

## 9. Automaattitestaus (Playwright)

Jokainen peli testataan automaattisesti headless-selaimella ennen julkaisua. Testit ajetaan Playwrightilla (Chromium) ja ne simuloivat oikeaa pelaamista — näppäimistöä, touch-eleitä ja pelitilan manipulointia.

### Ympäristö ja setup

```javascript
const { chromium } = require('playwright');
const path = require('path');

const browser = await chromium.launch();
const page = await browser.newPage({
  viewport: { width: 390, height: 844 },  // iPhone 14
  hasTouch: true                           // Pakollinen touch-testeille
});
await page.goto('file://' + path.resolve('peli.html'));
await page.waitForTimeout(500);
```

### Testikategoriat

Jokaisen pelin testiskripti kattaa nämä kategoriat:

#### 1. UI-tila ja overlay-ruudut
- Aloitusruutu näkyy ladattaessa
- Klikkaus/tap piilottaa aloitusruudun ja käynnistää pelin
- Game over -ruutu näkyy pelin päättyessä
- Retry käynnistää pelin uudelleen

```javascript
// Esimerkki
const startVisible = await page.isVisible('#startScreen');
await page.click('#startScreen');
await page.waitForTimeout(300);
const startHidden = await page.isHidden('#startScreen');
```

#### 2. Alkutila
- `gameRunning === true`, `paused === false`
- Pisteet 0, oikea elämienmäärä, taso 1
- Pelikohtaiset alkuarvot (esim. invaderien määrä, tiilien määrä)

```javascript
const state = await page.evaluate(() => ({
  gameRunning, paused, scoreVal, livesVal
}));
```

#### 3. Näppäimistöohjaus
- Nuolinäppäimet liikuttavat pelaajaa/palikkaa
- Ensisijainen toiminto (ylänuoli/välilyönti) toimii
- Yksittäiset painallukset rekisteröityvät (ei vain keysDown-looppi)

```javascript
await page.evaluate(() => { playerX = 150; });
for (let i = 0; i < 10; i++) await page.keyboard.press('ArrowLeft');
const after = await page.evaluate(() => playerX);
// assert after < 150
```

#### 4. Pelaajan rajat
- Pelaaja/maila ei mene pelialueen ulkopuolelle
- Clamp toimii myös suoralla state-manipulaatiolla (ei vain input-funktioissa)

```javascript
await page.evaluate(() => { playerX = -10; });
await page.evaluate(() => new Promise(r => requestAnimationFrame(() => setTimeout(r, 30))));
const clamped = await page.evaluate(() => playerX);
// assert clamped >= 0
```

#### 5. Pelimekaniikka
Pelikohtaiset testit jotka varmistavat ydinlogiikan:
- **Breakout:** pallo kimpoaa mailasta/tiilistä, tiilet tuhoutuvat, taso vaihtuu
- **Tetris:** palikan pudotus, rivien poisto, collision
- **Invaders:** ammus osuu invaderiin, pisteytys, invaderit liikkuvat, wave clear

```javascript
// Suora osuma — aseta ammus/pallo lähelle kohdetta
await page.evaluate(() => {
  playerBullet = { x: targetX, y: targetY + 5 };
});
for (let i = 0; i < 30; i++) {
  const s = await page.evaluate(() => ({ scoreVal }));
  if (s.scoreVal > 0) break;
  await page.evaluate(() => new Promise(r => requestAnimationFrame(r)));
}
```

#### 6. Pause
- P-näppäin aktivoi paussin
- Overlay näkyy
- Toinen P jatkaa peliä

#### 7. Elämät ja game over
- Elämän menetys laskee laskuria
- Nollassa game over -ruutu näkyy ja `gameRunning === false`

```javascript
// Pakota kuolema
await page.evaluate(() => {
  livesVal = 1;
  // aseta kuoleman aiheuttava tila
});
// Aja frameja
// Tarkista: gameRunning === false, gameOver-overlay näkyy
```

#### 8. Canvas-resize
- Eri viewport-koot tuottavat eri canvas CSS-koot
- Testataan vähintään kahdella koolla (esim. iPhone SE + iPhone 14 Pro Max)

```javascript
await page.setViewportSize({ width: 320, height: 568 });
await page.waitForTimeout(300);
const size1 = await page.evaluate(() => canvas.style.width);
await page.setViewportSize({ width: 428, height: 926 });
await page.waitForTimeout(300);
const size2 = await page.evaluate(() => canvas.style.width);
// assert size1 !== size2
```

#### 9. High score
- `localStorage`-avain tallennetaan kun saadaan pisteitä
- Avain noudattaa `pelinimi_hi` -muotoa

#### 10. Touch-simulaatio
- Tap pelialueella suorittaa ensisijaisen toiminnon
- Konteksti: `hasTouch: true` Playwright-asetuksissa

```javascript
const box = await page.locator('#boardWrap').boundingBox();
await page.touchscreen.tap(box.x + box.width/2, box.y + box.height/2);
```

#### 11. Pitkä pelitesti (stressitesti)
- AI-ohjattu pelaaminen 300–500 framea
- Tarkistetaan ettei tule NaN-arvoja, jumittuneita tiloja tai kaatumisia
- Yksinkertainen AI seuraa kohdetta ja suorittaa toimintoja

```javascript
for (let i = 0; i < 500; i++) {
  const s = await page.evaluate(() => ({
    gameRunning, playerX, scoreVal, /* pelikohtaiset */ 
  }));
  if (!s.gameRunning) break;
  
  // NaN-tarkistus
  if (isNaN(s.playerX)) { anomalies.push('NaN'); break; }
  
  // Yksinkertainen AI
  if (targetX < s.playerX) await page.keyboard.press('ArrowLeft');
  else if (targetX > s.playerX) await page.keyboard.press('ArrowRight');
  else await page.keyboard.press('ArrowUp'); // toiminto
  
  await page.evaluate(() => new Promise(r => requestAnimationFrame(r)));
}
```

### Tulostusmuoto

Testiskripti tulostaa selkeän listan tuloksista:

```
=== PELINIMI AUTOMATED TEST ===

[1] Start screen
  ✓ visible
[2] Tap to start
  ✓ hidden after tap
[3] Initial state
  ✓ running
  ✓ 3 lives
  ✗ score 0 FAIL
...

=== RESULTS: 30 passed, 2 failed ===
```

Helper-funktio:
```javascript
let pass = 0, fail = 0;
function check(name, ok) {
  if (ok) { pass++; console.log(`  ✓ ${name}`); }
  else { fail++; console.log(`  ✗ ${name} FAIL`); }
}
```

### Testien ajaminen

```bash
# Playwright-asennus (kerran)
npx playwright install chromium

# Testin ajo
node test-pelinimi.js
```

### Huomioita

- **Frame-by-frame ajo:** Käytä `await page.evaluate(() => new Promise(r => requestAnimationFrame(r)))` yhden framen ajamiseen — tarkempaa kuin `waitForTimeout`
- **State-manipulaatio:** `page.evaluate()` pääsee suoraan pelin muuttujiin — hyödynnä tätä vaikeiden tilanteiden testaamiseen (aseta pallo lähelle kohdetta, pakota kuolema jne.)
- **Ajoitusongelmat:** Playwright-framien välissä on overhead (~5–20ms), joten liikkuvien kohteiden osumatesteissä aseta ammus/pallo lähelle kohdetta sen sijaan että odotat luonnollista osumaa
- **Touch vaatii `hasTouch: true`:** Muuten `page.touchscreen.tap()` heittää virheen

---

## 10. Tarkistuslista uudelle pelille

Kun luot uuden pelin, varmista:

- [ ] Yksi HTML-tiedosto, ei ulkoisia riippuvuuksia (paitsi Google Fonts)
- [ ] CRT-efektit (scanlines, vinjetti) `body.crt`
- [ ] CSS-muuttujat — käytä samaa palettia
- [ ] Press Start 2P -fontti
- [ ] iPhone meta-tagit ja safe area -tuet
- [ ] Koko näyttö, ei scrollausta, `100dvh`
- [ ] Pelialue täyttää kaiken tilan topbarin ja hintin välissä
- [ ] Touch-ohjaus koko pelialueella (ei nappeja)
- [ ] Näppäimistötuki PC:lle
- [ ] Aloitusruutu ohjeilla
- [ ] Game over -ruutu tuloksilla
- [ ] Overlayt reagoivat Enter/Space-näppäimeen (peli pelattavissa kokonaan näppäimistöllä)
- [ ] Pause-toiminnallisuus (P-näppäin + nappi)
- [ ] Ääniefektit (beep-funktio)
- [ ] High score localStorageen
- [ ] `resize()`-funktio joka skaalaa canvasin
- [ ] `requestAnimationFrame`-pohjainen game loop delta-ajalla
- [ ] Kaikki tekstit englanniksi
- [ ] LocalStorage-avain muotoa `pelinimi_hi`
- [ ] Automaattitestit ajettu ja läpi (ks. kohta 9)

---

## 11. Referenssitoteutus

Tetris-klooni (`tetris.html`) toimii referenssinä kaikille tuleville peleille. Se demonstroi:
- CRT-tyylin toteutuksen
- Topbar-layoutin (nimi + score vasemmalla, level + lines + next-preview oikealla)
- Koko pelialueen touch-ohjauksen ilman nappeja
- Canvas-skaalauksen mobiilille
- 7-bag randomizerin
- Ghost piece -efektin
- Lock delay -mekaniikan
- Overlay-ruutujen rakenteen

Käytä sitä pohjana ja sovella pelin vaatimuksiin.
