# PROJECT CONTEXT — "Flag Battle Royale" (Project Helter)

> File ini dibuat supaya **Claude Code / model lain** bisa langsung paham konteks
> dan progress project tanpa harus baca ulang seluruh percakapan.
> File HTML final: **`index.html` di folder ini / repo ini.**

## RINGKASAN SINGKAT
Web game **battle royale simulator** berbasis HTML5 Canvas tunggal (no external libs).
Ada **150 bendera negara** (PNG asli, embedded base64) yang diacak & diadu di dalam
lingkaran arena. Ada **celah (gap)** di dinding yang nyapu bola — kena celah = tereliminasi.
Physics: pantulan elastis (bola-dinding & bola-bola). Suara disintesis via Web Audio API
(no file eksternal). Theme: galaksi / luar angkasa + arena kayak bangunan pesawat luar angkasa.

Target platform: browser (HP & PC). Semua teks UI dalam **BAHASA INGGRIS**.

## CARA JALAN
- Klik tombol **rocket icon** (bukan tulisan, SVG rocket) di bawah bulatan arena, tengah,
  diapit 2 turret laser dekoratif kiri-kanan. 1 tombol = Start & Stop (toggle via class
  `isRunning`; nyala = flame thrust animasi di bawah rocket).
- ⚙️ Settings **disembunyikan** — trigger-nya klik kata **"ROYALE"** di judul header
  (`#royaleBtn`, ada underline putus-putus halus sebagai hint).
- Settings berisi: gap size, gap speed, gravity, ball speed, simulation mode
  (Normal / Accelerated), sound toggle.
- File standalone: buka `index.html` di browser mana aja.

## LIVE / GITHUB
- GitHub user: **helterskeltersz**
- Repo: **Project-Helter** → https://github.com/helterskeltersz/Project-Helter
- Live (GitHub Pages): **https://helterskeltersz.github.io/Project-Helter/**
- Repo public, Pages dari branch `master` root `/`.
- Catatan: handle sering salah diketik jadi "helpskeltersz" — yang benar **helterskeltersz**.

## STRUCTUR TURNAMEN (3 FASE, kontinu)
1. **QUALIFICATION** (150 → 75), **4 celah**
   - Semua 150 negara diadu sekaligus. YANG BERTAHAN PALING AKHIR (last survivor) per round = pemenang.
   - 75 round, tiap round ambil 1 pemenang → masuk array `qualified` (grid "Qualified" 15×5 di bawah).
   - Setiap pemenang: popup bendera besar + nama + label "QUALIFIED", **pause 3 detik**, lalu round berikutnya.
   - Gap qualification selalu terbuka + ada "breathe" (pulse), dan di mode Accelerated gap melebar (85°).
2. **KNOCKOUT** (75 → 3), **2 celah, staggered** (⚠️ diubah dari 1 celah/1.3s)
   - KONTINU (bukan session restart): game jalan terus sampai sisa 3.
   - **2 gap independen**, tiap gap siklus 15 detik: **buka 10 detik, tutup 5 detik**.
     Gap kedua di-offset 5 detik dari gap pertama → hampir selalu ada MINIMAL 1 gap
     kebuka, kadang 2 gap kebuka bareng (overlap 5 detik tiap siklus). Lihat
     `gapOpenState(idx)` — knockout branch pakai `period=15,dur=10,offset=idx*5`.
   - Tiap bendera kena gap → **TIDAK langsung mati**. Bola terbang keluar radial
     (escape), hover di `HOVER_R=R+20` (ring merah berdenyut di sekitarnya), setelah
     `ESCAPE_DELAY=0.45s` ditembak laser merah dari turret terdekat (`fireLaser`,
     kiri/kanan tergantung posisi x), BARU final mati + masuk `elimLog` + trigger
     popup notif **2 detik** (bendera + nama + "DISQUALIFIED"), **non-blocking**.
   - Sampai sisa 3 → popup "FINAL 3" → masuk PODIUM.
   - Mekanik escape+laser ini KHUSUS knockout & podium (qualify tetap instant mati,
     gak ada notif DISQUALIFIED di sana).
