# Pangkalan Data SQLite Jalan Dewan Bahasa — `db/`

Direktori ini mengandungi 8 pangkalan data SQLite yang sepadan dengan setiap senarai perkataan Bahasa Melayu dalam format `.txt`. Pangkalan data ini membolehkan carian perkataan yang pantas dan integrasi terus dengan aplikasi pengurus kata laluan, alat CLI, atau laman web.

## Senarai Fail

| Fail | Saiz | Baris | Jadual | Penerangan |
|------|------|-------|--------|------------|
| `jalan-dewan-bahasa-besar.db` | 17,576 | Besar | `words(word)` | Senarai panjang |
| `jalan-dewan-bahasa-sederhana.db` | 8,192 | Sederhana | `words(word)` | Senarai sederhana |
| `jalan-dewan-bahasa-dadu.db` | 7,776 | Dadu | `words(roll, word)` | Dengan awalan dadu |
| `jalan-dewan-bahasa-dadu-bersih.db` | 7,776 | Dadu Bersih | `words(word)` | Tanpa awalan dadu |
| `jalan-dewan-bahasa-kecil-alpha.db` | 1,296 | Kecil Alpha | `words(word)` | Susunan abjad |
| `jalan-dewan-bahasa-kecil-alpha-dadu.db` | 1,296 | Kecil Alpha Dadu | `words(roll, word)` | Alpha dengan dadu |
| `jalan-dewan-bahasa-kecil-qwerty.db` | 1,296 | Kecil QWERTY | `words(word)` | Susunan QWERTY |
| `jalan-dewan-bahasa-kecil-qwerty-dadu.db` | 1,296 | Kecil QWERTY Dadu | `words(roll, word)` | QWERTY dengan dadu |

## Skema Pangkalan Data

### Senarai Biasa (tanpa dadu)

Jadual `words` dengan lajur:

```sql
CREATE TABLE words (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    word TEXT NOT NULL UNIQUE,
    length INTEGER GENERATED ALWAYS AS (LENGTH(word)) STORED
);
```

### Senarai Dadu (dengan awalan nombor lontaran dadu)

Jadual `words` dengan lajur tambahan `roll`:

```sql
CREATE TABLE words (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    roll TEXT NOT NULL UNIQUE,
    word TEXT NOT NULL,
    length INTEGER GENERATED ALWAYS AS (LENGTH(word)) STORED
);
```

## Cara Menggunakan

### Dengan alat baris arahan `sqlite3`

```bash
# Lihat skema pangkalan data
sqlite3 db/jalan-dewan-bahasa-sederhana.db ".schema words"

# Kira bilangan perkataan
sqlite3 db/jalan-dewan-bahasa-sederhana.db "SELECT COUNT(*) FROM words;"
# Output: 8192

# Pilih 6 perkataan rawak untuk frasa laluan
sqlite3 db/jalan-dewan-bahasa-sederhana.db "
    SELECT word FROM words ORDER BY RANDOM() LIMIT 6;
"
# Contoh output:
# makanan
# pendidikan
# membaca
# bantuan
# keluarga
# sistem

# Cari perkataan untuk lontaran dadu 11111
sqlite3 db/jalan-dewan-bahasa-dadu.db "
    SELECT roll, word FROM words WHERE roll = '11111';
"
# Output: 11111|abang

# Semak julat lontaran dadu
sqlite3 db/jalan-dewan-bahasa-dadu.db "
    SELECT MIN(roll), MAX(roll) FROM words;
"
# Output: 11111|66666

# Cari semua perkataan yang bermula dengan "pendidik"
sqlite3 db/jalan-dewan-bahasa-sederhana.db "
    SELECT word FROM words WHERE word LIKE 'pendidik%';
"
# Output: pendidikan
```

### Dalam Python — jana frasa laluan

```python
import sqlite3
from math import log2

# Sambung ke pangkalan data
conn = sqlite3.connect("db/jalan-dewan-bahasa-sederhana.db")
cur = conn.cursor()

# Kira entropi
cur.execute("SELECT COUNT(*) FROM words")
total_words = cur.fetchone()[0]
entropy_per_word = log2(total_words)
print(f"Jumlah perkataan: {total_words}")
print(f"Entropi per perkataan: {entropy_per_word:.3f} bit")

# Jana frasa laluan 6 perkataan
cur.execute("SELECT word FROM words ORDER BY RANDOM() LIMIT 6")
words = [row[0] for row in cur.fetchall()]
passphrase = "-".join(words)
print(f"Frasa laluan: {passphrase}")
total_entropy = entropy_per_word * 6
print(f"Jumlah entropi: ~{total_entropy:.1f} bit")

conn.close()
```

### Dalam Python — guna senarai dadu

