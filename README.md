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