3. **PODIUM** (3 → 1), **1 celah**
   - KONTINU sampai sisa 1 (juara). Celah terbuka **tiap 10 detik** (buka ±2s).
   - Tiap gugur → notif 2 detik. Juara = last survivor; yang gugur duluan = juara 2, berikutnya = juara 3.
   - Podium banner + nama negara + confetti + fanfare.
- **TIDAK ada babak Final terpisah** (dihapus). Alur: Qualify → Knockout → Podium.

## MEKANIK / LOGIKA KUNCI (di `index.html`)
State (var global): `phase`, `allN`, `qPool`, `qualified`, `knockoutPool`, `sessionPool`(unused),
`survivors3`, `podium`, `podiumOrder`(unused), `balls`, `gaps`, `gapA`, `simT`, `wallClock`,
`running`, `elimLog`, `lastElimNotified`, `particles`, `sparks`, `lasers`, `boost`, `mode`, `soundOn`.
Per-ball flag tambahan (di object ball, bukan global): `escaping`, `escT`, `laserFired`.

- **Gap buka/tutup siklik (PER-GAP, bukan shared lagi)**: `gapOpenState(idx)` → `{open,w}` per index gap.
  Qualify: selalu open + breathe pulse. Podium: 1 gap, buka 2s / 10s. Knockout: 2 gap staggered
  (lihat bagian STRUKTUR TURNAMEN di atas). `checkGapHit(angle,gapStates)` dipakai di `step()`,
  dan `drawShipArena()` bikin `gapStates` sendiri tiap frame buat gambar hatch per-gap.
  Fungsi lama `effGapW()`/`gapOpenNow()`/`inGap()` **sudah dihapus**, diganti skema ini.
- **Speed per fase**: qualify pakai tail speed-up (makin sedikit sisa makin cepat) + boost di Accelerated;
  knockout & podium jalan **2 step/frame (2x)** TANPA tail speed-up (user minta "gak usah edit speed, cukup 2x").
- **Collision ball-vs-ball pakai spatial grid** (round 7, bukan O(n²) polos lagi): di `step()`, sebelum
  loop tabrakan, semua bola alive&non-escaping di-bucket ke `Map` grid (`CELL=80`px per cell, key
  `"cx_cy"`). Tiap bola cuma dicek lawan bola-bola di cell-nya sendiri + 8 tetangga (3×3), bukan SEMUA
  bola lain. Turun ~74% jumlah pair-check di kondisi terpadat (150 bola). Fisika/rumus resolusi
  tabrakannya SAMA PERSIS — ini murni optimasi algoritma milih pasangan mana yg layak dicek.
- **Escape + laser turret — KNOCKOUT/PODIUM ONLY** (round 5 sempet bikin ini berlaku di qualify juga,
  tapi di-REVERT lagi di round 6 karena 150 bola × 4 gap kebuka = kebanyakan bola escaping bareng →
  lag parah di HP. Sekarang balik ke behavior awal): bola yang kena gap DI KNOCKOUT/PODIUM gak langsung
  `alive=false`. Ditandai `escaping=true`, terbang radial keluar sampai `HOVER_R`, hover situ sambil
  `escT` nambah tiap frame. Begitu `escT>=ESCAPE_DELAY(0.45s)` → `fireLaser(b)` dipanggil: gambar garis
  laser merah dari turret terdekat (kiri/kanan by `b.x<CX`, cuma 2 turret — lihat THEME VISUAL) ke posisi
  bola, spark merah, baru bola `alive=false` + masuk `elimLog`. Di QUALIFY, eliminasi tetap INSTAN
  (`b.alive=false;elimLog.push(b.n);` langsung, gak ada escape/hover) — percabangan
  `if(phase==='knockout'||phase==='podium'){...escape...} else {...instant...}` di `step()` itu PENTING,
  JANGAN dihapus lagi tanpa diminta eksplisit (udah pernah dicoba, hasilnya lag). `notifyFlag
  ('DISQUALIFIED')` otomatis muncul next tick karena baca `elimLog.length>lastElimNotified` (cuma
  kepasang di knockout/podium — qualify tetap gak nampilin DISQUALIFIED popup per bendera, cuma popup
  QUALIFIED buat pemenang tiap round). Selama escaping, bola dikecualikan dari collision & wall-bounce
  loop (biar gak mantul aneh pas udah "keluar"). Setiap kali bola KENA GAP (fase manapun, instan atau
  escape), `playEscape()` dipanggil — suara "whoosh" naik pitch, terpisah dari `playZap` (laser, baru
  bunyi belakangan pas knockout/podium finalize).
