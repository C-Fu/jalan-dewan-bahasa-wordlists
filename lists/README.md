# Senarai Perkataan Jalan Dewan Bahasa — `lists/`

Direktori ini mengandungi 8 senarai perkataan Bahasa Melayu (Standard Malaysia, ejaan DBP) untuk penciptaan frasa laluan, bersama 8 senarai sumber bahasa Inggeris yang asal.

## Senarai Bahasa Melayu

| Fail | Saiz | Perkataan | Entropi | Penerangan |
|------|------|-----------|---------|------------|
| `jalan-dewan-bahasa-besar.txt` | 17,576 | Besar | 14.101 bit | Senarai panjang untuk entropi maksimum |
| `jalan-dewan-bahasa-sederhana.txt` | 8,192 | Sederhana | 13.000 bit | Dioptimumkan untuk komputer perduaan (2¹³) |
| `jalan-dewan-bahasa-dadu.txt` | 7,776 | Dadu | 12.925 bit | Dengan awalan nombor lontaran dadu 5 digit (11111–66666) |
| `jalan-dewan-bahasa-dadu-bersih.txt` | 7,776 | Dadu Bersih | 12.925 bit | Sama seperti dadu tetapi tanpa awalan dadu |
| `jalan-dewan-bahasa-kecil-alpha.txt` | 1,296 | Kecil Alpha | 10.340 bit | Susunan abjad — untuk papan kekunci abjad |
| `jalan-dewan-bahasa-kecil-alpha-dadu.txt` | 1,296 | Kecil Alpha Dadu | 10.340 bit | Alpha dengan awalan dadu 4 digit |
| `jalan-dewan-bahasa-kecil-qwerty.txt` | 1,296 | Kecil QWERTY | 10.340 bit | Susunan QWERTY — untuk papan kekunci QWERTY |
| `jalan-dewan-bahasa-kecil-qwerty-dadu.txt` | 1,296 | Kecil QWERTY Dadu | 10.340 bit | QWERTY dengan awalan dadu 4 digit |

Semua senarai telah disaring untuk perkataan lucah, melalui audit ejaan DBP, dan dibuktikan **boleh dinyahkod secara unik** (uniquely decodable) — selamat untuk digabungkan tanpa pembatas antara perkataan.

## Format Fail

### Senarai Biasa (tanpa dadu)

Satu perkataan setiap baris, huruf kecil, aksara a-z sahaja:

```
abad
abadi
abai
abaikan
abang
```

### Senarai Dadu (dengan awalan nombor lontaran dadu)

Nombor lontaran dadu diikuti tab, kemudian perkataan:

```
11111	abang
11112	abbot
11113	abdi
11114	abdinya
11115	abdomen
```

Setiap nombor mewakili lontaran dadu:
- **Senarai besar**: Tiada varian dadu (melebihi 6⁵ = 7,776 kombinasi)
- **Senarai sederhana**: Tiada varian dadu
- **Senarai dadu**: 5 digit (11111–66666) — 6⁵ = 7,776 kombinasi
- **Senarai kecil**: 4 digit (1111–6666) — 6⁴ = 1,296 kombinasi

## Cara Menggunakan

### Dengan `grep` — pilih perkataan rawak

```bash
# Pilih 6 perkataan rawak daripada senarai besar (setiap baris adalah satu perkataan)
shuf -n 6 lists/jalan-dewan-bahasa-besar.txt

# Contoh output:
# profesionalisme
# abadi
# cabaran
# kualiti
# sistem
# fungsi
```

### Dalam Skrip Shell — jana frasa laluan

```bash
#!/bin/bash
# Jana frasa laluan 5 perkataan daripada senarai sederhana
PASSPHRASE=$(shuf -n 5 lists/jalan-dewan-bahasa-sederhana.txt | tr '\n' '-' | sed 's/-$//')
echo "Frasa laluan anda: $PASSPHRASE"
# Contoh: membaca-makanan-pendidikan-keluarga-bantuan
```

### Dengan Dadu Fizikal — guna senarai dadu

