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
- Klik **🚀 Start** (di atas, samping tombol ⚙️ Settings, centered).
- ⚙️ Settings (hidden, klik gear): gap size, gap speed, gravity, ball speed, simulation mode
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
2. **KNOCKOUT** (75 → 3), **1 celah**
   - KONTINU (bukan session restart): game jalan terus sampai sisa 3.
   - Celah cuma **1**, **terbuka tiap 5 detik** (buka ±1.3s, sisanya tertutup). Saat tertutup GAK ada eliminasi.
   - Tiap bendera gugur → notif popup **2 detik** (bendera + nama + "DISQUALIFIED"), **non-blocking** (game tetap jalan).
   - Sampai sisa 3 → popup "FINAL 3" → masuk PODIUM.
3. **PODIUM** (3 → 1), **1 celah**
   - KONTINU sampai sisa 1 (juara). Celah terbuka **tiap 10 detik** (buka ±2s).
   - Tiap gugur → notif 2 detik. Juara = last survivor; yang gugur duluan = juara 2, berikutnya = juara 3.
   - Podium banner + nama negara + confetti + fanfare.
- **TIDAK ada babak Final terpisah** (dihapus). Alur: Qualify → Knockout → Podium.

## MEKANIK / LOGIKA KUNCI (di `index.html`)
State (var global): `phase`, `allN`, `qPool`, `qualified`, `knockoutPool`, `sessionPool`(unused),
`survivors3`, `podium`, `podiumOrder`(unused), `balls`, `gaps`, `gapA`, `simT`, `wallClock`,
`running`, `elimLog`, `lastElimNotified`, `particles`, `sparks`, `boost`, `mode`, `soundOn`.

- **Gap buka/tutup siklik**: `gapOpenNow()` — qualify selalu open; knockout period 5s; podium period 10s.
  `effGapW()` mengembalikan 0 saat tertutup (== tidak ada eliminasi), gapW*pulse saat terbuka.
- **Speed per fase**: qualify pakai tail speed-up (makin sedikit sisa makin cepat) + boost di Accelerated;
  knockout & podium jalan **2 step/frame (2x)** TANPA tail speed-up (user minta "gak usah edit speed, cukup 2x").
- **POPUP FUNGSI**:
  - `showPopup(items,label,onDone,forcePause)` → popup pause (milestone QUALIFIED/FINAL 3/CHAMPION).
  - `notifyFlag(n,label)` → notif 2 detik NON-blocking (untuk elimasi knockout & podium).
- **Efek**:
  - `spawnSparks` + `renderEffects` → **percikan rollerblade** saat bendera saling tabrakan (di semua fase; podium lebih banyak).
  - `launchConfetti` → confetti untuk perayaan juara.
  - Suara: `beep`, `playBounce`, `playElim`, `playQualify`, `playFanfare`, `playPhase`.

## THEME VISUAL (GALAXY + PESAWAT LUAR ANGKASA)
- **Background galaksi** (`drawGalaxy`): gradient deep space, nebula ungu/biru/pink, ~135 bintang berkelip.
- **Arena = pesawat luar angkasa** (`drawShipArena`): ring hull metalik gelap, glint tepi **cyan**,
  energy rail cyan berdenyut di tepi dalam, lampu penerbangan (cockpit blips) cyan kelap-kelip.
- **CELAH (gap)**: bentuk kayak pintu hatch terbuka dengan glow + lampu tepi. **WAJIB AMBER/KUNING TERAKHIR**:
  lampu celah `rgba(255,225,80)` / glow `rgba(255,204,60)`. (Baru diubah dari cyan → kuning.)
  JANGAN ubah kembali ke cyan tanpa diminta.

## PERUBAHAN DESAIN TERAKHIR (sesi terbaru)
- Hapus efek "kocok" (shake) di knockout — jelek, user minta hapus.
- Knockout jadi kontinu (gak pause/restart tiap eliminasi) + notif disqualified 2 detik.
- Theme galaksi + arena pesawat luar angkasa (bukan api matahari).
- Lampu celah → kuning/amber. ✅ (baru)

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
- File final: `/root/battle_royale/index.html` (≈112 KB, berisi 150 flag PNG embedded + semua efisien).
- Juga tersimpan di repo `/root/flag-battle/index.html` dan GitHub.
