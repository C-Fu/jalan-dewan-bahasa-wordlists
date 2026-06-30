# Soalan Lazim

## Bagaimanakah saya boleh menggunakan dadu untuk mencipta frasa laluan?

Saya cadangkan anda merujuk [panduan EFF tentang cara melakukannya](https://www.eff.org/dice) dan [artikel oleh Micah Lee ini](https://theintercept.com/2015/03/26/passphrases-can-memorize-attackers-cant-guess/). Ambil perhatian bahawa anda perlu menggunakan sama ada Senarai Dadu Jalan Dewan Bahasa atau salah satu Senarai Kecil.

## Bolehkah saya menetapkan pengurus kata laluan saya menggunakan Senarai Perkataan Jalan Dewan Bahasa?

Sesetengah pengurus kata laluan membenarkan pengguna menggunakan mana-mana fail senarai perkataan untuk menjana frasa laluan. [KeePassXC](https://keepassxc.org) (v 2.7+) ialah salah satu pengurus kata laluan sedemikian.

Untuk menetapkan KeePassXC menggunakan salah satu senarai perkataan ini, klik ikon dadu KeePassXC untuk membuka penjana kata laluan, kemudian klik ke tab "Passphrase", kemudian klik butang + untuk memilih fail senarai perkataan.

![Tangkapan layar menunjukkan cara menukar senarai perkataan yang digunakan oleh KeePassXC](img/keepassxc-use.png)

## Berapa _banyak_ patah perkataan yang patut saya gunakan dalam frasa laluan?

Itu bergantung pada model ancaman anda, jadi saya tidak dapat memberikan jawapan yang umum. Tetapi jika saya terpaksa memberikan satu panduan umum, saya cadangkan menggunakan 6 patah perkataan daripada senarai besar atau sederhana (contohnya "tangan fasa telefon lembu amaran cahaya") dan 7 patah perkataan daripada senarai kecil (contohnya "jubah tunda kayu isyarat cawan cepat setiap") adalah pilihan yang selamat.

## Adakah mana-mana pengurus kata laluan yang sedang menggunakan Senarai Perkataan Jalan Dewan Bahasa?

* [Pengurus kata laluan Buttercup](https://buttercup.pw/) [kini menggunakan](https://github.com/buttercup/buttercup-generator/pull/18) Senarai Sederhana Jalan Dewan Bahasa sebagai senarai perkataan frasa laluannya.
* [Strongbox](https://strongboxsafe.com/) [menawarkan](https://github.com/strongbox-password-safe/Strongbox/blob/master/resources/wordlists/orchard-street-diceware.txt) Senarai Dadu Jalan Dewan Bahasa kepada pengguna yang ingin menjana frasa laluan.

Jika anda menjumpai contoh lain, sila cipta Isu atau PR!

## Adakah anda mengesyorkan sebarang alat CLI untuk menjana frasa laluan?

Saya mencipta [sebuah penjana frasa laluan yang menggunakan Senarai Perkataan Jalan Dewan Bahasa yang saya namakan Phraze](https://github.com/sts10/phraze).

Jika anda tidak mempercayai saya atau lebih gemar Rust, terdapat juga [alat passphraseme oleh Micah Lee](https://github.com/micahflee/passphraseme). Gunakan opsyen `-d` / `--dictionary` untuk menggunakan fail Senarai Perkataan Jalan Dewan Bahasa.

## Perlukah saya menggunakan tanda baca pemisah antara perkataan dalam frasa laluan daripada Senarai Perkataan Jalan Dewan Bahasa?

Tidak. Semua Senarai Perkataan Jalan Dewan Bahasa adalah **boleh dinyahkod secara unik**, yang bermaksud perkataan daripada mana-mana satu senarai boleh digabungkan dengan selamat dalam frasa laluan _tanpa_ tanda baca antara perkataan, contohnya "getarceritajelasmembuktikanregangsebiji". Walau bagaimanapun, tidak ada salahnya meletakkan ruang, sempang, garis bawah, dsb. antara perkataan jika anda lebih suka.

## Apakah perbezaan antara Senarai Dadu Jalan Dewan Bahasa dan "senarai panjang" EFF? Kedua-duanya mempunyai 7,776 patah perkataan...

Kedua-duanya agak serupa! Kedua-dua [senarai panjang EFF](https://www.eff.org/deeplinks/2016/07/new-wordlists-random-passphrases) dan Senarai Dadu Jalan Dewan Bahasa mengandungi tepat 7,776 patah perkataan. Ini bertujuan supaya setiap perkataan dapat sepadan dengan lontaran 5 biji dadu 6 sisi. Kedua-dua senarai juga _boleh dinyahkod secara unik_, yang bermaksud frasa laluan daripadanya tidak memerlukan pembatas antara perkataan.

Satu perbezaan ialah senarai EFF boleh dinyahkod secara unik kerana ia tidak mempunyai "perkataan awalan". Senarai Dadu Jalan Dewan Bahasa dijadikan boleh dinyahkod secara unik melalui [satu proses baharu yang saya cipta yang dipanggil pemangkasan Schlinkert](https://sts10.github.io/2022/08/12/efficiently-pruning-until-uniquely-decodable.html) (dan oleh itu, Senarai Dadu Jalan Dewan Bahasa memang mempunyai perkataan awalan di dalamnya). Saya juga ingin menyatakan bahawa purata panjang perkataan senarai EFF adalah lebih pendek sedikit (sebanyak 0.07 aksara).

Akhir sekali, senarai EFF memang mengandungi beberapa perkataan yang agak janggal dalam bahasa Inggeris, seperti "grope", "gonad", "ecard", dan "footsie", beberapa perkataan bersempang seperti "drop-down" dan "yo-yo", serta beberapa nama peranti Apple ("ipad", "iphone", "ipod").

Walaupun begitu, senarai EFF sememangnya lebih terkenal dan merupakan pilihan yang lebih meluas digunakan, justeru ia adalah pilihan yang kurang berisiko. Tetapi jika anda sedang membaca Soalan Lazim ini, mungkin anda ingin mencuba perkara baharu...

## Saya mencipta frasa laluan yang saya tahu akan kerap saya masukkan ke dalam televisyen pintar atau konsol permainan video. Senarai manakah yang patut saya gunakan?

Memasukkan kata laluan selamat pada alat kawalan jauh televisyen pintar atau pengawal permainan video adalah menyusahkan. Untuk memudahkannya, Senarai Kecil Jalan Dewan Bahasa dioptimumkan untuk meminimumkan bilangan "klik" yang perlu anda lakukan untuk memasukkan frasa laluan.

Bilangan klik ini bergantung pada susun atur papan kekunci. Jika papan kekunci kata laluan perkhidmatan tersebut kelihatan seperti susun atur QWERTY tradisional:

```txt
qwertyuiop
asdfghjkl
zxcvbnm
```

gunakan [Senarai Kecil QWERTY Jalan Dewan Bahasa](lists/jalan-dewan-bahasa-kecil-qwerty.txt).

Jika ia lebih hampir kepada susunan abjad:

```txt
abcdef
ghijkl
mnopqr
stuvwx
yz
```

gunakan [Senarai Kecil Abjad Jalan Dewan Bahasa](lists/jalan-dewan-bahasa-kecil-alpha.txt).

Anda boleh membaca [catatan blog ini untuk maklumat lanjut](https://sts10.github.io/2022/10/24/a-good-netflix-password.html).

## Saya sedang mencipta perisian penjanaan frasa laluan. Bolehkah saya menggunakan satu atau lebih Senarai Perkataan Jalan Dewan Bahasa dalam projek saya?

Sudah tentu! Cuma pastikan anda mematuhi lesen yang berkenaan (lihat fail readme).

## Jika saya mahukan senarai yang sangat panjang, bolehkah saya menggabungkan semua Senarai Perkataan Jalan Dewan Bahasa menjadi satu senarai yang amat panjang?

Saya TIDAK mengesyorkan melakukan perkara ini. Sebabnya, walaupun anda membuang perkataan pendua, senarai yang terhasil **hampir pasti tidak boleh dinyahkod secara unik**, satu kualiti yang penting.

Walau bagaimanapun, jika anda benar-benar perlu menyunting senarai sedia ada atau membuat senarai perkataan anda sendiri, anda dialu-alukan untuk menggunakan alat yang saya tulis bernama [Tidy](https://github.com/sts10/tidy), yang boleh menjadikan senarai boleh dinyahkod secara unik menggunakan pelbagai kaedah, termasuk pemangkasan Schlinkert.

Akhir sekali, jika anda mahukan senarai yang sangat panjang yang boleh dinyahkod secara unik, anda boleh mencuba [senarai 40,000 patah perkataan yang saya cipta ini](https://github.com/sts10/generated-wordlists/blob/main/lists/experimental/ud2.txt) sebagai sebahagian daripada projek lain.

## Bagaimanakah Senarai Perkataan Jalan Dewan Bahasa dicipta?

Senarai Perkataan Jalan Dewan Bahasa ialah terjemahan langsung ke dalam Bahasa Melayu Standard (Malaysia, ejaan DBP) bagi Senarai Perkataan Orchard Street yang asal dalam bahasa Inggeris. Perkataan-perkataan dalam senarai asal bahasa Inggeris diambil daripada dua sumber: [data Google Books Ngram](https://storage.googleapis.com/books/ngrams/books/datasetsv3.html) dan Wikipedia (bahasa Inggeris), melalui [projek kekerapan perkataan Wikipedia](https://github.com/IlyaSemenov/wikipedia-word-frequency/). Terjemahan dilakukan menggunakan DeepSeek v4 Pro.

Selepas terjemahan, senarai dijadikan boleh dinyahkod secara unik menggunakan proses berdasarkan [algoritma Sardinas–Patterson](https://en.wikipedia.org/wiki/Sardinas%E2%80%93Patterson_algorithm) yang saya panggil [pemangkasan Schlinkert](https://sts10.github.io/2022/08/12/efficiently-pruning-until-uniquely-decodable.html), dilaksanakan melalui alat Tidy. Penormalan Unicode, penyisihan locale Bahasa Melayu, dan penyingkiran perkataan lucah turut dijalankan bagi memastikan kualiti senarai akhir.

### Apakah alat yang digunakan untuk mencipta Senarai Perkataan Jalan Dewan Bahasa?

- [Common Word List Maker](https://github.com/sts10/common_word_list_maker): Mengikis data Google Books Ngram untuk mencipta senarai panjang perkataan yang lazim digunakan (untuk senarai asal bahasa Inggeris)
- [Penjana kekerapan perkataan Wikipedia](https://github.com/IlyaSemenov/wikipedia-word-frequency): Mengumpul kekerapan perkataan daripada rencana Wikipedia (untuk senarai asal bahasa Inggeris)
- [DeepSeek v4 Pro](https://deepseek.com/): Enjin terjemahan AI yang digunakan untuk menterjemah perkataan bahasa Inggeris ke Bahasa Melayu Standard
- [Tidy](https://github.com/sts10/tidy): Utiliti baris perintah untuk menyunting senarai perkataan. Saya yang menulisnya! Digunakan untuk penormalan, pemangkasan Schlinkert, dan pengesahan kebolehnahkodan unik