```python
import sqlite3, random

conn = sqlite3.connect("db/jalan-dewan-bahasa-kecil-qwerty-dadu.db")
cur = conn.cursor()

# Simulasi lontaran dadu fizikal
rolls = [str(random.randint(1, 6)) for _ in range(4)]
dice_code = "".join(rolls)
print(f"Anda melontar: {' '.join(rolls)}")
print(f"Kod dadu: {dice_code}")

cur.execute("SELECT word FROM words WHERE roll = ?", (dice_code,))
result = cur.fetchone()
if result:
    print(f"Perkataan: {result[0]}")
else:
    print(f"Tiada perkataan untuk kod {dice_code}")

conn.close()
```

### Dalam Node.js / JavaScript

Gunakan pakej `better-sqlite3` (synchronous, native) atau `sql.js` (WebAssembly, berfungsi dalam pelayar):

```javascript
// Native (Node.js) dengan better-sqlite3
const Database = require('better-sqlite3');
const db = new Database('db/jalan-dewan-bahasa-sederhana.db');

// Kira jumlah perkataan
const count = db.prepare('SELECT COUNT(*) AS c FROM words').get();
console.log(`Jumlah perkataan: ${count.c}`);

// Pilih 4 perkataan rawak
const words = db.prepare('SELECT word FROM words ORDER BY RANDOM() LIMIT 4')
  .all()
  .map(row => row.word);

console.log(`Frasa laluan: ${words.join('-')}`);
db.close();
```

```javascript
// Dalam pelayar dengan sql.js (WebAssembly)
// Lihat: https://sql.js.org/
async function janaFrasa() {
  const SQL = await initSqlJs({
    locateFile: file => `https://cdn.jsdelivr.net/npm/sql.js@1.10/dist/${file}`
  });

  const resp = await fetch('db/jalan-dewan-bahasa-kecil-qwerty.db');
  const buf = await resp.arrayBuffer();
  const db = new SQL.Database(new Uint8Array(buf));

  const result = db.exec("SELECT word FROM words ORDER BY RANDOM() LIMIT 6");
  const words = result[0].values.map(row => row[0]);
  console.log(`Frasa laluan: ${words.join('-')}`);
}
```

### Dalam Rust

```rust
use rusqlite::Connection;
use rand::seq::SliceRandom;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let conn = Connection::open("db/jalan-dewan-bahasa-besar.db")?;

    // Kira jumlah perkataan
    let count: i64 = conn.query_row(
        "SELECT COUNT(*) FROM words", [], |row| row.get(0)
    )?;
    println!("Jumlah perkataan: {}", count);

    // Pilih 5 perkataan rawak
    let mut stmt = conn.prepare("SELECT word FROM words ORDER BY RANDOM() LIMIT 5")?;
    let words: Vec<String> = stmt.query_map([], |row| row.get(0))?
        .collect::<Result<Vec<_>, _>>()?;

    println!("Frasa laluan: {}", words.join("-"));

    Ok(())
}
```

### Dengan senarai dadu — cari perkataan untuk lontaran tertentu

```bash
# Cari perkataan untuk lontaran dadu 3-3-5-5 (senarai kecil)
sqlite3 db/jalan-dewan-bahasa-kecil-alpha-dadu.db \
  "SELECT word FROM words WHERE roll = '3355';"

# Cari perkataan untuk lontaran dadu 2-4-6-1-3 (senarai dadu)
sqlite3 db/jalan-dewan-bahasa-dadu.db \
  "SELECT word FROM words WHERE roll = '24613';"

# Jana 3 frasa laluan berbeza dalam satu arahan
sqlite3 db/jalan-dewan-bahasa-sederhana.db "
  SELECT group_concat(word, '-') AS passphrase FROM (
    SELECT word FROM words ORDER BY RANDOM() LIMIT 5
  );
"
```

## Pengesahan Integriti

Semua pangkalan data telah diuji dan lulus pemeriksaan integriti:

```bash
for db in db/*.db; do
  result=$(sqlite3 "$db" "PRAGMA integrity_check;")
  rows=$(sqlite3 "$db" "SELECT COUNT(*) FROM words;")
  echo "$db: $result ($rows baris)"
done
```

## Nota Prestasi

- Pangkalan data ini menggunakan indeks automatik melalui kekangan `UNIQUE` pada lajur `word` (dan `roll` untuk varian dadu)
- Lajur `length` adalah lajur janaan (GENERATED ALWAYS AS) — dikira secara automatik, tiada ruang storan tambahan
- Pertanyaan `ORDER BY RANDOM() LIMIT N` adalah cekap untuk saiz senarai ini (sehingga 17,576 baris)

---

*Untuk senarai perkataan dalam format teks biasa, lihat direktori [`lists/`](../lists/). Untuk maklumat lanjut, lihat [README](../README.md) utama projek.*
