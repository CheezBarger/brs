[![brs on crates.io][cratesio-image]][cratesio]
[![brs on docs.rs][docsrs-image]][docsrs]

[cratesio-image]: https://img.shields.io/crates/v/brs.svg
[cratesio]: https://crates.io/crates/brs
[docsrs-image]: https://docs.rs/brs/badge.svg
[docsrs]: https://docs.rs/brs

Interfaces for reading and writing Brickadia save files.

Aims to be able to read all previous versions just like the game,
but only write the newest version of the format.

# Usage

## Reading

First, create a reader from any
[`Read`](https://doc.rust-lang.org/std/io/trait.Read.html)
source, such as a file or buffer.

```rust
let reader = brs::Reader::new(File::open("village.brs")?)?;
```

Brickadia save files have information split into sections ordered
such that one can extract simple information
without needing to parse the entire file.

This library surfaces this by strictly enforcing the way that data is read
and made available at the type level; you can't go wrong.

To continue, reading the first header gets you basic information.
For details on what is available, see
[`HasHeader1`](https://docs.rs/brs/*/brs/read/trait.HasHeader1.html).

```rust
use brs::HasHeader1;
let reader = reader.read_header1();
println!("Brick count: {}", reader.brick_count());
println!("Map: {}", reader.map());
```

The next header contains data less likely to be relevant for simpler
introspection, but rather things such as tables for loading bricks.
See [`HasHeader2`](https://docs.rs/brs/*/brs/read/trait.HasHeader2.html).

```rust
use brs::HasHeader2;
let reader = reader.read_header2();
println!("Mods: {:?}", reader.mods());
println!("Color count: {}", reader.colors().len());
// Properties from header 1 are still available:
println!("Description: {}", reader.description();
```

After both headers have been read, you may now iterate over the bricks.
See [`Brick`](https://docs.rs/brs/*/brs/struct.Brick.html).

```rust
for brick in reader.iter_bricks()? {
    let brick = brick?;
    println!("{:?}", brick);
}
```

You may retain access to the header information while getting the iterator:

```rust
let (rdr, bricks) = reader.iter_bricks_and_reader()?;
```

## Writing

Writing save files isn't as fancy, for now you simply just put all the data
in the [`WriteData`](https://docs.rs/brs/*/brs/struct.WriteData.html) struct and pass it to
[`write_save`](https://docs.rs/brs/*/brs/fn.write_save.html) along with a
[`Write`](https://doc.rust-lang.org/std/io/trait.Write.html) destination.

```rust
let data = brs::WriteData {
    map: String::from("Plate"),
    description: String::from("A quaint park full of ducks and turkeys."),
    // ...
};
brs::write_save(&mut File::create("park.brs")?, &data)?;
```