- **POPUP FUNGSI**:
  - `showPopup(items,label,onDone,forcePause)` → popup pause (milestone QUALIFIED/FINAL 3/CHAMPION).
  - `showPhaseIntro(key,onDone)` → popup 5 detik pas ganti fase (qualify/knockout/podium), isi
    kriteria fase itu dari `PHASE_INFO`. Blocking (`popupBusy=true`) sama kayak showPopup.
  - `notifyFlag(n,label)` → notif 2 detik NON-blocking (untuk elimasi knockout & podium, dipanggil
    otomatis setelah laser finalize, BUKAN pas bola mulai escaping).
- **Efek**:
  - `spawnSparks(x,y,n,color?)` + `renderEffects` → percikan rollerblade (tabrakan bola, warna random
    kuning/putih) DAN percikan merah (laser hit, `color='#ff3b3b'`).
  - `lasers[]` array (`{x1,y1,x2,y2,life,max}`) → garis laser merah glowing, fade out, digambar di
    `renderEffects` (canvas efek, sama kayak sparks/confetti).
  - `launchConfetti` → confetti untuk perayaan juara. `fireworks[]` + `spawnFirework`/`launchFireworks`
    → ledakan partikel radial (6 burst staggered ~400ms, warna random per burst) + `playBoom` tiap burst,
    dipanggil bareng `launchConfetti` di `showPodium()`.
  - Suara: `beep`, `playBounce` (wall), `playScrape` (tabrakan bola-bola, sawtooth — beda dari playBounce),
    `playEscape` (bola kena gap/keluar, whoosh naik pitch, SEMUA fase), `playQualify`, `playFanfare`,
    `playPhase`, `playZap` (laser finalize, cuma knockout/podium), `playBoom` (firework).
  - **`withAudio(fn)`** (round 6): helper wrapper dipakai semua fungsi suara di atas. Fix bug "sound
    hilang" round sebelumnya — `beep()` cuma manggil `actx.resume()` tanpa NUNGGU promise-nya selesai
    sebelum jadwalin oscillator, jadi suaranya sering ke-drop diam-diam kalau context lagi suspended
    (misal abis `speechSynthesis` ngomong). Sekarang: kalau `actx.state==='suspended'`, tunggu
    `.resume().then(fn)` baru jalanin suaranya; kalau udah `running`, langsung jalanin `fn()`.
  - **Voice-over** via Web Speech API (`speak(text)`): SEJAK ROUND 5 — suara normal (rate 1, pitch 1),
    prefer voice PEREMPUAN (`pickVoice()` nyari nama match `/Samantha|Zira|Karen|Victoria|Moira|Tessa|
    Female|Google US English|Google UK English Female/i`), TANPA chirp radio/efek luar angkasa (dihapus
    total, sebelumnya ada tapi user minta dihapus). `unlockSpeech()` dipanggil sinkron di dalam
    `startBtn.onclick` (ngomong utterance kosong volume 0.01) buat "unlock" TTS di browser mobile yang
    strict soal user-gesture (iOS Safari dkk) — kalau ini gak ada, `speak()` yang dipanggil belakangan
    dari dalam `step()` (bukan langsung dari klik user) sering ke-block/silent di HP. Ikut toggle
    `soundOn`. Dipanggil di 3 titik: `showPhaseIntro`, transisi knockout→podium, saat champion ditentukan.
