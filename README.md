# safeconv

A small library of **bounds-checked integer conversions** for Go.

Narrowing an integer in Go (`int` to `uint32`, `int64` to `int16`, and so on) silently wraps on
overflow, which is how a large positive value turns into a small or negative one with no
warning. `safeconv` gives every common narrowing conversion a named function that clamps to the
target type's range instead, so the result is always the closest representable value. It also
satisfies gosec's G115 check without scattering `//nolint` directives through calling code.

Module `piko.sh/safeconv`. Go 1.27. Apache-2.0. No dependencies outside the standard library.

## Install

```sh
go get piko.sh/safeconv
```

## Usage

```go
package main

import (
	"fmt"
	"math"

	"piko.sh/safeconv"
)

func main() {
	fmt.Println(safeconv.IntToUint32(-5))            // 0
	fmt.Println(safeconv.IntToUint32(math.MaxInt64)) // 4294967295
	fmt.Println(safeconv.Int64ToInt16(40000))        // 32767

	// Generic conversion for values whose concrete type varies by platform.
	fmt.Println(safeconv.ToUint64(int32(-1))) // 0
}
```

## What it provides

- **Clamping conversions** (the default): negative values clamp to 0 for unsigned targets,
  values above the target's maximum clamp to that maximum, and values below a signed target's
  minimum clamp to that minimum. For example `IntToUint64`, `IntToUint32`, `IntToInt16`,
  `Int64ToInt32`, `Uint64ToInt`, `Int32ToUint32`, `RuneToByte` and `UintptrToInt64`.
- **`ToUint64[T]`**: a generic clamp from any integer type, useful for values such as
  `syscall.Statfs_t` fields whose type differs between platforms.
- **`Must*` variants** (`MustIntToUint8`, `MustIntToUint16`, `MustIntToInt16`,
  `MustUintToUint8`, `MustUint8ToInt8`, `MustInt8ToUint8`): panic on out-of-range input, for
  call sites where overflow can only mean a programming error.
- **Bit-pattern reinterpretation** (`Uint16ToInt16`, `Int16ToUint16`,
  `Int64ToUint64Reinterpret`, `Uint64ToInt64Reinterpret`): keep the bits and change the
  type, for hashing and wire formats.
- **Explicit truncation** (`Uint64ToByteTruncate`, `IntToUint32Truncate`): keep the low
  bits, named so the intent is visible at the call site.

Every function is pure and safe for concurrent use. The package builds and passes its tests
on 32-bit and 64-bit targets, including `GOOS=js GOARCH=wasm`.

## Development

```sh
go test -race ./...
GOARCH=386 go test ./...
golangci-lint run
```

safeconv was originally developed as part of the [Piko](https://github.com/piko-sh/piko) web
framework.
