# Return command errors

Open when returning command failures as Results and serializing custom Rust errors.

[Topic context](rust-command-error-serialization.md)

### Error Handling

When a handler might fail and has to be capable of reporting an error, make the function return a `Result`:

```rust
#[tauri::command]
fn login(user: String, password: String) -> Result<String, String> {
  if user == "tauri" && password == "tauri" {
    // resolve
    Ok("logged_in".to_string())
  } else {
    // reject
    Err("invalid credentials".to_string())
  }
}
```

Should the command yield an error the promise rejects; failing that, it resolves:

```javascript
invoke('login', { user: 'tauri', password: '0j4rijw8=' })
  .then((message) => console.log(message))
  .catch((error) => console.error(error));
```

Per the earlier note, anything a command returns — errors included — has to implement [`serde::Serialize`](https://docs.serde.rs/serde/trait.Serialize.html). That becomes awkward when you lean on error types from Rust's std library or from outside crates, since the bulk of error types skip that implementation. For the easy cases, lean on `map_err` to turn such errors into `String`:

```rust
#[tauri::command]
fn my_custom_command() -> Result<(), String> {
  std::fs::File::open("path/to/file").map_err(|err| err.to_string())?;
  // Return `null` on success
  Ok(())
}
```

As that approach is hardly idiomatic, you might prefer rolling your own error type that implements `serde::Serialize`. The example that follows leans on the [`thiserror`](https://github.com/dtolnay/thiserror) crate to assemble the error type; by deriving the `thiserror::Error` trait it lets you promote enums into error types. Its documentation covers the finer points.

```rust
// create the error type that represents all errors possible in our program
#[derive(Debug, thiserror::Error)]
enum Error {
  #[error(transparent)]
  Io(#[from] std::io::Error)
}

// we must manually implement serde::Serialize
impl serde::Serialize for Error {
  fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
  where
    S: serde::ser::Serializer,
  {
    serializer.serialize_str(self.to_string().as_ref())
  }
}

#[tauri::command]
fn my_custom_command() -> Result<(), Error> {
  // This will return an error
  std::fs::File::open("path/that/does/not/exist")?;
  // Return `null` on success
  Ok(())
}
```

One upside of a custom error type is that it lays every possible error out in the open, letting readers see at a glance which failures may occur. Down the line, that spares everyone — you included — a great deal of effort during review and refactoring.
