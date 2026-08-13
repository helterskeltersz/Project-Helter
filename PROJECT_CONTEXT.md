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
- **Escape + laser turret (knockout/podium only)**: bola yang kena gap gak langsung `alive=false`.
  Ditandai `escaping=true`, terbang radial keluar sampai `HOVER_R`, hover situ sambil `escT` nambah
  tiap frame. Begitu `escT>=ESCAPE_DELAY(0.45s)` → `fireLaser(b)` dipanggil: gambar garis laser merah
  dari turret terdekat (`TURRET_L`/`TURRET_R`, digambar di canvas via `drawTurrets`, juga ada versi
  dekoratif HTML di `.turretIcon` sebelah tombol start) ke posisi bola, spark merah, baru bola
  `alive=false` + masuk `elimLog`. `notifyFlag('DISQUALIFIED')` otomatis muncul next tick karena
  baca `elimLog.length>lastElimNotified` (logic ini gak diubah). Selama escaping, bola dikecualikan
  dari collision & wall-bounce loop (biar gak mantul aneh pas udah "keluar").
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
    `playElim`, `playQualify`, `playFanfare`, `playPhase`, `playZap` (laser), `playBoom` (firework).
  - **Voice-over** via Web Speech API (`speak(text)`/`radioAnnounce(text)`): baca teks pakai
    `SpeechSynthesisUtterance` (pitch 0.82, rate 0.95), dibungkus chirp radio (`beep` dua nada) sebelum
    mulai & sesudah selesai (`u.onend`) biar berasa "transmisi radio luar angkasa". Ikut toggle `soundOn`
    yang sama dengan efek suara lain (gak ada toggle terpisah). Dipanggil di 3 titik: `showPhaseIntro`
    (baca judul+rules tiap ganti fase), transisi knockout→podium (baca 3 negara yang lolos ke standing
    round), dan saat champion ditentukan (baca nama juara). Availability API di-cek (`'speechSynthesis'
    in window`) jadi aman kalau browser gak support.
- **Ticker "Qualified" & "Eliminated"**: keduanya pakai fungsi shared `renderTicker(list,prefix,countId)`
  → 3 baris sweep marquee (`qRow0-2` / `eRow0-2`), animasi CSS `qsweep` pakai custom properties
  `--sx`/`--ex` yang di-hitung JS per-render dari `container.clientWidth` & `row.scrollWidth` — jadi
  SELALU nyapu full lebar box (dari tepi kanan container sampai lewat tepi kiri konten atau sebaliknya),
  gak kepotong walau kontennya dikit (bug lama: dupe-content+translateX(-50%) cuma jalan separo box
  kalau konten pendek — udah diperbaiki). `renderG()`=qualified ticker (di atas arena), `renderLog()`=
  eliminated ticker (di bawah tombol start, border merah + `.ticker.elim img{filter:grayscale(1)}`).

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
- **Laser turret**: HANYA 2, digambar di canvas (`drawTurrets`/`TURRET_L`/`TURRET_R`, dekat bawah ring).
  Versi dekoratif HTML di samping tombol start **udah dihapus** (user minta cukup 2 turret aja).

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