- **Ticker "Qualified" & "Eliminated"**: SEJAK ROUND 5, bukan CSS `animation` lagi — sekarang JS-driven
  continuous transform biar GAK PERNAH restart/lompat pas ada entry baru masuk (request eksplisit user).
  Tiap row (`qRow0-2`/`eRow0-2`) punya state di `tickerRows[rowId] = {pos, expandedWidth, dir}` — `pos`
  itu MONOTONIC (naik terus, gak pernah di-reset oleh `renderTicker()`), diambil modulo
  `expandedWidth` tiap frame di `stepTickers(dt)` (dipanggil dari `loop()`) buat dapetin posisi transform.
  `renderTicker()` cuma update KONTEN (innerHTML) & `expandedWidth` — list-nya diduplikasi PERSIS 2x
  aja (bukan di-pad/di-repeat sampai lebar container — itu bug round 5 yang bikin 1-2 bendera doang
  ke-repeat belasan/puluhan kali buat "isi" kotak, kelihatan kayak bendera dobel-dobel DAN bikin lag
  parah karena ratusan `<img>` node per row. Round 6: dibalikin simpel, list cuma 2x lipat, titik).
  `pos` gak disentuh sama sekali pas render → makanya nambah bendera baru gak bikin yang lagi jalan
  "lompat"/restart. `renderG()`/`renderLog()`
  masih ada guard `lastQualifiedRender`/`lastElimRender` (skip render kalau `.length` gak berubah, biar
  gak measure DOM tiap tick sia-sia — TAPI ini cuma optimasi render konten, animasinya sendiri udah gak
  butuh guard ini lagi karena `stepTickers` jalan independen tiap frame).

## PERUBAHAN DESAIN TERAKHIR (round 7, sama hari 2026-08-13)
User masih lapor lag + no-sound abis round 6. Didiagnosis bareng (bukan asal-tebak lagi):
- **Lag**: dikonfirmasi user CUMA pas battle jalan (bukan idle) → nunjuk ke physics, bukan CSS
  background. Root cause: collision ball-vs-ball di `step()` itu O(n²) — 150 bola = 11.175 pasangan
  dicek TIAP FRAME, gak ada spatial optimization dari awal. Fix: **spatial grid broad-phase**
  (`CELL=80`, ukuran cell nutupin diameter bola terbesar × 2 biar pasangan yg collide dijamin ke-cover
  di cell yg sama/sebelahan). Bola di-bucket ke grid (`Map` key `"cx_cy"`), tiap bola cuma ngecek 3×3
  cell tetangga, bukan SEMUA bola lain. Terukur: 11.175 → 2.902 pair-check (turun 74%) di kondisi
  terpadat (150 bola bareng, awal qualify). Physics/hasil collision-nya SAMA PERSIS (cuma optimasi
  ALGORITMA milih pasangan mana yg dicek, bukan ubah rumus resolusi tabrakannya).
- **Sound**: user pake iPhone dengan **saklar Silent/Ring fisik posisi ON**. Ini expose bug/behavior
  khas iOS Safari — Web Audio API (dipake buat SEMUA `beep()`-based sound effect) di-treat sebagai
  kategori audio "ambient" yg PATUH sama saklar silent, sedangkan `SpeechSynthesis` (voice-over) TIDAK
  kena efek itu (makanya voice tetep kedengeran tapi efek suara enggak — persis sesuai laporan user).
  Fix standar: elemen `<audio id="silentUnlock" loop playsinline>` isinya WAV senyap (silent PCM data,
  generated via Python, base64 di HTML), di-`.play()` dari `unlockIOSAudio()` yang dipanggil di dalam
  `startBtn.onclick` (harus sinkron dalem user-gesture). Begitu elemen `<audio>` ini "playing", Safari
  switch kategori audio session halaman ke "playback" yg NGABAIIN saklar silent — otomatis bikin Web
  Audio ikut kedengeran juga. **CATATAN PENTING buat sesi depan**: kalau ada laporan "sound gak ada
  padahal voice over ada" lagi, ini kemungkinan besar user iPhone dengan silent switch ON — cek itu
  DULU sebelum ngoprek kode lagi, dan pastiin elemen `#silentUnlock` masih ada & ke-play di startBtn.

