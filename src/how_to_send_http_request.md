# Send HTTP request

`std.http` doesn't support `HTTP2` yet, but it works well with `HTTP 1.1`.

The following examples use `https://jsonplaceholder.typicode.com/` for the fake
JSON data.

</br>

Here is the JSON struct:

```c
const Post = struct {
    userId: usize,
    id: usize,
    title: []const u8,
    body: []const u8,

    const Self = @This();

    pub fn show(self: *const Self) !void {
        var buffer: [256]u8 = undefined;
        const info = try std.fmt.bufPrint(
            &buffer,
            "[ Post ]\n{{\n\tuserId: {d},\n\tid: {d},\n\ttitle: {s},\n\tbody: {s}\n}}",
            .{
                self.userId, self.id, self.title, self.body,
            },
        );
        print("\n>>> {s}", .{info});
    }
};
```

</br>

- GET request to retrieve list of post

    ```c
    fn list_posts(parent_allocator: std.mem.Allocator) !void {
        var arena = std.heap.ArenaAllocator.init(parent_allocator);
        defer arena.deinit();
        const allocator = arena.allocator();

        //
        // Url string
        //
        const url_string = "https://jsonplaceholder.typicode.com/posts";
        print("\n>>> [ list_posts ] - url: {s}", .{url_string});
        const url = try std.Uri.parse(url_string);

        //
        // Http client
        //
        var client: std.http.Client = .{ .allocator = allocator };
        defer client.deinit();

        //
        // Http client request
        //
        var headers = std.http.Headers{ .allocator = allocator };
        var req = try client.request(.GET, url, headers, .{});
        defer req.deinit();

        //
        // Start sending request and wait for the response to finish
        //
        try req.start();
        try req.wait();

        //
        // Read the entire response body, but only allow 256KB max
        // allocate memory. If response body is larger than the 256KB,
        // it fails with `StreamTooLong` error!!!
        //
        const body_json_str = try req.reader().readAllAlloc(
            allocator,
            256 * 1024,
        );
        // print("\n>>> [ list_posts ] Response body_json_str: {s}", .{body_json_str});

        const post_arr = try std.json.parseFromSlice(
            []Post,
            allocator,
            body_json_str,
            .{},
        );
        defer std.json.parseFree([]Post, allocator, post_arr);

        print("\n>>> [ list_posts ] - post count: {d}", .{post_arr.len});
        print("\n>>> [ list_posts ] - the last post:", .{});
        try post_arr[post_arr.len - 1].show();
    }
    ```

    ```bash
    # >>> [ list_posts ] - url: https://jsonplaceholder.typicode.com/posts
    # >>> [ list_posts ] - post count: 100
    # >>> [ list_posts ] - the last post:
    # >>> [ Post ]
    # {
    #         userId: 10,
    #         id: 100,
    #         title: at nam consequatur ea labore ea harum,
    #         body: cupiditate quo est a modi nesciunt soluta
    # ipsa voluptas error itaque dicta in
    # autem qui minus magnam et distinctio eum
    # accusamus ratione error aut
    # }
    ```

    </br>

