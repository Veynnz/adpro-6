# MODULE-6

## Commit 1 Reflection Notes

- Inside the `handle_connection` method
    - The function takes a `TcpStream` as a mutable parameter (`mut stream`), which represents an active network connection.
    - A `BufReader` is created from `stream` to read data in buffered mode.
    - .lines(): Reads the stream line by line.
    - .map(|result| result.unwrap()): Extracts the String from the `Result`, if the reading was successful
    - .take_while(|line| !line.is_empty()): Continues collecting lines until an empty line is encountered.
    - .collect(): Stores all collected lines in a as a vector, representing the HTTP request headers.
    - Prints the collected HTTP request headers.

## Commit 2 Reflection notes

- Additional lines in the `handle_connection` method
    - `"HTTP/1.1 200 OK"` defines the status line for the HTTP response, telling the client the request was successful.
    - Reads the `hello.html` file and stores its contents in the `contents` variable.
    - `.unwrap()` forces the program to crash if the file doesn't exist.
    - Calculates the `content` size/length.
    - Creates the full HTTP response from the given `status_line`, `Content-length`, and `contents`.
    - Sends the response as bytes (`.as_bytes()`) to the client using `stream.write_all()`, and forces the program to crash if there's an error using `.unwrap()`.

    ![Commit 2 screen capture](/public/images/commit2.png)

## Commit 3 Reflection notes

- How to split between responses
    - The HTTP request lines are being read just like before.
    - `.next()` in `request_line` takes only the first line, which contains the request method and path.
    - I made another html file, `404.html`, that handles every bad requests.
        - If the HTTP request is sent to `/`, it returns as a success (status code: 200), and loads `hello.html`.
        - If the HTTP request is anything else (not `/`), it returns as a failure (status code: 404), and loads `404.html`.

- Why refactoring is needed
    - Both conditions (`if` and `else`) basically contain the same steps, with the only difference being the returned `file name` and `status code`.
    - Duplicated logic would make the program seem messy and harder to maintain.
    - The refactored version eliminates the duplication and makes the code cleaner and more maintainable.
    - If i want to create more pages, i would only need to modify the `if` block.

    ![Commit 3 screen capture](/public/images/commit3.png)

## Commit 4 Reflection notes

- Why the browser take some time to load
    - A request to `/sleep` takes longer because of:
        ```rust
        thread::sleep(Duration::from_secs(10));
        ```
        - When the browser requests `/sleep`, the server pauses for 10 seconds before responding.
    - If the browser requests `/` first, it loads quickly.
    However, if `/sleep` is requested first, the `/` request must wait until `/sleep` is finished.
        - This happens because the server runs on a single thread, processing one request at a time.
        -  As a result, the next request cannot be handled until the previous one is completed.

## Commit 5 Reflection notes

- Creating the ThreadPool
    - A fixed number of `worker threads` is created (in this case 4).
    - A message queue (`channel`) is set up to send tasks (`jobs`).
    - A shared queue (`Arc<Mutex<Receiver<Job>>`) allows `workers` to safely access `jobs`.

    ```rust
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

    ...

    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
    ...
    ```

- Executing `jobs`
    - The `job` (a function) is wrapped in a Box and sent to the queue.
    - `Workers` pick up `jobs` from the queue when they are free.

    ```rust
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
    ```

- Worker Threads (Worker::new)
    - Each worker is a separate thread that continuously:
        - Waits for a job in the queue.
        - Locks the queue and takes the next available job.
        - Executes the job.
        - Repeats the process.

    ```rust
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

## Commit Bonus Reflection notes

Pada bagian ini, hanya diubah approach pembuatan ThreadPool, dari `new` menjadi `build`. 
- Metode `new()` biasa digunakan untuk membuat objek yang selalu berhasil tanpa kemungkinan gagal. Jika ada kondisi yang tidak valid, seperti ukuran `ThreadPool` yang nol, biasanya digunakan `panic!` untuk menghentikan program. Ini sesuai dengan konvensi `Rust`, di mana `new()` diasumsikan aman digunakan tanpa perlu penanganan kesalahan tambahan.

- Sebaliknya, metode `build()` lebih cocok jika ada kemungkinan kesalahan yang perlu ditangani. Dibandingkan menyebabkan crash dengan panic!, `build()` dapat mengembalikan `Result<T, E>` sehingga pemanggilnya bisa menangani kesalahan dengan lebih baik. Dalam kasus `ThreadPool`, penggunaan `build()` lebih disarankan karena memungkinkan validasi yang lebih fleksibel tanpa menghentikan program secara tiba-tiba.