Gulung dadu 5 kali untuk senarai dadu, atau 4 kali untuk senarai kecil. Setiap nombor lontaran digabungkan untuk membentuk kod (cth: 1-1-1-1-1 → 11111). Cari kod tersebut dalam senarai dadu:

```bash
# Cari perkataan untuk lontaran dadu 11111
grep "^11111" lists/jalan-dewan-bahasa-dadu.txt
# Output: 11111	abang
```

### Dalam Python

```python
import random

# Muatkan senarai perkataan
with open("lists/jalan-dewan-bahasa-sederhana.txt") as f:
    words = [line.strip() for line in f]

# Jana frasa laluan 6 perkataan
passphrase = "-".join(random.choices(words, k=6))
print(f"Frasa laluan: {passphrase}")
# Contoh: makanan-pendidikan-membaca-bantuan-keluarga-sistem

# Kira entropi
from math import log2
entropy_per_word = log2(len(words))
total_entropy = entropy_per_word * 6
print(f"Entropi: ~{total_entropy:.1f} bit")
# Entropi: ~78.0 bit
```

### Dalam Node.js / JavaScript

```javascript
import { readFileSync } from 'fs';

// Muatkan senarai perkataan dadu (dengan awalan)
const lines = readFileSync('lists/jalan-dewan-bahasa-dadu.txt', 'utf-8')
  .trim().split('\n');

// Cari perkataan berdasarkan lontaran dadu
function cariDadu(roll) {
  const line = lines.find(l => l.startsWith(roll + '\t'));
  return line ? line.split('\t')[1] : null;
}

// Contoh: gulung dadu 3-2-5-4-1 → 32541
console.log(cariDadu('32541')); // → perkataan yang sepadan

// Atau jana frasa laluan rawak daripada senarai bersih
const cleanWords = readFileSync('lists/jalan-dewan-bahasa-dadu-bersih.txt', 'utf-8')
  .trim().split('\n');

function janaFrasa(n = 6) {
  const chosen = [];
  for (let i = 0; i < n; i++) {
    chosen.push(cleanWords[Math.floor(Math.random() * cleanWords.length)]);
  }
  return chosen.join('-');
}

console.log(janaFrasa(5)); // Contoh: percaya-lukisan-mekanisme-hamba-panel
```

### Dalam Rust

```rust
use std::fs::File;
use std::io::{BufRead, BufReader};
use rand::seq::SliceRandom;

fn main() -> Result<(), std::io::Error> {
    // Muatkan senarai kecil QWERTY
    let file = File::open("lists/jalan-dewan-bahasa-kecil-qwerty.txt")?;
    let reader = BufReader::new(file);
    let words: Vec<String> = reader.lines()
        .map(|l| l.unwrap())
        .collect();

    // Pilih 4 perkataan rawak
    let mut rng = rand::thread_rng();
    let passphrase: Vec<&str> = words.choose_multiple(&mut rng, 4)
        .map(|s| s.as_str())
        .collect();

    println!("Frasa laluan: {}", passphrase.join("-"));
    // Contoh: akses-baru-cuba-emas

    Ok(())
}
```

## Nota Penting

- **Jangan gabungkan senarai** — menggabungkan perkataan daripada senarai yang berbeza hampir pasti akan memusnahkan sifat kebolehnilaian unik
- **Pilih saiz yang sesuai** — senarai besar memberikan entropi lebih tinggi tetapi perkataan lebih panjang; senarai kecil lebih mudah ditaip pada TV pintar atau konsol permainan
- **Frasa laluan tanpa pembatas** — oleh kerana senarai ini boleh dinyahkod secara unik, anda boleh menggabungkan perkataan tanpa sebarang pembatas: `makananpendidikanbantuan` selamat digunakan

## Senarai Sumber Bahasa Inggeris

Fail `orchard-street-*.txt` ialah senarai asal daripada projek [Orchard Street Wordlists](https://github.com/sts10/orchard-street-wordlists) dalam bahasa Inggeris, dikekalkan untuk rujukan dan kebolehkesanan.

---

*Untuk maklumat lanjut, lihat [README](../README.md) utama projek.*