## PERUBAHAN DESAIN TERAKHIR (round 6, sama hari 2026-08-13)
- **User report: lag parah** setelah round 5. Root cause ganda: (1) escape+laser aktif di qualify bikin
  banyak bola escaping bareng (150 bola × 4 gap), (2) ticker ngerepeat 1-2 bendera sampai puluhan kali
  buat "isi" kotak → ratusan `<img>` DOM node + reflow tiap render. Kedua-duanya di-REVERT/fix.
- Laser turret balik ke 2 (bawah aja), mekanik escape+laser balik cuma di knockout/podium, qualify balik
  instant. **User report: bendera dobel-dobel** di tabel Qualified pas negara baru lolos — ternyata ini
  gejala yang sama persis dengan bug lag di atas (repeat-padding). Fix: `renderTicker` sekarang cuma
  duplikasi list PERSIS 2x (teknik seamless-loop standar), gak lagi di-pad sampai lebar container.
- **User report: sound effect masih ilang.** Root cause: `beep()` manggil `actx.resume()` tapi gak
  nunggu promise-nya selesai sebelum jadwalin suara → suara abis TTS ngomong sering ke-drop diam-diam.
  Fix pakai `withAudio(fn)` helper yang nunggu `.resume().then(fn)` kalau context lagi suspended.
  **Round 7**: itu ternyata belum cukup buat iPhone dengan saklar Silent ON — lihat elemen
  `<audio id="silentUnlock">` (WAV senyap) + `unlockIOSAudio()` yang dipanggil di `startBtn.onclick`,
  detail lengkap di changelog round 7 di bawah. JANGAN dihapus, itu yang bikin efek suara kedengeran
  di iPhone mode silent.
  Ditambah `playEscape()` — suara baru pas bola KENA GAP (whoosh naik pitch), terpisah dari `playZap`
  (laser finalize) yang udah ada, biar user denger feedback pas bendera "keluar" bukan cuma pas
  "ketembak".

## PERUBAHAN DESAIN TERAKHIR (round 5, sama hari 2026-08-13)
- Voice-over dirombak TOTAL: hapus efek radio/chirp & pitch/rate dramatis, ganti suara PEREMPUAN normal
  (`pickVoice` prefer nama voice perempuan), tambah `unlockSpeech()` di klik Start biar reliable diputer
  di HP (browser strict soal TTS harus nempel user-gesture).
- Ticker Qualified/Eliminated: gak lagi restart animasi pas ada entry baru — full rewrite jadi JS-driven
  continuous transform (`tickerRows`/`stepTickers`). Lihat detail lengkap di bagian MEKANIK.
- HUD kecil di pojok canvas (alive/total/nama-fase) dihapus, user bilang bikin berantakan.
- Laser turret jadi 4 (nambah 2 di atas ring), dan mekanik escape+laser sekarang aktif dari fase
  QUALIFICATION juga (sebelumnya cuma knockout/podium).

## PERUBAHAN DESAIN TERAKHIR (round 4, sama hari 2026-08-13)
- **Fix bug suara hilang**: `beep()` sekarang cek `actx.state==='suspended'` dan `resume()` tiap dipanggil,
  plus safety-net `loop()` ngecek tiap ~1 detik. Root cause: browser HP nge-suspend shared AudioContext
  pas `speechSynthesis` ngambil "audio focus", dan sebelumnya gak pernah di-resume lagi setelahnya.
- **Voice-over dirombak**: tiap `PHASE_INFO[key]` sekarang punya field `voice` terpisah dari `desc`
  (teks di popup tetap informatif/singkat, teks yg diucapkan lebih dramatis ala "space commander").
  `pickVoice()` nyoba cari voice yg namanya match `/Daniel|Google UK English Male|Male|David|Fred/i`
  dulu sebelum fallback ke voice English biasa — biar lebih "gravitas". Pitch diturunin ke 0.78, rate 0.92.
