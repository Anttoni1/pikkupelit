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
├── breakout.html                  ← Breakout ✓
└── 2048.html                      ← 2048 ✓
```

## Tärkeimmät säännöt

### Ennen uuden pelin tekemistä
**Lue aina `retro-arcade-design-guide.md` kokonaan.** Se sisältää kaiken: visuaalinen tyyli, layout, touch-ohjaus, pelirakenne, testaus ja tarkistuslista.

### Teknologia
- Vanilla HTML/CSS/JS — ei frameworkeja, ei bundlereita
- Yksi tiedosto per peli — kaikki HTML, CSS ja JS samassa `.html`-tiedostossa
- Ainoa sallittu ulkoinen riippuvuus: Google Fonts (`Press Start 2P`)

### Visuaalinen identiteetti (ei neuvoteltavissa)
```css
:root {
  --g: #33ff33;                    /* pääväri */
  --gd: #1a8c1a;                   /* himmennetty vihreä */
  --bg: #070707;                   /* tausta */
  --glow: rgba(51,255,51,0.12);    /* hehku */
  --amber: #ffaa00;                /* korostus: pisteet, arvot */
}
```
- Fontti: `'Press Start 2P', monospace`
- CRT-efektit: scanlines (`::before`) + vinjetti (`::after`) `body.crt`-luokalla
- Canvas: `border: 2px solid var(--gd)`, `box-shadow: 0 0 15px var(--glow)`
- Overlay h1: `color: #d0ffd0` (ei `var(--g)` — teksti muuttuisi epäselväksi hehkuksi)

### Layout
```
TOPBAR (flex-shrink: 0)
  vasemmalla: pelin nimi (titleGlow) + SCORE / HI
  oikealla: pelikohtaiset statit (LEVEL, LIVES, NEXT jne.)
BOARD WRAP (flex: 1)
  canvas — kiinteä looginen resoluutio, CSS-koko skaalataan resize():llä
  pause-nappi — topbarin keskellä (vasemman ja oikean välissä), teksti "PAUSE"
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
Muoto: `pelinimi_hi` — esim. `tetris_hi`, `invaders_hi`, `breakout_hi`, `2048_hi`

### Ääni
Web Audio API, square wave, `beep(freq, duration, volume)`. Muista `audioCtx.resume()` — iOS luo AudioContextin suspended-tilassa.

## Pelit ja niiden LocalStorage-avaimet

| Peli | Tiedosto | localStorage |
|------|----------|-------------|
| Tetris | tetris.html | `tetris_hi` |
| Space Invaders | invaders.html | `invaders_hi` |
| Breakout | breakout.html | `breakout_hi` |
| 2048 | 2048.html | `2048_hi` |

## Pohja ja referenssipeli

`template.html` on pohja uusille peleille — kopioi se ja täytä `// TODO`-kohdat.

`tetris.html` on referenssipeli — katso sieltä miten asiat on toteutettu oikeassa pelissä, mutta älä kopioi sitä pohjana (sisältää tetris-spesifistä koodia: 7-bag, lock delay, ghost piece, next-preview).

## Tarkistuslista uudelle pelille

Ennen kuin pidät peliä valmiina:

- [ ] Yksi HTML-tiedosto, ei ulkoisia riippuvuuksia (paitsi Google Fonts)
- [ ] CRT-efektit `body.crt`-luokalla
- [ ] Samat CSS-muuttujat (`--g`, `--gd`, `--bg`, `--glow`, `--amber`)
- [ ] Press Start 2P -fontti
- [ ] iPhone meta-tagit + `viewport-fit=cover` + safe area insets
- [ ] `100dvh`, ei scrollausta, `touch-action: none`
- [ ] Pelialue täyttää kaiken tilan topbarin ja hintin välissä (`flex: 1`)
- [ ] Touch-ohjaus koko pelialueella — ei alareunan nappeja
- [ ] Näppäimistötuki (nuolet + Space/ArrowUp + P)
- [ ] Aloitusruutu: pelin nimi + ohjeet + vilkkuva "TAP TO START"
- [ ] Game over -ruutu: tulokset + "TAP TO RETRY"
- [ ] Enter/Space käynnistää/yrittää uudelleen overlayilta (pelattavissa ilman hiirtä)
- [ ] Pause: P-näppäin + pause-nappi, oma overlay — pauseBtn:llä sekä `click` että `touchend` (molemmat tarvitaan mobiilissa), overlay sisältää "EXIT TO MENU" -napin
- [ ] `beep()`-funktio, ääniefektit keskeisiin tapahtumiin
- [ ] High score `localStorage`een avaimella `pelinimi_hi`
- [ ] `resize()`-funktio, kutsutaan `onresize` + `setTimeout(resize, 50/200)`
- [ ] `requestAnimationFrame`-game loop delta-ajalla (`dt`)
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
