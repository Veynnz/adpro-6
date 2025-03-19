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