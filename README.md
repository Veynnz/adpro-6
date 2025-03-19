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