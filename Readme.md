# Commit 1 Reflection notes

Fungsi `handle_connection` memiliki peran untuk menangani koneksi TCP yang masuk. Selain itu, fungsi tersebut juga mengambil informasi HTTP request dari browser. Parameter `stream` merupakan koneksi antara client dan server yang nantinya dibungkus oleh `BufReader`. Pembungkusan ini bertujuan agar pembacaan data menjadi lebih efisien melalui teknik buffering. Data kemudian dibaca per baris menggunakan `.lines()`.

`.map(|result| result.unwrap())` mengolah data hasil pembacaan berupa `Result<String, Error>` untuk mengambil nilai `String`-nya. Pendekatan ini dapat menyebabkan program crash ketika terjadi error.

Pembacaan data dibatasi sampai menemukan baris kosong menggunakan `.take_while(|line| !line.is_empty())` karena pada format HTTP, header diakhiri oleh baris kosong.

Terakhir, `.collect()` mengumpulkan semua baris yang telah dibaca menjadi `Vec<String>`, sehingga data tersebut dapat disimpan dan digunakan selanjutnya.

Secara keseluruhan, fungsi ini menggambarkan bagaimana rust membaca struktur dasar HTTP request secara sederhana.

# Commit 2 Reflection notes

![Commit 2 screen capture](/assets/images/commit2.png)

Fungsi `handle_connection` diubah agar tidak hanya membaca HTTP request, tetapi juga memberikan respons berupa file HTML. Setelah melewati proses yang sudah dijelaskan pada commit 1, server mulai menyusun response HTTP yang akan dikembalikan. `fs::read_to_string("hello.html").unwrap();` digunakan untuk membaca file `hello.html` dan mengubahnya menjadi string. Server akan menyusun response dengan detail sebagai berikut:

- `status_line` (`HTTP/1.1 200 OK`) yang menandakan bahwa request berhasil diproses.
- `Content-Length` yang menunjukkan panjang isi response (jumlah byte dari file HTML).
- Baris kosong (`\r\n\r\n`) sebagai pemisah antara header dan body.
- `contents` yaitu isi dari file HTML yang akan ditampilkan di browser.

`stream.write_all():` Method ini digunakan untuk mengirim response yang sudah disusun ke client melalui koneksi TCP. Sebelum dikirim, response diubah menjadi byte menggunakan `.as_bytes()`. Penggunaan `unwrap()` di sini berarti program akan berhenti jika terjadi error saat pengiriman data.