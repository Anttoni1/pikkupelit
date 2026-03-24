# Retro Arcade — CLAUDE.md

Tämä projekti on kokoelma retro-mobiilipelejä. Jokainen peli on yksittäinen HTML-tiedosto (vanilla HTML/CSS/JS, ei build-työkaluja, ei riippuvuuksia paitsi Google Fonts).

## Projektirakenne

```
/
├── CLAUDE.md
├── retro-arcade-design-guide.md   ← Koko design- ja tekninen ohje (LUE AINA)
├── template.html                  ← Pohja uudelle pelille (KOPIOI TÄSTÄ)
├── index.html                     ← Päävalikko ✓
├── tetris.html                    ← Referenssipeli ✓ (älä kopioi pohjana)
├── invaders.html                  ← Space Invaders ✓
├── arkanoid.html                  ← Arkanoid ✓
├── 2048.html                      ← 2048 ✓
├── asteroids.html                 ← Asteroids ✓
├── flappy.html                    ← Flappy Bird ✓
└── puzzle-bobble.html             ← Puzzle Bobble ✓
```

## Tärkeimmät säännöt

### Ennen uuden pelin tekemistä
**Lue aina `retro-arcade-design-guide.md` kokonaan.** Se sisältää kaiken: visuaalinen tyyli, layout, touch-ohjaus, pelirakenne, testaus ja tarkistuslista.

### Teknologia
- Vanilla HTML/CSS/JS — ei frameworkeja, ei bundlereita
- Yksi tiedosto per peli — kaikki HTML, CSS ja JS samassa `.html`-tiedostossa
- Ainoa sallittu ulkoinen riippuvuus: Google Fonts (`Press Start 2P`)

### Visuaalinen identiteetti
- Fontti: `'Press Start 2P', monospace`
- CRT-efektit: scanlines (`::before`, 4px välein, kevyt) + vinjetti (`::after`, kevyt) `body.crt`-luokalla
- Canvas: `border: 2px solid var(--gd)`, `box-shadow: 0 0 15px var(--glow)`
- Overlay h1: `color: #ffffff`
- CSS-muuttujat (`--g`, `--gd`, `--bg`, `--glow`, `--amber`) valitaan pelin teeman mukaan

### Layout
```
TOPBAR (flex-shrink: 0, justify-content: space-between)
  vasemmalla: pelin nimi (titleGlow) + SCORE / HI (topbar-left)
  keskellä: PAUSE-nappi (topbar-left ja topbar-right välissä)
  oikealla: pelikohtaiset statit (LEVEL, LIVES, NEXT jne.) (topbar-right)
BOARD WRAP (flex: 1, align-items: flex-start, padding: 0)
  canvas — istuu tiivisti topbarin alapuolella (ei pystysuoraa keskitystä)
  kiinteä looginen resoluutio, CSS-koko skaalataan resize():llä
TOUCH HINT (flex-shrink: 0)
  5px teksti, opacity: 0.3, swipe/tap-ohjeet
```

### Ohjaus
- **EI alareunan nappeja** — pelkästään swipe + tap + näppäimistö
- Touch sidotaan `#boardWrap`-elementtiin, koko pelialue on kosketusaluetta
- Tap (<250ms, <12px liike) = ensisijainen toiminto (rotate, fire, launch)
- Swipe vasen/oikea = liikkuminen (toistuva, SH=24px askel)
- Swipe alas = soft drop (SV=30px askel)
- Nopea swipe alas = hard drop (nopeus >1.5 px/ms)
- Näppäimistö: nuolet, Space/ArrowUp = toiminto, P = pause

### Kaikki tekstit englanniksi
SCORE, HI, LEVEL, LIVES, WAVE, NEXT, GAME OVER, TAP TO START, TAP TO RETRY, PAUSE jne.

### localStorage-avaimet
Muoto: `pelinimi_hi` — esim. `tetris_hi`, `invaders_hi`, `arkanoid_hi`, `2048_hi`

### Ääni
Enhanced `beep(freq, duration, opts)` — tukee `{vol, wave, attack, pitchEnd, detune}` tai pelkkä numero (volume). `beepNoise(dur, vol, filterFreq)` percussive-efekteihin. Muista `audioCtx.resume()` — iOS luo AudioContextin suspended-tilassa.

### Musiikki (MIDI-tyylinen sekvensseri)
Jokaisessa pelissä on taustamusiikki: pattern-pohjainen sekvensseri Web Audio API:lla. `SONG`-objekti määrittelee melodian, basson, arpeggion ja rummut. `music.play()/stop()/pause()/resume()` — hookit `startGame()`, `togglePause()`, `showGameOver()`. Musiikkitoggle: ♪-nappi topbarissa + M-näppäin. Asetus: `localStorage` avain `arcade_music` (`"1"` = päällä, oletus = pois).

## Pelit ja niiden LocalStorage-avaimet