- **Fix ticker "nunggu"**: `renderG()`/`renderLog()` sebelumnya manggil `renderTicker()` (yg restart CSS
  animation) SETIAP TICK step() — bikin sweep animation ke-reset 60x/detik jadi keliatan macet. Sekarang
  ada guard `lastQualifiedRender`/`lastElimRender` (cuma re-render kalau `.length` berubah), di-reset ke
  `-1` di `initTournament()` & `enterPhase()` (biar tetep bersih tiap tournament/round baru).
- **Layout dikompakin biar 1 layar gak scroll**: tombol start 72px→50px, ticker row 26px→20px, semua
  margin/padding diperketat, DAN `#wrap` (arena canvas) dibatasi `max-width` via media query berdasar
  `max-height` viewport (780px/680px/600px breakpoints → 330px/270px/220px) biar canvas ikut ngecil di
  layar HP pendek. Diverifikasi lolos no-scroll di iPhone SE (667px), 700px, iPhone 14 (844px), Pixel (915px).
- **Logo wordmark**: h1 sekarang pakai font **Orbitron** (Google Fonts, via `<link>` di `<head>` — SATU-
  SATUNYA dependency eksternal di file ini, graceful fallback ke system font kalau offline/gagal load)
  + gradient text (`background-clip:text`) amber→cyan + `drop-shadow` berlapis. Span `#royaleBtn` (trigger
  settings) masih nempel di dalam teks yang sama.
- **Champion popup diperbesar + efek "present"**: `showPopup()` sekarang punya branch khusus kalau
  `label` mengandung "CHAMPION" — flag jadi 150×100 (naik dari 88×59), bounce-in animasi (`champPop`),
  5 sparkle bertebaran & berkedip (`champSpark`/`champSparkle`) di sekitar bendera. Popup QUALIFIED biasa
  gak kena efek ini (size lama tetap).

## THEME VISUAL (GALAXY + PESAWAT LUAR ANGKASA)
- **Background galaksi**: DUA layer sekarang —
  1. `body` CSS full-page starfield (radial-gradient bintang statis + nebula ungu/biru/pink + base
     gradient gelap) biar seluruh halaman (termasuk area ticker & tombol start di luar canvas) nyambung,
     gak "kepotong" kayak sebelumnya.
  2. `drawGalaxy()` di canvas — versi animasi (bintang berkelip, nebula) khusus di dalam arena 640×640.
- **Arena = pesawat luar angkasa** (`drawShipArena`): ring hull metalik gelap, glint tepi **cyan**,
  energy rail cyan berdenyut di tepi dalam, lampu penerbangan (cockpit blips) cyan kelap-kelip.
- **CELAH (gap)**: bentuk kayak pintu hatch terbuka dengan glow + lampu tepi. **WAJIB AMBER/KUNING TERAKHIR**:
  lampu celah `rgba(255,225,80)` / glow `rgba(255,204,60)`. JANGAN ubah kembali ke cyan tanpa diminta.
- **Laser turret**: 2 aja — `TURRET_L`/`TURRET_R`, di bawah ring, digambar via `drawTurrets`.
  (Round 5 sempet nambah 2 lagi di atas jadi 4 + aktifin dari qualify, tapi di-REVERT round 6 karena
  bikin lag — lihat MEKANIK di atas. JANGAN diulang lagi tanpa diminta eksplisit.) `fireLaser()` milih
  kiri/kanan simpel berdasar `b.x<CX`. Versi dekoratif HTML di samping tombol start tetap gak ada
  (dihapus round 3).
- **HUD overlay dihapus** (round 5): kotak kecil "🎯 alive/total | ⚡ nama-fase" yang dulu nempel di
  pojok kiri-atas canvas (`#hud`) udah gak ada — user minta dibersihin. `updateHUD()` sekarang cuma
  manggil `renderLog()`, gak ada lagi elemen `#alive`/`#total`/`#fase` di DOM.

## PERUBAHAN DESAIN TERAKHIR (round 3, sama hari 2026-08-13)
- Turret laser dekoratif di samping tombol start **dihapus** — cukup 2 turret (yang di canvas, kiri-kanan
  bawah ring).
