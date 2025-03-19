# Rust - Concurrency

## Athallah Damar Jiwanto - B - 2306245024

### Module 6

> Milestone 1

Saat ini, kita telah mengembangkan fungsi handle_connection yang berfungsi untuk menangani koneksi dari browser. Fungsi ini menerima objek TcpStream yang didapat dari TcpListener. Dengan menggunakan BufReader, fungsi ini membaca permintaan HTTP secara bertahap, baris per baris, sampai menemukan baris kosong yang menandakan bahwa header permintaan telah selesai. Setelah proses pembacaan selesai, permintaan tersebut disimpan dalam sebuah vektor yang berisi setiap baris sebagai string dan ditampilkan di konsol. Meskipun aplikasi ini sudah berhasil menerima dan menampilkan permintaan dari browser, respons kepada browser belum dapat diberikan.
