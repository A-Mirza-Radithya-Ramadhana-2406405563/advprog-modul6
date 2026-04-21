# Commit 1 Reflection notes

Fungsi `handle_connection` memiliki peran untuk menangani koneksi TCP yang masuk. Selain itu, fungsi tersebut juga mengambil informasi HTTP request dari browser. Parameter `stream` merupakan koneksi antara client dan server yang nantinya dibungkus oleh `BufReader`. Pembungkusan ini bertujuan agar pembacaan data menjadi lebih efisien melalui teknik buffering. Data kemudian dibaca per baris menggunakan `.lines()`.

`.map(|result| result.unwrap())` mengolah data hasil pembacaan berupa `Result<String, Error>` untuk mengambil nilai `String`-nya. Pendekatan ini dapat menyebabkan program crash ketika terjadi error.

Pembacaan data dibatasi sampai menemukan baris kosong menggunakan `.take_while(|line| !line.is_empty())` karena pada format HTTP, header diakhiri oleh baris kosong.

Terakhir, `.collect()` mengumpulkan semua baris yang telah dibaca menjadi `Vec<String>`, sehingga data tersebut dapat disimpan dan digunakan selanjutnya.

Secara keseluruhan, fungsi ini menggambarkan bagaimana rust membaca struktur dasar HTTP request secara sederhana.