| Peli | Tiedosto | localStorage |
|------|----------|-------------|
| Tetris | tetris.html | `tetris_hi` |
| Space Invaders | invaders.html | `invaders_hi` |
| Arkanoid | arkanoid.html | `arkanoid_hi` |
| 2048 | 2048.html | `2048_hi` |
| Asteroids | asteroids.html | `asteroids_hi` |
| Flappy Bird | flappy.html | `flappy_hi` |
| Puzzle Bobble | puzzle-bobble.html | `puzzlebobble_hi` |

## Pohja ja referenssipeli

`template.html` on pohja uusille peleille — kopioi se ja täytä `// TODO`-kohdat.

`tetris.html` on referenssipeli — katso sieltä miten asiat on toteutettu oikeassa pelissä, mutta älä kopioi sitä pohjana (sisältää tetris-spesifistä koodia: 7-bag, lock delay, ghost piece, next-preview).

## Tarkistuslista uudelle pelille

Ennen kuin pidät peliä valmiina:

- [ ] Yksi HTML-tiedosto, ei ulkoisia riippuvuuksia (paitsi Google Fonts)
- [ ] CRT-efektit `body.crt`-luokalla
- [ ] CSS-muuttujat (`--g`, `--gd`, `--bg`, `--glow`, `--amber`) pelin teemaan sopivina
- [ ] Press Start 2P -fontti
- [ ] iPhone meta-tagit + `viewport-fit=cover` + safe area insets: body `padding: 0 env(safe-area-inset-right) env(safe-area-inset-bottom) env(safe-area-inset-left)`, topbar `padding-top: calc(Xpx + env(safe-area-inset-top))`
- [ ] `100dvh`, ei scrollausta, `touch-action: none`
- [ ] Pelialue täyttää kaiken tilan topbarin ja hintin välissä (`flex: 1`)
- [ ] Touch-ohjaus koko pelialueella — ei alareunan nappeja
- [ ] Näppäimistötuki (nuolet + Space/ArrowUp + P/Escape)
- [ ] Aloitusruutu: pelin nimi + ohjeet + vilkkuva "TAP TO START"
- [ ] Game over -ruutu: tulokset + RETRY / EXIT TO MENU -napit (`pause-menu-btn`-tyylillä)
- [ ] Game over -näppäimistönavigointi: ↑↓ vaihtaa RETRY / EXIT TO MENU välillä, Enter/Space vahvistaa (EXIT TO MENU vie suoraan valikkoon, ei confirm-dialogia)
- [ ] Enter/Space käynnistää overlayilta (pelattavissa ilman hiirtä)
- [ ] Pause: P-näppäin + Escape pause/resume, pause-nappi, oma overlay — pauseBtn:llä sekä `click` että `touchend` (molemmat tarvitaan mobiilissa)
- [ ] Pause overlay -näppäimistönavigointi: ↑↓ vaihtaa valintaa RESUME / EXIT TO MENU välillä (EXIT TO MENU highlightautuu amber-väriseksi), Enter vahvistaa valitun, Escape/P/Escape aina sulkee pausen (resume)
- [ ] EXIT TO MENU avaa confirm-dialogi ("EXIT TO MENU?") jossa YES/NO valittavissa ←→ nuolilla tai Y/N-näppäimillä, Enter vahvistaa, Escape/N peruuttaa
- [ ] Enhanced `beep()`-funktio (`ensureAudio`, `beepNoise`), ääniefektit keskeisiin tapahtumiin
- [ ] MIDI-musiikki: `SONG`-objekti, `music`-sekvensseri, ♪-toggle topbarissa + M-näppäin
- [ ] Musiikki-integraatio: `music.play()` startGame:ssa, `music.pause()/resume()` togglePause:ssa, `music.stop()` showGameOver:ssa
- [ ] High score `localStorage`een avaimella `pelinimi_hi`
- [ ] `resize()`-funktio, kutsutaan `onresize` + `setTimeout(resize, 50/200)`
- [ ] `requestAnimationFrame`-game loop delta-ajalla (`dt`)
- [ ] Adaptiivinen laatu: `lowPerf`-lippu + `checkPerf()` joka framella; `ctx.shadowBlur` suojattu `if (!lowPerf)`:llä; gradientit cachessa (ei per frame)
- [ ] Kaikki tekstit englanniksi
- [ ] Automaattitestit (Playwright) — ks. design-guide kohta 9

## Uuden pelin luominen — workflow

1. Lue `retro-arcade-design-guide.md`
2. Kopioi `template.html` pohjaksi (ei `tetris.html` — se on referenssipeli, ei pohja)
3. Korvaa kaikki `GAMENAME` pelin nimellä ja `gamename_hi` oikealla localStorage-avaimella
4. Aseta canvas-resoluutio (`CANVAS_W`, `CANVAS_H`) pelin mukaan
5. Toteuta pelilogiikka `// TODO`-kohtiin
6. Lisää pelikohtaiset statit topbar-right:iin
7. Tarkista tarkistuslista
8. Aja Playwright-testit
9. Lisää peli `index.html`-valikkoon (GAMES-taulukkoon)
