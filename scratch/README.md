# Scratch — Artifak Saluran Paip Sementara

Direktori ini mengandungi fail-fail perantaraan yang dihasilkan semasa saluran paip (pipeline) pemprosesan senarai perkataan. Fail-fail ini **bukan** hasil akhir — ia adalah rekod setiap peringkat pemprosesan untuk tujuan kebolehkesanan, diagnosis, dan pengauditan.

**Jangan gunakan fail dalam direktori ini untuk penjanaan frasa laluan.** Gunakan fail dalam [`lists/`](../lists/) atau [`db/`](../db/) sebagai gantinya.

## Konvensyen Penamaan Fail

Setiap senarai perkataan melalui 7 peringkat saluran paip. Fail-fail dinamakan mengikut peringkat:

| Awalan | Peringkat | Penerangan |
|--------|-----------|------------|
| `raw-` | 1 — Terjemahan | Terjemahan mentah bahasa Inggeris → Melayu (DeepSeek v4 Pro), satu perkataan setiap baris |
| `normalized-` | 2 — Penormalan | Huruf kecil, NFC, buang aksara bukan abjad |
| `deduped-` | 3 — Nyahduplikasi | Perkataan unik sahaja (sort -u) |
| `pruned-` | 4 — Pemangkasan | Selepas Schlinkert pruning (tidy -K) — boleh dinyahkod secara unik |
| `extended-` | 4b — Suplementasi | Gabungan perkataan terpangkas + perkataan tambahan (jika kurang sasaran) |
| `supplement-` | 4c — Perkataan tambahan | Perkataan Melayu tambahan yang dikurasi untuk mencapai sasaran |
| `final-pruned-` | 4d — Pemangkasan akhir | Selepas suplementasi + pemangkasan semula |
| `pruned-whittled-` | 4e — Whittled | Dipangkas tepat kepada saiz sasaran (tidy --whittle-to) |
| `final-whittled-` | 4e — Whittled akhir | Versi akhir selepas whittle |

Setiap fail mengandungi nama senarai sebagai akhiran:

| Akhiran | Senarai |
|---------|---------|
| `-kecil-alpha` | Alpha (1,296) |
| `-kecil-qwerty` | QWERTY (1,296) |
| `-sederhana` | Sederhana (8,192) |
| `-dadu` | Dadu (7,776) |
| `-besar` | Besar (17,576) |

Contoh: `deduped-besar.txt` = perkataan unik selepas penormalan untuk senarai besar.

## Saluran Paip 7 Peringkat

Skrip `pipeline.sh` mengautomasikan peringkat 2–7. Penggunaan:

```bash
./scratch/pipeline.sh lists/orchard-street-alpha.txt kecil-alpha
```

### Peringkat 1: Terjemahan (manual)
Setiap patah perkataan bahasa Inggeris diterjemahkan ke Bahasa Melayu Standard (ejaan DBP) menggunakan DeepSeek v4 Pro. Hasilnya disimpan dalam `raw-{nama}.txt`. Peringkat ini manual kerana terjemahan memerlukan pertimbangan manusia untuk mengekalkan kualiti.

### Peringkat 2: Penormalan (tidy)
```bash
tidy -l -z nfc --locale ms --remove-nonalphabetic
```
- Huruf kecil (lowercase)
- Normalisasi Unicode NFC
- Buang aksara bukan abjad (angka, tanda baca, ruang)
- Susun ikut abjad Melayu (`--locale ms`)

### Peringkat 3: Nyahduplikasi
```bash
sort -u
```
Pemetaan banyak-ke-satu dalam terjemahan (cth: "big" dan "large" kedua-duanya → *besar*) menghasilkan perkataan pendua. Peringkat ini mengeluarkan semua pendua.

### Peringkat 4: Pemangkasan Schlinkert (tidy -K)
```bash
tidy -K -l -z nfc --locale ms
```
Algoritma tamak berdasarkan siri Sardinas–Patterson yang memulihkan kebolehnilaian unik dengan mengalih keluar perkataan yang menyebabkan kekaburan. Fallback: `-P` atau `-S` jika `-K` membuang terlalu banyak perkataan.

### Peringkat 5: Pengesahan (tidy -AAAA)
```bash
tidy -AAAA -z nfc --locale ms
```
Mengesahkan kebolehnilaian unik (`Uniquely decodable? : true`), entropi, dan metrik lain.

### Peringkat 6: Format bersih
```bash
tidy -z nfc --locale ms -o lists/jalan-dewan-bahasa-{nama}.txt
```

### Peringkat 7: Format dadu (jika berkenaan)
```bash
tidy --dice 6 -z nfc --locale ms -o lists/jalan-dewan-bahasa-{nama}-dadu.txt
```

## Suplementasi

Apabila pemangkasan menghasilkan kurang daripada sasaran, perkataan Melayu tambahan ditambah dalam pusingan suplementasi:

- `supplement-{list}.txt` — perkataan tambahan untuk suatu senarai
- `supplement-besar-r{N}.txt` — pusingan suplementasi ke-N untuk senarai besar (7 pusingan)
- `extended-{list}.txt` — gabungan pruned + supplement sebelum pemangkasan semula
- `extended-besar-r{N}.txt` — versi lanjutan selepas pusingan N

Fallback pruning juga direkodkan:
- `pruned-{list}-p.txt` — hasil `tidy -P`
- `pruned-{list}-s.txt` — hasil `tidy -S`

## Laporan Audit

Fail `dbp-audit-*` mengandungi laporan audit ejaan DBP (Dewan Bahasa dan Pustaka) bagi setiap senarai, menunjukkan perkataan yang berpotensi mempunyai ejaan Indonesia (KBBI) dan keputusan semakan.

Fail `profanity-reject-ms.txt` ialah senarai 33 perkataan lucah atau sensitif yang digunakan untuk menyaring semua senarai. Senarai ini dikurasi daripada sumber LDNOOBW, sumber komuniti Malaysia, dan semakan manual.

## Nota

- Fail dalam `scratch/` dikekalkan untuk kebolehkesanan dan pengauditan. Ia tidak diperlukan untuk penggunaan senarai perkataan.
- Jika ruang menjadi isu, fail-fail ini boleh dipadamkan dengan selamat — ia boleh dijana semula melalui pipeline.
- `raw-test-run.txt`, `normalized-test-run.txt`, dll. adalah daripada ujian awal saluran paip sebelum terjemahan sebenar.
