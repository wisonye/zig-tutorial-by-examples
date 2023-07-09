# Test fixed dependencies locally

When your project depends on another zig project as dependencies and the
following situation happens:

- The dependency project has bug
- You can fix that bug
- But you can't commit the fix to the project master branch

Ok, that means you can't compile or run your project until the dependencies
get fixed and commit to its master branch, so what you can do about this?

</br>

Here are the steps you can solve that problem until they fix that:

- Clone that repo and fix the bug

- Use `git archive` to create the archive of the entire source tree in `.tar.gz` format

    For example:

    ```bash
    git archive --format=tar.gz -o zap-musl-fixed-0.0.11.tar.gz --prefix=zap-musl-fixed-0.0.11/ HEAD
    ```

    So, you got the `zap-musl-fixed-0.0.11.tar.gz` archive file which can be
    served via the local HTTP server.

    </br>

- Use any HTTP server to serve that file

    For example:

    ```bash
    python3 -m http.server 8001
    ```

    </br>

- Modify your `build.zig.zon` to use that served archive file

    ```zon
    .{
        .name = "zig-demo-service",
        .version = "0.0.1",

        .dependencies = .{
            // zap v0.1.8-pre
            .zap = .{
                //
                // .url = "https://github.com/zigzap/zap/archive/refs/tags/v0.1.8-pre.tar.gz",
                //

                //
                // Only for local fixed archive
                //
                .url = "http://localhost:8001/zap-musl-fixed-0.0.11.tar.gz",
                .hash = "12201247239025627931235019c070f759e0648291133e8bca86ef55492ec2edc721",
            }
        }
    }
    ```

    </br>

    Then your `zig build` will work:)



