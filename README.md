# scursor

[![CI](https://github.com/stepfunc/scursor/workflows/CI/badge.svg)](https://github.com/stepfunc/scursor/actions)
[![Crates.io](https://img.shields.io/crates/v/scursor.svg)](https://crates.io/crates/scursor)
[![Documentation](https://docs.rs/scursor/badge.svg)](https://docs.rs/scursor)

A secure, no-std cursor library for safe binary data reading and writing with transaction support.

## Features

- **Memory Safe**: `forbid(unsafe_code)` - no unsafe code allowed
- **no_std Compatible**: Works in embedded and constrained environments
- **Panic-Free**: All operations return `Result` types instead of panicking
- **Recursion-Free**: Stack-safe implementation suitable for embedded systems
- **Transaction Support**: Atomic operations with rollback on failure
- **Endianness Support**: Built-in little-endian and big-endian operations
- **Comprehensive Types**: Support for u8, u16, u24, u32, u48, u64, i16, i32, i64, f32, f64

## Usage

### Reading Binary Data

```rust
use scursor::ReadCursor;

let data = [0xCA, 0xFE, 0xBA, 0xBE];
let mut cursor = ReadCursor::new(&data);

// Read individual bytes
let first_byte = cursor.read_u8()?;

// Read multi-byte values in different endianness
let value_le = cursor.read_u16_le()?;  // Little-endian u16
let remaining = cursor.read_all();     // Get remaining bytes as slice

// Transaction support - rollback on error
cursor.transaction(|cur| {
    let a = cur.read_u8()?;
    let b = cur.read_u8()?;
    // If this fails, cursor position is restored
    let c = cur.read_u16_le()?;
    Ok((a, b, c))
})?;
```

### Writing Binary Data

```rust
use scursor::WriteCursor;

let mut buffer = [0u8; 64];
let mut cursor = WriteCursor::new(&mut buffer);

// Write individual bytes and multi-byte values
cursor.write_u8(0xFF)?;
cursor.write_u32_le(0xDEADBEEF)?;
cursor.write_f64_le(3.14159)?;

// Transaction support with rollback
cursor.transaction(|cur| {
    cur.write_u16_le(0x1234)?;
    cur.write_u16_le(0x5678)?;
    // If this fails, all writes in transaction are rolled back
    cur.write_u32_le(0x9ABCDEF0)
})?;

// Get written data
let written_data = cursor.written();
```

### Position and Seeking

```rust
use scursor::{ReadCursor, WriteCursor};

// Read cursor positioning
let mut read_cursor = ReadCursor::new(&[1, 2, 3, 4, 5]);
println!("Position: {}", read_cursor.position());
println!("Remaining: {}", read_cursor.remaining());

// Write cursor seeking
let mut buffer = [0u8; 10];
let mut write_cursor = WriteCursor::new(&mut buffer);
write_cursor.seek_to(5)?;
write_cursor.write_u8(0xFF)?;
```

## Safety Guarantees

- **No Panics**: All operations return `Result` types
- **Bounds Checking**: Automatic bounds checking prevents buffer overflows
- **Integer Overflow Protection**: All arithmetic operations check for overflow
- **Transaction Atomicity**: Failed transactions leave cursors in original state

## Supported Types

### Integer Types

- `u8`, `u16`, `u24` (as u32), `u32`, `u48` (as u64), `u64`
- `i16`, `i32`, `i64`

### Floating Point

- `f32`, `f64` (IEEE-754 format)

### Endianness

- Little-endian: `_le` suffix
- Big-endian: `_be` suffix
