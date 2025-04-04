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

> Milestone 3

Untuk membedakan antara respons yang sukses seperti `200 OK` dan respons error seperti `404 NOT FOUND`, kita dapat memisahkan logika pengolahan respons menjadi bagian yang lebih terstruktur. Misalnya, kita bisa membuat fungsi atau modul terpisah untuk menangani pembuatan respons, seperti memisahkan bagian format status, header, dan konten. Ini akan meningkatkan keterbacaan dan pemeliharaan kode, karena kita tidak perlu menulis ulang logika yang sama untuk setiap jenis respons. Selain itu, refaktoring seperti ini juga membuat kode lebih modular dan dapat digunakan kembali, memungkinkan penanganan berbagai jenis respons dalam satu tempat. Refaktoring ini penting untuk menjaga konsistensi, meminimalkan duplikasi kode, dan memungkinkan pengembangan lebih lanjut tanpa merusak struktur kode yang ada. Dengan memisahkan logika pengolahan respons, kita juga mempermudah pengujian unit dan debugging aplikasi secara keseluruhan.

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
}
```

![404-html](images/commit3.png)

> Milestone 4

Penggunaan satu thread ini, meskipun sederhana, dapat menimbulkan masalah saat banyak pengguna mengakses server secara bersamaan. Karena server hanya memiliki satu thread yang menangani semua permintaan, ketika ada banyak pengguna yang meminta akses ke server, setiap permintaan akan diproses secara berurutan. Artinya, jika satu pengguna mengakses path yang menyebabkan server menunggu, server akan "terblokir" dan tidak dapat menangani permintaan lainnya sampai permintaan tersebut selesai. Ini dapat menyebabkan kinerja yang buruk dan waktu respons yang sangat lama bagi pengguna lain.

```rust
...
    let (status_line, filename) = match &request_line[..] { 
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"), 
        "GET /sleep HTTP/1.1" => { 
            thread::sleep(Duration::from_secs(10)); 
            ("HTTP/1.1 200 OK", "hello.html") 
        } 
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"), 
    }; 
...
```

> Milestone 5

ThreadPool digunakan untuk mengelola sekumpulan worker threads yang bertugas menangani permintaan dari klien secara bersamaan. Ketika server menerima permintaan melalui `TcpListener`, alih-alih menangani setiap permintaan secara berurutan dalam satu thread, permintaan tersebut diserahkan ke ThreadPool untuk diproses oleh salah satu worker threads yang ada. Dengan begitu, server dapat mengeksekusi pekerjaan secara paralel, sehingga menghindari penundaan akibat permintaan lambat, seperti pada kasus permintaan yang mengakses `/sleep`. ThreadPool terdiri dari beberapa worker yang setiapnya menjalankan loop untuk mengambil job dari channel dan mengeksekusinya, memungkinkan server untuk menangani banyak permintaan tanpa terblokir. Pendekatan ini meningkatkan skalabilitas dan responsivitas server, memungkinkan server untuk tetap berjalan efisien meskipun ada banyak pengguna yang mengaksesnya pada waktu yang bersamaan.

implementasi pada main.rs
```rust
fn main() { 
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() { 
        let stream = stream.unwrap(); 
 
        pool.execute(|| {
            handle_connection(stream);
        });
    } 
}
```

implementasi dengan pembuatan file baru, lib.rs
```rust
use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool { workers, sender }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();

            println!("Worker {id} got a job; executing.");

            job();
        });

        Worker { id, thread }
    }
}
```