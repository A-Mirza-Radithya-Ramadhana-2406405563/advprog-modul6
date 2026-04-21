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

# Commit 3 Reflection notes

![Commit 3 screen capture](/assets/images/commit3.png)

Pada tahap ini, saya menambahkan logika untuk memeriksa request yang diterima oleh server. Server mengambil baris pertama dari HTTP request (`request_line`) dan membandingkannya dengan `"GET / HTTP/1.1"`. Jika sesuai (artinya client meminta halaman utama), maka server akan mengirimkan response dengan status `200 OK` beserta isi file `hello.html`. Sebaliknya, jika request tidak sesuai, server akan mengembalikan status `404 NOT FOUND` dan menampilkan file `404.html` sebagai halaman error.

Dalam proses refactoring, sebelumnya terdapat duplikasi kode pada blok `if-else`, terutama pada bagian membaca file, menyusun response, dan mengirimkannya ke client. Untuk menghindari pengulangan kode (prinsip DRY - Don't Repeat Yourself), refactoring dilakukan dengan memusatkan perbedaan hanya pada nilai `status_line` dan `filename` dalam bentuk tuple. Dengan cara ini, proses pembacaan file dan pengiriman response cukup ditulis satu kali di luar kondisi `if-else`. Hasilnya, kode menjadi lebih ringkas, lebih mudah dibaca, dan lebih mudah untuk dikembangkan di kemudian hari.

# Commit 4 Reflection notes

Pada tahap ini, saya menambahkan rute baru yaitu `/sleep` untuk mensimulasikan proses yang membutuhkan waktu lama. Ketika endpoint ini diakses, server akan menghentikan sementara eksekusi menggunakan `thread::sleep(Duration::from_secs(10))` sebelum akhirnya mengembalikan response berupa file `hello.html` dengan status 200 OK. Selain itu, saya juga menggunakan `match` untuk memetakan beberapa kemungkinan request sehingga penanganan rute menjadi lebih terstruktur dibandingkan sebelumnya.

Selanjutnya, saya melakukan pengujian dengan membuka dua tab browser. Tab pertama mengakses `127.0.0.1:7878/sleep`, kemudian tab kedua mengakses `127.0.0.1:7878`. Hasilnya, tab kedua tidak langsung mendapatkan response dan harus menunggu hingga proses `sleep` di tab pertama selesai. Hal ini menunjukkan bahwa server belum mampu menangani beberapa request secara bersamaan.

Dari percobaan ini, dapat disimpulkan bahwa server masih berjalan secara sinkron (single-threaded). Artinya, hanya satu koneksi yang dapat diproses dalam satu waktu, sementara request lainnya harus menunggu giliran. Kondisi ini berpotensi menimbulkan bottleneck dan membuat performa server tidak optimal, terutama jika ada request yang lambat. Oleh karena itu, diperlukan pendekatan seperti multi-threading atau thread pool agar server dapat menangani banyak koneksi secara paralel dan lebih efisien.