- Background galaxy dijadikan full-page (CSS di `body`), bukan cuma di dalam canvas — biar gak "kepotong"
  di area tombol start / ticker.
- Ticker Qualified/Eliminated diperbaiki: sweep sekarang selalu FULL lebar box (lihat penjelasan
  `renderTicker` di atas), bukan cuma separo pas item dikit.
- Podium/champion sekarang ada fireworks (`launchFireworks`) selain confetti — lebih meriah.
- Tabrakan bola-bola sekarang bunyi beda (`playScrape`, sawtooth) dari tabrakan ke dinding (`playBounce`,
  triangle).
- Voice-over ditambahkan (Web Speech API) — baca rules tiap ganti fase, baca 3 negara yang lolos ke
  podium, baca nama juara. Dibungkus radio-chirp biar berasa transmisi luar angkasa. Lihat detail di
  bagian MEKANIK di atas.

## PERUBAHAN DESAIN TERAKHIR (round 2 — 2026-08-13)
- Header jadi teks polos "THE FLAG BATTLE ROYALE", tanpa logo/subtext. Kata "ROYALE"
  jadi trigger tersembunyi buat buka Settings (gear icon dihapus dari UI).
- Tabel fase (Qualification/Knockout/Podium chips) dihapus, diganti popup 5 detik
  (`showPhaseIntro`) tiap pergantian babak.
- Tabel "Qualified" dipindah ke atas arena, jadi ticker scroll 3 baris (bukan grid statis).
- Tombol Start/Stop jadi 1 icon rocket SVG (bukan teks/emoji) di bawah arena, dengan
  animasi flame thrust pas jalan. Diapit 2 turret icon dekoratif.
- Elimination Log didesain ulang mirip ticker Qualified (3 baris scroll), tapi bendera
  jadi grayscale + border merah, posisi tetap di bawah (di bawah tombol start).
- **BARU**: mekanik escape+laser turret buat eliminasi knockout/podium — bola gak
  langsung mati, terbang keluar dulu, ditembak laser merah dari turret, baru
  disqualified. Lihat bagian MEKANIK di atas.
- **BARU**: knockout dari 1 gap (buka 1.3s/5s) jadi 2 gap staggered (masing-masing
  buka 10s/tutup 5s, offset 5s) — direquest eksplisit, sudah dikonfirmasi ke user
  lewat AskUserQuestion karena angka awalnya ambigu.
- Sebelumnya: hapus efek "kocok" (shake) di knockout, knockout kontinu + notif 2 detik,
  theme galaksi + arena pesawat luar angkasa, lampu celah kuning/amber (masih berlaku,
  JANGAN diubah ke cyan tanpa diminta).

## PESAN DALAM MEMORY HARMAS (dari percakapan)
- Wen orang Indonesia, pakai "gue/lo", suka komunikasi langsung, prefer deliverable file.
- **JANGAN langsung coding saat user cuma NANYA pendapat ("bisa ngasih X?")** — tunggu instruksi
  eksplisit ("start/kerjain/gas") dulu. Konfirmasi sebelum eksekusi.
- Preference: REAL flag PNG (bukan emoji), bendera bentuk persegi, UI teks bahasa Inggris.
- Game build: "Flags Battle Royale" → Project Helter.

## CARA UPDATE KE GITHUB (kalau perlu)
```
cd /root/flag-battle
cp /root/battle_royale/index.html index.html
git add -A && git commit -m "update" && git push
```
Pages otomatis rebuild dari branch `master` → https://helterskeltersz.github.io/Project-Helter/

## LATEST HTML
- File final: `/root/battle_royale/index.html` (≈118 KB, berisi 150 flag PNG embedded + semua efisien).
- **Belum di-push ke GitHub** per akhir sesi round 2 ini — masih perlu `cp` ke `/root/flag-battle/`
  lalu commit+push kalau user sudah oke sama hasilnya (lihat CARA UPDATE KE GITHUB).
- Backup versi-versi sebelumnya ada di `/root/battle_royale/backups/`.
