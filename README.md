# Rust - Concurrency

## Athallah Damar Jiwanto - B - 2306245024

### Module 6

> Milestone 1

Saat ini, kita telah mengembangkan fungsi handle_connection yang berfungsi untuk menangani koneksi dari browser. Fungsi ini menerima objek TcpStream yang didapat dari TcpListener. Dengan menggunakan BufReader, fungsi ini membaca permintaan HTTP secara bertahap, baris per baris, sampai menemukan baris kosong yang menandakan bahwa header permintaan telah selesai. Setelah proses pembacaan selesai, permintaan tersebut disimpan dalam sebuah vektor yang berisi setiap baris sebagai string dan ditampilkan di konsol. Meskipun aplikasi ini sudah berhasil menerima dan menampilkan permintaan dari browser, respons kepada browser belum dapat diberikan.

> Milestone 2

Fungsi handle_connection dalam kode ini bertugas untuk menangani koneksi yang masuk dari klien, seperti browser, menggunakan objek TcpStream. Fungsi ini membaca permintaan HTTP yang diterima melalui stream menggunakan BufReader dan mengumpulkan baris-baris permintaan hingga menemukan baris kosong, yang menandakan akhir dari header permintaan. Setelah itu, fungsi ini menyiapkan respons HTTP dengan status "200 OK" dan membaca konten dari file hello.html. Panjang konten tersebut dihitung dan ditambahkan dalam header Content-Length. Respons kemudian diformat dengan lengkap dan dikirim kembali ke klien melalui stream, sehingga klien menerima respons yang berisi konten dari file hello.html.

```rust
    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
```

![hello-html](images/commit2.png)