- GET request with Custom headers

    ```c
    fn query_post(parent_allocator: std.mem.Allocator, post_id: usize) !void {
        var arena = std.heap.ArenaAllocator.init(parent_allocator);
        defer arena.deinit();
        const allocator = arena.allocator();

        //
        // Url string
        //
        var url_buffer = [_]u8{0x00} ** 256;
        const url_string = try std.fmt.bufPrint(
            &url_buffer,
            "https://jsonplaceholder.typicode.com/posts/{d}",
            .{post_id},
        );
        print("\n>>> [ query_post ] - url: {s}", .{url_string});

        const url = std.Uri.parse(url_string) catch unreachable;

        //
        // Http client
        //
        var client: std.http.Client = .{ .allocator = allocator };
        defer client.deinit();

        //
        // Custom headers
        //
        var headers = std.http.Headers{ .allocator = allocator };
        try headers.append("Content-Type", "application/json");
        try headers.append("Authentication", "My secret key heye:)");

        //
        // Http client request
        //
        var req = try client.request(.GET, url, headers, .{});
        defer req.deinit();

        // Print headers (just for debugging purpose)
        for (req.headers.list.items) |entry| {
            print(
                "\n>>> [ query_post ] - header: {{ name: {s}, value: '{s}' }}",
                .{ entry.name, entry.value },
            );
        }

        //
        // Start sending request and wait for the response to finish
        //
        try req.start();
        try req.wait();

        // //
        // // Read response status, headers for debugging purpose
        // //
        // print("\n>>> [ query_post ] Response status: {d}", .{req.response.status});
        // print("\n>>> [ query_post ] Response reason: {s}", .{req.response.reason});
        // print(
        //     "\n>>> [ query_post ] Response header: {{ name: Content-Type, value: '{s}' }}",
        //     .{req.response.headers.getFirstValue("Content-Type") orelse ""},
        // );
        // print(
        //     "\n>>> [ query_post ] Response header: {{ name: Header-non-exists, value: '{s}' }}",
        //     .{req.response.headers.getFirstValue("Header-non-exists") orelse ""},
        // );

        //
        // Read the entire response body, but only allow 256KB max
        // allocate memory. If response body is larger than the 256KB,
        // it fails with `StreamTooLong` error!!!
        //
        const body_json_str = try req.reader().readAllAlloc(
            allocator,
            256 * 1024,
        );
        print("\n>>> [ query_post ] Response body_json_str: {s}", .{body_json_str});

        const post = try std.json.parseFromSlice(
            Post,
            allocator,
            body_json_str,
            .{},
        );
        defer std.json.parseFree(Post, allocator, post);

        try post.show();
    }
    ```

    ```bash
    # >>> [ query_post ] - url: https://jsonplaceholder.typicode.com/posts/8
    # >>> [ query_post ] - header: { name: Content-Type, value: 'application/json' }
    # >>> [ query_post ] - header: { name: Authentication, value: 'My secret key heye:)' }
    # >>> [ query_post ] Response body_json_str: {
    #   "userId": 1,
    #   "id": 8,
    #   "title": "dolorem dolore est ipsam",
    #   "body": "dignissimos aperiam dolorem qui eum\nfacilis quibusdam animi sint suscipit qui sint possimus cum\nquaerat magni maiores excepturi\nipsam ut commodi dolor voluptatum modi aut vitae"
    # }
    # >>> [ Post ]
    # {
    #         userId: 1,
    #         id: 8,
    #         title: dolorem dolore est ipsam,
    #         body: dignissimos aperiam dolorem qui eum
    # facilis quibusdam animi sint suscipit qui sint possimus cum
    # quaerat magni maiores excepturi
    # ipsam ut commodi dolor voluptatum modi aut vitae
    # }
    ```

    </br>

- POST request

    ```c
    ```

    ```bash
    ```

    </br>


### About choosing allocator for the HTTP server request handler

As you can see above, both `list_posts` and `query_post` accept a
`parent_allocator` to create an `ArenaAllocator`, and I use
`GPA (General Purpose Allocator)` in those examples:

```c
pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    const allocator = gpa.allocator();
    defer {
        const deinit_status = gpa.deinit();
        //fail test; can't try in defer as defer is executed after we return
        if (deinit_status == .leak) std.testing.expect(false) catch @panic("\nGPA detected a memory leak!!!\n");
    }

    try list_posts(allocator);
    try query_post(allocator, 8);
}
```

</br>


But if you're implementing the `Request Handler` in a HTTP server, that means
each `Request Handler` (function) does a heap-allocation (`alloc` and `free`)
In a very high frequency, and that hurts the performance.

So, in that case, I prefer to use `Arean + FixedBufferAllocator` to handle
each HTTP request, as For short periods of small chunks of memory allocation,
stack-allocation is must faster.

</br>

