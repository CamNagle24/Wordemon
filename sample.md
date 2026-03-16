<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>PokéWordle</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Fredoka+One&family=Fredoka:wght@400;600&display=swap" rel="stylesheet">
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    /* ══════════════════════════════════════════
       BACKGROUND  — red + explosion
    ══════════════════════════════════════════ */
    body {
      min-height: 100vh;
      background-color: #CC0000;
      font-family: 'Fredoka One', sans-serif;
      position: relative;
      overflow-x: hidden;
    }

    body::before {
      content: '';
      position: fixed; inset: 0;
      background: repeating-conic-gradient(
        rgba(255,255,255,0.14) 0deg 12deg,
        rgba(20,100,210,0.07) 12deg 24deg
      );
      pointer-events: none; z-index: 0;
    }

    body::after {
      content: '';
      position: fixed; inset: 0;
      background: radial-gradient(ellipse at 50% 50%,
        rgba(255,255,255,0.96) 0%,
        rgba(200,235,255,0.82) 12%,
        rgba(90,180,255,0.60) 28%,
        rgba(30,110,220,0.36) 48%,
        rgba(5,40,160,0.12) 65%,
        transparent 78%
      );
      pointer-events: none; z-index: 0;
    }

    /* ══════════════════════════════════════════
       SHELL
    ══════════════════════════════════════════ */
    #app { position: relative; z-index: 1; }

    .shell {
      max-width: 740px;
      margin: 0 auto;
      padding: 0 14px 60px;
    }

    /* ══════════════════════════════════════════
       HEADER
    ══════════════════════════════════════════ */
    header { text-align: center; padding: 18px 0 12px; }

    .logo {
      font-family: 'Press Start 2P', monospace;
      font-size: clamp(13px, 4.5vw, 22px);
      color: #FFD700;
      text-shadow:
        3px  3px 0 #7A5900,
        -2px -2px 0 #000,
         2px -2px 0 #000,
        -2px  2px 0 #000,
         2px  2px 0 #000,
         0    6px 14px rgba(0,0,0,0.5);
      letter-spacing: 1px;
    }

    .logo .w {
      color: #fff;
      text-shadow:
         3px  3px 0 #800000,
        -2px -2px 0 #000,
         2px -2px 0 #000,
        -2px  2px 0 #000;
    }

    .subtitle {
      margin-top: 6px;
      font-family: 'Press Start 2P', monospace;
      font-size: clamp(6px, 1.8vw, 9px);
      color: rgba(255,255,255,0.85);
      letter-spacing: 2px;
      text-shadow: 1px 2px 4px rgba(0,0,0,0.5);
    }

    /* ══════════════════════════════════════════
       TABS
    ══════════════════════════════════════════ */
    .tab-bar {
      display: flex;
      justify-content: center;
      gap: 8px;
      margin-bottom: 16px;
    }

    .tab-btn {
      font-family: 'Press Start 2P', monospace;
      font-size: 8px;
      padding: 9px 18px;
      border-radius: 6px;
      cursor: pointer;
      transition: all 0.15s;
      border: 3px solid #111;
      letter-spacing: 0.5px;
    }

    .tab-btn.inactive {
      background: rgba(255,255,255,0.15);
      color: rgba(255,255,255,0.8);
      box-shadow: 3px 3px 0 rgba(0,0,0,0.3);
    }

    .tab-btn.inactive:hover {
      background: rgba(255,255,255,0.25);
    }

    .tab-btn.active {
      background: #FFD700;
      color: #111;
      box-shadow: 4px 4px 0 #7A5900;
    }

    .tab-panel { display: none; }
    .tab-panel.visible { display: block; }

    /* ══════════════════════════════════════════
       GAME LAYOUT
    ══════════════════════════════════════════ */
    .game-layout {
      display: flex;
      gap: 14px;
      align-items: flex-start;
    }

    /* ── LEFT: Who's That Pokémon panel ── */
    .wtp-col {
      flex: 0 0 240px;
      position: sticky;
      top: 14px;
    }

    /* TV-screen outer bezel */
    .wtp-bezel {
      background: #111;
      border: 4px solid #111;
      border-radius: 14px;
      padding: 5px;
      box-shadow: 6px 6px 0 rgba(0,0,0,0.55);
    }

    /* Clean inner screen */
    .wtp-screen {
      border-radius: 10px;
      overflow: hidden;
      background: #d4d0c8;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 14px 10px 10px;
      min-height: 280px;
      position: relative;
      gap: 8px;
    }

    .wtp-headline {
      font-family: 'Press Start 2P', monospace;
      font-size: clamp(7px, 1.8vw, 9px);
      color: #333;
      text-align: center;
      line-height: 2;
      letter-spacing: 0.5px;
      text-shadow: 1px 1px 0 rgba(255,255,255,0.5);
      z-index: 1;
      position: relative;
    }

    .wtp-silhouette {
      width: min(190px, 48vw);
      height: min(190px, 48vw);
      object-fit: contain;
      image-rendering: pixelated;
      filter: brightness(0) drop-shadow(0 4px 8px rgba(0,0,0,0.5));
      transition: filter 1.2s ease;
      position: relative;
      z-index: 1;
    }

    @keyframes float {
      0%, 100% { transform: translateY(0); }
      50%      { transform: translateY(-8px); }
    }

    .wtp-silhouette.revealed {
      filter: brightness(1) drop-shadow(0 0 14px rgba(255,215,0,0.6));
      animation: float 2.5s ease-in-out infinite;
    }

    .wtp-qmark {
      font-family: 'Press Start 2P', monospace;
      font-size: 36px;
      color: rgba(0,0,0,0.08);
      position: absolute;
      bottom: 12px; right: 14px;
      z-index: 1;
      line-height: 1;
      pointer-events: none;
    }

    /* Pokéballs row */
    .lives {
      display: flex;
      justify-content: center;
      gap: 7px;
      flex-wrap: wrap;
      margin-top: 10px;
    }

    .pokeball {
      width: 28px; height: 28px;
      border-radius: 50%;
      background: linear-gradient(180deg, #FF2222 49%, #111 49%, #111 51%, #fff 51%);
      border: 3px solid #111;
      position: relative;
      box-shadow: 3px 3px 0 rgba(0,0,0,0.45);
      transition: filter .3s, opacity .3s;
      flex-shrink: 0;
    }

    .pokeball::after {
      content: '';
      position: absolute;
      top: 50%; left: 50%;
      transform: translate(-50%, -50%);
      width: 9px; height: 9px;
      border-radius: 50%;
      background: #fff;
      border: 2px solid #111;
    }

    .pokeball.used { filter: grayscale(1); opacity: 0.3; }

    /* ── RIGHT: hints + guess ── */
    .game-col { flex: 1; min-width: 0; }

    /* ══════════════════════════════════════════
       HINT CARDS  — retro offset-shadow style
    ══════════════════════════════════════════ */
    .hints-container {
      display: flex;
      flex-direction: column;
      gap: 10px;
      margin-bottom: 12px;
    }

    @keyframes slideDown {
      from { opacity: 0; transform: translateY(-10px) scale(0.96); }
      to   { opacity: 1; transform: translateY(0) scale(1); }
    }

    /* Retro RPG dialog box style */
    .hint-card {
      background: #f5f0e8;
      border: 3px solid #888;
      border-radius: 4px;
      box-shadow:
        inset 2px 2px 0 rgba(255,255,255,0.5),
        4px 4px 0 #555;
      padding: 10px 14px;
      display: flex;
      align-items: center;
      gap: 10px;
      animation: slideDown 0.3s cubic-bezier(0.34,1.56,0.64,1);
      position: relative;
    }

    /* Pointer arrow like Pokemon menu */
    .hint-card::before {
      content: '\25B6';
      font-size: 10px;
      color: #CC0000;
      flex-shrink: 0;
    }

    .hint-icon { font-size: 20px; flex-shrink: 0; }

    .hint-label {
      font-family: 'Press Start 2P', monospace;
      font-size: 7px;
      text-transform: uppercase;
      letter-spacing: 1px;
      color: #888;
      margin-bottom: 4px;
    }

    .hint-value {
      font-family: 'Press Start 2P', monospace;
      font-size: 9px;
      color: #333;
      line-height: 1.8;
    }

    .hint-value strong { color: #CC0000; }

    /* ══════════════════════════════════════════
       TYPE BADGES
    ══════════════════════════════════════════ */
    .type-badge {
      display: inline-block;
      padding: 3px 8px;
      border-radius: 2px;
      font-family: 'Press Start 2P', monospace;
      font-size: 7px;
      color: #fff;
      text-shadow: 1px 1px 0 rgba(0,0,0,0.5);
      margin: 1px 2px;
      border: 2px solid rgba(255,255,255,0.2);
      box-shadow: 2px 2px 0 rgba(0,0,0,0.4);
      letter-spacing: 0.5px;
      line-height: 1.6;
    }

    /* ══════════════════════════════════════════
       GUESS AREA
    ══════════════════════════════════════════ */
    .guess-area {
      background: #f5f0e8;
      border: 3px solid #888;
      border-radius: 4px;
      box-shadow: 4px 4px 0 #555;
      padding: 12px;
      margin-bottom: 12px;
    }

    .guess-form { display: flex; gap: 8px; }
    .guess-input-wrapper { flex: 1; position: relative; }

    .guess-input {
      width: 100%;
      padding: 10px 12px;
      font-family: 'Press Start 2P', monospace;
      font-size: 10px;
      border: 3px solid #aaa;
      border-radius: 3px;
      outline: none;
      background: #fff;
      color: #333;
      transition: border-color .2s;
      text-transform: uppercase;
      letter-spacing: 0.5px;
      line-height: 1.6;
    }

    .guess-input::placeholder {
      color: #bbb;
      font-size: 8px;
    }

    .guess-input:focus {
      border-color: #CC0000;
    }

    @keyframes shake {
      0%,100% { transform: translateX(0); }
      20%     { transform: translateX(-9px); }
      40%     { transform: translateX(9px); }
      60%     { transform: translateX(-6px); }
      80%     { transform: translateX(6px); }
    }
    .guess-input.shake { animation: shake .4s ease; }

    .guess-btn {
      padding: 10px 14px;
      background: #CC0000;
      color: #fff;
      border: 3px solid #888;
      border-radius: 3px;
      font-family: 'Press Start 2P', monospace;
      font-size: 7px;
      cursor: pointer;
      white-space: nowrap;
      box-shadow: 4px 4px 0 #555;
      transition: transform .1s, box-shadow .1s;
      line-height: 1.8;
    }

    .guess-btn:hover  { transform: translate(-1px,-1px); box-shadow: 5px 5px 0 #555; }
    .guess-btn:active { transform: translate(2px,2px);   box-shadow: 2px 2px 0 #555; }
    .guess-btn:disabled { background: #aaa; color: #ddd; cursor: not-allowed; transform: none; }

    .error-msg {
      font-family: 'Press Start 2P', monospace;
      font-size: 7px;
      color: #CC0000;
      margin-top: 6px;
      min-height: 16px;
      line-height: 1.7;
    }

    /* ── Autocomplete — RPG menu style ── */
    .autocomplete-dropdown {
      position: absolute;
      top: calc(100% + 4px);
      left: 0; right: 0;
      background: #f5f0e8;
      border: 3px solid #888;
      border-radius: 3px;
      max-height: 200px;
      overflow-y: auto;
      z-index: 100;
      box-shadow: 4px 4px 0 #555;
    }

    .autocomplete-dropdown::-webkit-scrollbar { width: 4px; }
    .autocomplete-dropdown::-webkit-scrollbar-thumb { background: #bbb; }

    .autocomplete-item {
      padding: 8px 12px;
      cursor: pointer;
      font-family: 'Press Start 2P', monospace;
      font-size: 8px;
      color: #444;
      border-bottom: 1px solid #ddd5c8;
      transition: background .1s, color .1s;
      text-transform: uppercase;
      letter-spacing: 0.5px;
      line-height: 1.6;
    }

    .autocomplete-item:last-child { border-bottom: none; }
    .autocomplete-item:hover, .autocomplete-item.active {
      background: #CC0000; color: #fff;
    }

    /* ── Previous guesses — retro RPG log ── */
    .guesses-list {
      display: flex;
      flex-direction: column;
      gap: 5px;
      margin-bottom: 12px;
    }

    .guess-item {
      background: #f0ebe0;
      border: 2px solid #bbb;
      border-radius: 3px;
      box-shadow: 3px 3px 0 #888;
      padding: 8px 12px;
      display: flex;
      align-items: center;
      gap: 10px;
      font-family: 'Press Start 2P', monospace;
      font-size: 8px;
      color: #999;
      text-transform: uppercase;
      animation: slideDown .22s ease;
      letter-spacing: 0.5px;
      line-height: 1.6;
      text-decoration: line-through;
      text-decoration-color: #cc6666;
    }

    .guess-item span:first-child {
      font-size: 12px;
      text-decoration: none;
    }

    .status-bar { text-align: center; margin-top: 12px; }

    .status-pill {
      display: inline-block;
      font-family: 'Press Start 2P', monospace;
      font-size: 8px;
      color: #FFD700;
      background: #111;
      padding: 7px 16px;
      border-radius: 4px;
      border: 2px solid #333;
      box-shadow: 3px 3px 0 rgba(0,0,0,0.4);
      letter-spacing: 0.5px;
      line-height: 1.6;
    }

    /* ══════════════════════════════════════════
       POKÉDEX TAB
    ══════════════════════════════════════════ */
    .pokedex-shell {
      background: #111;
      border: 4px solid #111;
      border-radius: 18px;
      overflow: hidden;
      box-shadow: 6px 6px 0 rgba(0,0,0,0.5);
    }

    .pokedex-header {
      background: #CC0000;
      padding: 14px 18px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      border-bottom: 5px solid #800000;
    }

    .pokedex-lights { display: flex; gap: 5px; align-items: center; }

    .pdx-light {
      border-radius: 50%;
      border: 2px solid rgba(0,0,0,0.4);
    }

    .pdx-light.big   { width: 26px; height: 26px; background: radial-gradient(circle at 38% 38%, #aaeeff, #0077cc); box-shadow: 0 0 6px #0af; }
    .pdx-light.small { width: 11px; height: 11px; }
    .pdx-light.r { background: radial-gradient(circle at 38% 38%, #ff9988, #ff2200); }
    .pdx-light.y { background: radial-gradient(circle at 38% 38%, #ffee88, #ffaa00); }
    .pdx-light.g { background: radial-gradient(circle at 38% 38%, #88ffaa, #00cc44); }

    .pokedex-title {
      font-family: 'Press Start 2P', monospace;
      font-size: clamp(9px, 2.5vw, 13px);
      color: #FFD700;
      text-shadow: 2px 2px 0 rgba(0,0,0,0.6);
      letter-spacing: 2px;
    }

    .pokedex-count {
      font-family: 'Press Start 2P', monospace;
      font-size: 8px;
      color: #FFD700;
      background: rgba(0,0,0,0.35);
      padding: 5px 10px;
      border-radius: 4px;
      border: 2px solid rgba(0,0,0,0.3);
    }

    /* Pokédex hinge divider */
    .pokedex-hinge {
      height: 12px;
      background: linear-gradient(180deg, #880000 0%, #330000 100%);
      border-bottom: 3px solid #111;
    }

    /* Screen area */
    .pokedex-body {
      background: #0a0a18;
      padding: 14px;
    }

    .pokedex-screen-top {
      background: #111;
      border: 3px solid #222;
      border-bottom: none;
      border-radius: 8px 8px 0 0;
      padding: 7px 14px;
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .screen-led {
      width: 7px; height: 7px;
      border-radius: 50%;
      background: #00ff44;
      box-shadow: 0 0 6px #00ff44;
    }

    .screen-label {
      font-family: 'Press Start 2P', monospace;
      font-size: 7px;
      color: #444;
      letter-spacing: 1px;
    }

    .pokedex-list {
      background: #0d1117;
      border: 3px solid #222;
      border-top: none;
      border-radius: 0 0 8px 8px;
      padding: 10px;
      min-height: 280px;
    }

    .pokedex-empty {
      text-align: center;
      padding: 60px 20px;
      font-family: 'Press Start 2P', monospace;
      font-size: 8px;
      color: #223;
      line-height: 2.2;
    }

    .pdx-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
      gap: 10px;
    }

    @keyframes entryPop {
      from { opacity: 0; transform: scale(0.8); }
      to   { opacity: 1; transform: scale(1); }
    }

    .pdx-entry {
      background: #12121f;
      border: 2px solid #252540;
      border-radius: 8px;
      padding: 12px 8px 10px;
      text-align: center;
      cursor: pointer;
      transition: border-color .2s, transform .15s, box-shadow .2s;
      animation: entryPop .3s cubic-bezier(.34,1.56,.64,1);
      box-shadow: 3px 3px 0 #000;
    }

    .pdx-entry:hover {
      border-color: #FFD700;
      transform: translate(-2px,-2px);
      box-shadow: 5px 5px 0 #7A5900;
    }

    .pdx-entry-num {
      font-family: 'Press Start 2P', monospace;
      font-size: 7px;
      color: #334;
      margin-bottom: 6px;
    }

    .pdx-entry-sprite {
      width: 70px; height: 70px;
      object-fit: contain;
      image-rendering: pixelated;
      display: block;
      margin: 0 auto 6px;
      filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5));
    }

    .pdx-entry-name {
      font-family: 'Fredoka One', sans-serif;
      font-size: 13px;
      color: #bbbbdd;
      text-transform: capitalize;
      margin-bottom: 6px;
    }

    .pdx-entry-types { display: flex; justify-content: center; gap: 3px; flex-wrap: wrap; }

    .pdx-type-badge {
      font-family: 'Press Start 2P', monospace;
      font-size: 6px;
      color: #fff;
      padding: 2px 6px;
      border-radius: 3px;
      text-shadow: 0 1px 2px rgba(0,0,0,0.4);
      border: 1px solid rgba(255,255,255,0.2);
    }

    /* Pokédex detail overlay */
    .pdx-detail-overlay {
      position: fixed; inset: 0;
      background: rgba(0,0,0,0.8);
      z-index: 300;
      display: flex; align-items: center; justify-content: center;
      padding: 20px;
      animation: fadeIn .2s ease;
    }

    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

    .pdx-detail {
      background: #12121f;
      border: 4px solid #252540;
      border-radius: 16px;
      padding: 24px 20px 20px;
      max-width: 340px;
      width: 100%;
      text-align: center;
      box-shadow: 6px 6px 0 rgba(0,0,0,0.7);
      animation: popIn .35s cubic-bezier(.34,1.56,.64,1);
      position: relative;
    }

    @keyframes popIn {
      from { opacity: 0; transform: scale(0.7); }
      to   { opacity: 1; transform: scale(1); }
    }

    .pdx-detail-close {
      position: absolute;
      top: 10px; right: 14px;
      background: none; border: none;
      color: #444;
      font-size: 22px;
      cursor: pointer;
      transition: color .2s;
      line-height: 1;
    }

    .pdx-detail-close:hover { color: #FFD700; }

    .pdx-detail-num {
      font-family: 'Press Start 2P', monospace;
      font-size: 8px; color: #334; margin-bottom: 6px;
    }

    .pdx-detail-sprite {
      width: 120px; height: 120px;
      object-fit: contain;
      image-rendering: pixelated;
      filter: drop-shadow(0 4px 10px rgba(255,215,0,0.3));
      animation: bounceSpr .45s cubic-bezier(.34,1.56,.64,1) .1s both, float 2.5s ease-in-out .5s infinite;
    }

    @keyframes bounceSpr {
      from { transform: scale(0) rotate(-10deg); }
      to   { transform: scale(1) rotate(0); }
    }

    .pdx-detail-name {
      font-family: 'Fredoka One', sans-serif;
      font-size: 24px; color: #eee;
      text-transform: capitalize; margin: 6px 0 8px;
    }

    .pdx-detail-types { display: flex; justify-content: center; gap: 8px; margin-bottom: 8px; }

    .pdx-detail-meta {
      font-family: 'Press Start 2P', monospace;
      font-size: 7px; color: #334;
      line-height: 2; margin-bottom: 12px;
    }

    .pdx-detail-flavor {
      font-family: 'Fredoka', sans-serif;
      font-size: 13px; color: #778;
      font-style: italic; line-height: 1.6;
      padding: 10px 12px;
      background: rgba(255,255,255,0.04);
      border-radius: 8px;
      border-left: 3px solid #CC0000;
      text-align: left;
    }

    /* ══════════════════════════════════════════
       WIN / LOSE MODAL
    ══════════════════════════════════════════ */
    .modal-overlay {
      position: fixed; inset: 0;
      background: rgba(0,0,0,0.75);
      z-index: 200;
      display: flex; align-items: center; justify-content: center;
      padding: 20px;
      animation: fadeIn .3s ease;
    }

    .modal {
      background: #FFFDE7;
      border: 4px solid #111;
      border-radius: 12px;
      padding: 24px 20px 20px;
      max-width: 370px; width: 100%;
      text-align: center;
      box-shadow: 6px 6px 0 #111;
      animation: popIn .4s cubic-bezier(.34,1.56,.64,1);
    }

    .modal.win  { border-top: 8px solid #FFD700; }
    .modal.lose { border-top: 8px solid #CC0000; }

    .modal-title {
      font-family: 'Press Start 2P', monospace;
      font-size: clamp(10px, 3vw, 14px);
      margin-bottom: 4px; line-height: 1.8;
    }

    .modal.win  .modal-title { color: #CC8800; }
    .modal.lose .modal-title { color: #CC0000; }

    .modal-pokemon-name {
      font-family: 'Fredoka One', sans-serif;
      font-size: 26px; color: #1a1a1a;
      text-transform: capitalize; margin-bottom: 8px;
    }

    .modal-sprite {
      width: 140px; height: 140px;
      object-fit: contain; image-rendering: pixelated;
      animation: bounceSpr .5s cubic-bezier(.34,1.56,.64,1) .15s both, float 2.5s ease-in-out .7s infinite;
    }

    .modal-types { display: flex; justify-content: center; gap: 8px; margin: 8px 0; }

    .modal-meta {
      font-family: 'Press Start 2P', monospace;
      font-size: 7px; color: #888; margin-bottom: 10px; line-height: 2;
    }

    .modal-flavor {
      font-family: 'Fredoka', sans-serif;
      font-size: 13px; color: #555; font-style: italic;
      line-height: 1.6; padding: 10px 12px;
      background: rgba(0,0,0,0.05);
      border-radius: 8px; margin: 10px 0 16px;
      border-left: 4px solid #CC0000; text-align: left;
    }

    .play-again-btn {
      padding: 12px 22px;
      background: #CC0000; color: #FFD700;
      border: 3px solid #111; border-radius: 6px;
      font-family: 'Press Start 2P', monospace;
      font-size: 8px; cursor: pointer;
      box-shadow: 4px 4px 0 #111;
      transition: transform .1s, box-shadow .1s;
    }

    .play-again-btn:hover  { transform: translate(-1px,-1px); box-shadow: 5px 5px 0 #111; }
    .play-again-btn:active { transform: translate(2px,2px);   box-shadow: 2px 2px 0 #111; }

    /* ── Loading ── */
    .loading-screen {
      display: flex; flex-direction: column;
      align-items: center; justify-content: center;
      min-height: 70vh; gap: 20px;
    }

    @keyframes spin { to { transform: rotate(360deg); } }

    .loading-pokeball {
      width: 64px; height: 64px; border-radius: 50%;
      background: linear-gradient(180deg, #FF2222 49%, #111 49%, #111 51%, #fff 51%);
      border: 5px solid #111;
      position: relative;
      animation: spin .85s linear infinite;
      box-shadow: 4px 4px 0 rgba(0,0,0,0.4);
    }

    .loading-pokeball::after {
      content: '';
      position: absolute; top: 50%; left: 50%;
      transform: translate(-50%,-50%);
      width: 16px; height: 16px;
      border-radius: 50%; background: #fff; border: 3px solid #111;
    }

    .loading-text {
      font-family: 'Press Start 2P', monospace;
      font-size: 10px; color: rgba(255,255,255,.92);
      text-shadow: 2px 2px 4px rgba(0,0,0,.5);
    }

    /* ── Confetti ── */
    .confetti-container { position: fixed; inset: 0; pointer-events: none; z-index: 400; }

    @keyframes confettiFall {
      from { transform: translateY(-20px) rotate(0deg); opacity: 1; }
      85%  { opacity: 1; }
      to   { transform: translateY(105vh) rotate(600deg); opacity: 0; }
    }

    .confetti-piece { position: absolute; animation: confettiFall linear forwards; }

    /* ── Responsive ── */
    @media (max-width: 560px) {
      .wtp-col { flex: 0 0 170px; }
      .wtp-silhouette { width: min(140px, 38vw); height: min(140px, 38vw); }
      .wtp-screen { min-height: 220px; }
      .hint-value { font-size: 8px; }
    }

    @media (max-width: 400px) {
      .game-layout { flex-direction: column; }
      .wtp-col { flex: none; width: 100%; position: static; }
      .wtp-screen { min-height: 200px; }
      .wtp-headline { font-size: 8px; }
      .wtp-qmark { display: none; }
    }
  </style>
</head>
<body>
<div id="app">
  <div class="shell">
    <div class="loading-screen">
      <div class="loading-pokeball"></div>
      <div class="loading-text">Loading...</div>
    </div>
  </div>
</div>

<script>
// ════════════════════════════════════════════
// CONSTANTS
// ════════════════════════════════════════════

const GEN_TO_REGION = {
  1:'Kanto',2:'Johto',3:'Hoenn',4:'Sinnoh',
  5:'Unova',6:'Kalos',7:'Alola',8:'Galar',9:'Paldea'
};

const GEN_NAME_MAP = {
  'generation-i':1,'generation-ii':2,'generation-iii':3,
  'generation-iv':4,'generation-v':5,'generation-vi':6,
  'generation-vii':7,'generation-viii':8,'generation-ix':9
};

const GEN_ROMAN = ['I','II','III','IV','V','VI','VII','VIII','IX'];

const TYPE_COLORS = {
  normal:'#9A9A78', fire:'#F08030', water:'#6890F0', electric:'#F8D030',
  grass:'#78C850', ice:'#98D8D8', fighting:'#C03028', poison:'#A040A0',
  ground:'#E0C068', flying:'#A890F0', psychic:'#F85888', bug:'#A8B820',
  rock:'#B8A038', ghost:'#705898', dragon:'#7038F8', dark:'#705848',
  steel:'#B8B8D0', fairy:'#EE99AC'
};

const MAX_GUESSES = 6;
const POKEDEX_KEY = 'pokewordle_pokedex';

// ════════════════════════════════════════════
// STATE
// ════════════════════════════════════════════

let S = {
  answer:null, answerId:null, hints:null,
  guesses:[], step:0,
  gameOver:false, won:false,
  allPokemon:[], flavorText:'',
  autoValue:'', dropdownOpen:false,
  dropdownItems:[], activeIdx:-1,
  activeTab:'game',
};

let pokedex = JSON.parse(localStorage.getItem(POKEDEX_KEY) || '[]');

// ════════════════════════════════════════════
// HELPERS
// ════════════════════════════════════════════

const cap    = s => s.charAt(0).toUpperCase() + s.slice(1);
const capAll = s => s.split('-').map(cap).join(' ');
const pad3   = n => String(n).padStart(3,'0');

function typeBadge(t, small=false) {
  const c = TYPE_COLORS[t.toLowerCase()] || '#888';
  const cl = small ? 'pdx-type-badge' : 'type-badge';
  return `<span class="${cl}" style="background:${c}">${t}</span>`;
}

async function fetchJSON(url) {
  const r = await fetch(url);
  if (!r.ok) throw new Error(`HTTP ${r.status}`);
  return r.json();
}

function getEvoStage(chain, name) {
  function depth(n, t) {
    if (n.species.name === t) return 0;
    for (const c of n.evolves_to) { const d=depth(c,t); if(d!==null) return d+1; }
    return null;
  }
  function isFinal(n, t) {
    if (n.species.name === t) return n.evolves_to.length===0;
    for (const c of n.evolves_to) { const r=isFinal(c,t); if(r!==null) return r; }
    return null;
  }
  function maxD(n) {
    if (!n.evolves_to.length) return 0;
    return 1 + Math.max(...n.evolves_to.map(maxD));
  }
  const d=depth(chain,name); if(d===null) return 'Unknown';
  const max=maxD(chain), fin=isFinal(chain,name);
  if(max===0) return 'Standalone (Does Not Evolve)';
  if(d===0) return 'Basic (First Stage)';
  if(fin) return max>=2 ? 'Final Evolution (Stage 3)' : 'Final Evolution (Stage 2)';
  return `Middle Evolution (Stage ${d+1})`;
}

// ════════════════════════════════════════════
// LOAD
// ════════════════════════════════════════════

async function loadGame() {
  try {
    const list = await fetchJSON('https://pokeapi.co/api/v2/pokemon?limit=1025&offset=0');
    S.allPokemon = list.results.map(p=>({name:p.name, display:capAll(p.name)}));

    const id = Math.floor(Math.random()*1025)+1;
    S.answerId = id;

    const [poke, species] = await Promise.all([
      fetchJSON(`https://pokeapi.co/api/v2/pokemon/${id}`),
      fetchJSON(`https://pokeapi.co/api/v2/pokemon-species/${id}`)
    ]);

    const evoData  = await fetchJSON(species.evolution_chain.url);
    const genNum   = GEN_NAME_MAP[species.generation.name] || 1;
    const genLabel = `Generation ${GEN_ROMAN[genNum-1]}`;
    const region   = GEN_TO_REGION[genNum] || 'Unknown';
    const types    = poke.types.map(t=>cap(t.type.name));
    const sprite   = poke.sprites.front_default ||
      `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${id}.png`;
    const evoStage = getEvoStage(evoData.chain, poke.name);
    const flavors  = species.flavor_text_entries.filter(e=>e.language.name==='en');
    const flavor   = flavors.length
      ? flavors[flavors.length-1].flavor_text.replace(/[\f\n]/g,' ')
      : 'A mysterious Pokémon.';

    S.answer     = poke.name;
    S.flavorText = flavor;
    S.hints      = {
      sprite, nameLength:poke.name.length,
      firstLetter:poke.name[0].toUpperCase(),
      lastLetter:poke.name[poke.name.length-1].toUpperCase(),
      generation:genLabel, evoStage, region, types
    };
    S.step = 1;
    render();
  } catch(e) {
    console.error(e);
    document.getElementById('app').innerHTML = `
      <div class="shell"><div class="loading-screen">
        <div style="font-family:'Press Start 2P',monospace;font-size:9px;color:#FFD700;text-align:center;line-height:2.4">
          FAILED TO LOAD!<br>CHECK CONNECTION
        </div>
        <button class="play-again-btn" onclick="location.reload()">RETRY</button>
      </div></div>`;
  }
}

// ════════════════════════════════════════════
// GAME LOGIC
// ════════════════════════════════════════════

function submitGuess(raw) {
  const g = raw.trim().toLowerCase().replace(/\s+/g,'-');
  if (!g || S.gameOver) return null;
  if (g === S.answer) { S.won=true; S.gameOver=true; saveToPokedex(); return 'win'; }
  S.guesses.push(g);
  S.step = Math.min(S.guesses.length+1, MAX_GUESSES);
  if (S.guesses.length >= MAX_GUESSES) { S.gameOver=true; return 'lose'; }
  return 'wrong';
}

function saveToPokedex() {
  if (pokedex.some(p=>p.name===S.answer)) return;
  pokedex.push({
    id:S.answerId, name:S.answer, sprite:S.hints.sprite,
    types:S.hints.types, generation:S.hints.generation,
    region:S.hints.region, evoStage:S.hints.evoStage, flavor:S.flavorText
  });
  pokedex.sort((a,b)=>a.id-b.id);
  localStorage.setItem(POKEDEX_KEY, JSON.stringify(pokedex));
}

// ════════════════════════════════════════════
// HTML BUILDERS
// ════════════════════════════════════════════

function wtpPanelHTML() {
  const revealedClass = S.gameOver ? ' revealed' : '';
  return `
    <div class="wtp-col">
      <div class="wtp-bezel">
        <div class="wtp-screen">
          <div class="wtp-headline">WHO'S<br>THAT<br>POKEMON?</div>
          <img id="silhouette-img"
            class="wtp-silhouette${revealedClass}"
            src="${S.hints.sprite}" alt="silhouette" />
          <div class="wtp-qmark">?</div>
        </div>
      </div>
      <div class="lives">${pokeballsHTML()}</div>
    </div>`;
}

function pokeballsHTML() {
  let h = '';
  for (let i=0; i<MAX_GUESSES; i++) {
    h += `<div class="pokeball${i<S.guesses.length?' used':''}"></div>`;
  }
  return h;
}

function hintCardsHTML() {
  const {hints,step} = S;
  let h = '';
  if (step>=2) h += `<div class="hint-card step-2">
    <div class="hint-icon">&#128220;</div>
    <div class="hint-text">
      <div class="hint-label">Name Length</div>
      <div class="hint-value">Name has <strong>${hints.nameLength} letters</strong></div>
    </div></div>`;
  if (step>=3) h += `<div class="hint-card step-3">
    <div class="hint-icon">&#128286;</div>
    <div class="hint-text">
      <div class="hint-label">First Letter</div>
      <div class="hint-value">Name starts with <strong>${hints.firstLetter}</strong></div>
    </div></div>`;
  if (step>=4) h += `<div class="hint-card step-4">
    <div class="hint-icon">&#128197;</div>
    <div class="hint-text">
      <div class="hint-label">Generation</div>
      <div class="hint-value">From <strong>${hints.generation}</strong></div>
    </div></div>`;
  if (step>=5) h += `<div class="hint-card step-5">
    <div class="hint-icon">&#127991;</div>
    <div class="hint-text">
      <div class="hint-label">Type</div>
      <div class="hint-value">${hints.types.map(t=>typeBadge(t)).join(' ')}</div>
    </div></div>`;
  if (step>=6) h += `<div class="hint-card step-6">
    <div class="hint-icon">&#128285;</div>
    <div class="hint-text">
      <div class="hint-label">Last Letter</div>
      <div class="hint-value">Name ends with <strong>${hints.lastLetter}</strong></div>
    </div></div>`;
  return h;
}

function guessesHTML() {
  if (!S.guesses.length) return '';
  let h = '<div class="guesses-list">';
  for (const g of S.guesses)
    h += `<div class="guess-item"><span>&#10060;</span><span>${capAll(g)}</span></div>`;
  return h+'</div>';
}

function dropdownHTML() {
  if (!S.dropdownOpen || !S.dropdownItems.length) return '';
  let h = '<div class="autocomplete-dropdown" id="ac-dd">';
  S.dropdownItems.forEach((item,i)=>{
    h += `<div class="autocomplete-item${i===S.activeIdx?' active':''}" data-name="${item.name}">${item.display}</div>`;
  });
  return h+'</div>';
}

function escAttr(s) { return s.replace(/"/g,'&quot;').replace(/</g,'&lt;'); }

function statusText() {
  if (S.won)      return 'YOU GOT IT!';
  if (S.gameOver) return 'GAME OVER!';
  return `${MAX_GUESSES - S.guesses.length} GUESSES LEFT`;
}

// ════════════════════════════════════════════
// RENDER
// ════════════════════════════════════════════

function render() {
  const guessArea = S.gameOver ? '' : `
    <div class="guess-area">
      <div class="guess-form">
        <div class="guess-input-wrapper">
          <input type="text" id="guess-input" class="guess-input"
            placeholder="Enter Pokémon name..."
            autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false" name="pokemon-guess-nofill" data-form-type="other"
            value="${escAttr(S.autoValue)}" />
          ${dropdownHTML()}
        </div>
        <button class="guess-btn" id="guess-btn">GUESS!</button>
      </div>
      <div id="guess-error" class="error-msg"></div>
    </div>`;

  const pokedexContent = pokedex.length===0
    ? `<div class="pokedex-empty">NO POKEMON<br>CAUGHT YET!<br><br>WIN A GAME<br>TO START YOUR<br>POKEDEX.</div>`
    : `<div class="pdx-grid">${pokedex.map(p=>`
        <div class="pdx-entry" onclick="showPdxDetail('${p.name}')">
          <div class="pdx-entry-num">#${pad3(p.id)}</div>
          <img class="pdx-entry-sprite" src="${p.sprite}" alt="${p.name}"/>
          <div class="pdx-entry-name">${capAll(p.name)}</div>
          <div class="pdx-entry-types">${p.types.map(t=>typeBadge(t,true)).join('')}</div>
        </div>`).join('')}
      </div>`;

  const countBadge = pokedex.length>0
    ? `<span style="font-size:9px;opacity:0.75">(${pokedex.length})</span>` : '';

  document.getElementById('app').innerHTML = `
    <div class="shell">
      <header>
        <div class="logo">Poké<span class="w">Wordle</span></div>
        <div class="subtitle">- GUESS THE POKEMON -</div>
      </header>

      <div class="tab-bar">
        <button class="tab-btn${S.activeTab==='game'?' active':' inactive'}" onclick="switchTab('game')">GAME</button>
        <button class="tab-btn${S.activeTab==='pokedex'?' active':' inactive'}" onclick="switchTab('pokedex')">POKEDEX ${countBadge}</button>
      </div>

      <div class="tab-panel${S.activeTab==='game'?' visible':''}">
        <div class="game-layout">
          ${wtpPanelHTML()}
          <div class="game-col">
            <div class="hints-container">${hintCardsHTML()}</div>
            ${guessesHTML()}
            ${guessArea}
            <div class="status-bar"><span class="status-pill">${statusText()}</span></div>
          </div>
        </div>
      </div>

      <div class="tab-panel${S.activeTab==='pokedex'?' visible':''}">
        <div class="pokedex-shell">
          <div class="pokedex-header">
            <div class="pokedex-lights">
              <div class="pdx-light big"></div>
              <div class="pdx-light small r"></div>
              <div class="pdx-light small y"></div>
              <div class="pdx-light small g"></div>
            </div>
            <div class="pokedex-title">POKEDEX</div>
            <div class="pokedex-count">${pokedex.length}/1025</div>
          </div>
          <div class="pokedex-hinge"></div>
          <div class="pokedex-body">
            <div class="pokedex-screen-top">
              <div class="screen-led"></div>
              <div class="screen-label">CAUGHT POKEMON</div>
            </div>
            <div class="pokedex-list">${pokedexContent}</div>
          </div>
        </div>
      </div>
    </div>`;

  attachListeners();
  if (S.gameOver && S.activeTab==='game') showModal();
}

// ════════════════════════════════════════════
// TABS
// ════════════════════════════════════════════

window.switchTab = tab => { S.activeTab=tab; render(); };

// ════════════════════════════════════════════
// POKÉDEX DETAIL
// ════════════════════════════════════════════

window.showPdxDetail = name => {
  const p = pokedex.find(x=>x.name===name);
  if (!p) return;
  const overlay = document.createElement('div');
  overlay.className = 'pdx-detail-overlay';
  overlay.innerHTML = `
    <div class="pdx-detail">
      <button class="pdx-detail-close" id="pdx-close">&times;</button>
      <div class="pdx-detail-num">#${pad3(p.id)}</div>
      <img class="pdx-detail-sprite" src="${p.sprite}" alt="${p.name}"/>
      <div class="pdx-detail-name">${capAll(p.name)}</div>
      <div class="pdx-detail-types">${p.types.map(t=>typeBadge(t)).join('')}</div>
      <div class="pdx-detail-meta">${p.generation}<br>${p.region} &bull; ${p.evoStage}</div>
      <div class="pdx-detail-flavor">"${p.flavor}"</div>
    </div>`;
  document.body.appendChild(overlay);
  overlay.querySelector('#pdx-close').addEventListener('click', ()=>overlay.remove());
  overlay.addEventListener('click', e=>{ if(e.target===overlay) overlay.remove(); });
};

// ════════════════════════════════════════════
// LISTENERS
// ════════════════════════════════════════════

function attachListeners() {
  const input = document.getElementById('guess-input');
  const btn   = document.getElementById('guess-btn');
  if (!input) return;
  input.focus();

  input.addEventListener('input', e=>{
    S.autoValue = e.target.value;
    S.activeIdx = -1;
    const q = S.autoValue.trim().toLowerCase().replace(/\s+/g,'-');
    if (q.length>=1) {
      S.dropdownItems = S.allPokemon.filter(p=>p.name.startsWith(q)).slice(0,8);
      S.dropdownOpen  = S.dropdownItems.length>0;
    } else { S.dropdownOpen=false; S.dropdownItems=[]; }
    refreshDropdown(input);
  });

  input.addEventListener('keydown', e=>{
    if (e.key==='ArrowDown') {
      e.preventDefault();
      S.activeIdx=Math.min(S.activeIdx+1, S.dropdownItems.length-1);
      refreshDropdown(input);
    } else if (e.key==='ArrowUp') {
      e.preventDefault();
      S.activeIdx=Math.max(S.activeIdx-1,-1);
      refreshDropdown(input);
    } else if (e.key==='Enter') {
      e.preventDefault();
      if (S.activeIdx>=0 && S.dropdownItems[S.activeIdx]) selectItem(S.dropdownItems[S.activeIdx],input);
      else doGuess(input.value);
    } else if (e.key==='Escape') {
      S.dropdownOpen=false; refreshDropdown(input);
    }
  });

  if (btn) btn.addEventListener('click', ()=>doGuess(input.value));

  document.addEventListener('pointerdown', e=>{
    if (!e.target.closest('.guess-input-wrapper')) {
      S.dropdownOpen=false;
      const dd=document.getElementById('ac-dd'); if(dd) dd.remove();
    }
  },{once:true});

  attachDropdown(input);
}

function refreshDropdown(input) {
  const w=input.parentElement;
  const ex=w.querySelector('#ac-dd'); if(ex) ex.remove();
  if (S.dropdownOpen) { w.insertAdjacentHTML('beforeend',dropdownHTML()); attachDropdown(input); }
  if (S.activeIdx>=0 && S.dropdownItems[S.activeIdx]) input.value=S.dropdownItems[S.activeIdx].display;
}

function attachDropdown(input) {
  const dd=document.getElementById('ac-dd'); if(!dd) return;
  dd.querySelectorAll('.autocomplete-item').forEach(item=>{
    item.addEventListener('pointerdown', e=>{
      e.preventDefault();
      const f=S.allPokemon.find(p=>p.name===item.dataset.name);
      if(f) selectItem(f,input);
    });
  });
}

function selectItem(item,input) {
  S.autoValue=item.display; input.value=item.display;
  S.dropdownOpen=false; S.activeIdx=-1;
  const dd=document.getElementById('ac-dd'); if(dd) dd.remove();
}

function doGuess(value) {
  const errEl=document.getElementById('guess-error');
  const input=document.getElementById('guess-input');
  const norm=value.trim().toLowerCase().replace(/\s+/g,'-');
  if (!norm) { if(errEl) errEl.textContent='Enter a Pokemon name!'; return; }
  if (!S.allPokemon.some(p=>p.name===norm)) {
    if(errEl) errEl.textContent='NOT A VALID POKEMON!';
    if(input){ input.classList.add('shake'); setTimeout(()=>input.classList.remove('shake'),500); }
    return;
  }
  if (S.guesses.includes(norm) && norm!==S.answer) {
    if(errEl) errEl.textContent='ALREADY TRIED THAT!'; return;
  }
  S.autoValue=''; S.dropdownOpen=false;
  const result=submitGuess(norm);
  if (result==='win') { render(); showConfetti(); }
  else render();
}

// ════════════════════════════════════════════
// MODAL
// ════════════════════════════════════════════

function showModal() {
  const {won,answer,hints,flavorText}=S;
  const overlay=document.createElement('div');
  overlay.className='modal-overlay';
  overlay.innerHTML=`
    <div class="modal ${won?'win':'lose'}">
      <div class="modal-title">${won?'YOU WIN!':'GAME OVER!'}</div>
      <div class="modal-pokemon-name">${capAll(answer)}</div>
      <img class="modal-sprite" src="${hints.sprite}" alt="${answer}"/>
      <div class="modal-types">${hints.types.map(t=>typeBadge(t)).join('')}</div>
      <div class="modal-meta">${hints.generation} / ${hints.region}<br>${hints.evoStage}</div>
      <div class="modal-flavor">"${flavorText}"</div>
      <button class="play-again-btn" id="play-again">PLAY AGAIN</button>
    </div>`;
  document.body.appendChild(overlay);
  document.getElementById('play-again').addEventListener('click',()=>location.reload());
  setTimeout(()=>{
    const img=document.getElementById('silhouette-img');
    if(img) img.classList.add('revealed');
  },200);
}

// ════════════════════════════════════════════
// CONFETTI
// ════════════════════════════════════════════

function showConfetti() {
  const el=document.createElement('div'); el.className='confetti-container';
  document.body.appendChild(el);
  const colors=['#FFD700','#CC0000','#3b9fdf','#4CAF50','#FF9800','#9C27B0','#fff'];
  for (let i=0;i<90;i++){
    const p=document.createElement('div'); p.className='confetti-piece';
    const size=6+Math.random()*9;
    p.style.cssText=`
      left:${Math.random()*100}%;top:-12px;
      width:${size}px;height:${size*(Math.random()>.5?1:.4)}px;
      background:${colors[Math.floor(Math.random()*colors.length)]};
      border-radius:${Math.random()>.4?'50%':'2px'};
      animation-duration:${1.8+Math.random()*2.2}s;
      animation-delay:${Math.random()*.7}s;`;
    el.appendChild(p);
  }
  setTimeout(()=>el.remove(),5000);
}

// ════════════════════════════════════════════
// INIT
// ════════════════════════════════════════════
loadGame();
</script>
</body>
</html>
