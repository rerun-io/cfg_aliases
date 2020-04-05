# CFG Aliases

CFG Aliases is a tiny utility to help save you a lot of effort with long winded `#[cfg()]` checks. For example:

**Cargo.toml:**

```toml
[build-dependencies]
cfg_aliases = "0.1.0-alpha.0"
```

**build.rs:**

```rust
use cfg_aliases::cfg_aliases;

fn main() {
    // Setup cfg aliases
    cfg_aliases! {
        // Platforms
        wasm: { target_arch = "wasm32" },
        android: { target_os = "android" },
        macos: { target_os = "macos" },
        linux: { target_os = "linux" },
        // Backends
        surfman: { all(unix, feature = "surfman", not(wasm)) },
        glutin: { all(feature = "glutin", not(wasm)) },
        wgl: { all(windows, feature = "wgl", not(wasm)) },
        dummy: { not(any(wasm, glutin, wgl, surfman)) },
    }
}
```

Now that we have our aliases setup we can use them just like you would expect:

```rust
#[cfg(wasm)]
println!("This is running in WASM");

#[cfg(surfman)]
{
    // Do stuff related to surfman
}

#[cfg(dummy)]
println!("We're in dummy mode, specify another feature if you want a smarter app!");
```

This greatly improves what would otherwise look like this without the aliases:

```rust
#[cfg(target_arch = "wasm32")]
println!("We're running in WASM");

#[cfg(all(unix, feature = "surfman", not(target_arch = "22")))]
{
    // Do stuff related to surfman
}

#[cfg(not(any(
    target_arch = "wasm32",
    all(unix, feature = "surfman", not(target_arch = "wasm32")),
    all(windows, feature = "wgl", not(target_arch = "wasm32")),
    all(feature = "glutin", not(target_arch = "wasm32")),
)))]
println!("We're in dummy mode, specify another feature if you want a smarter app!");
```

You can also use the `cfg!` macro or combine your aliases with other checks using `all()`, `not()`, and `any()`. Your aliases are genuine `cfg` flags now!

```rust
if cfg!(glutin) {
    // use glutin
} else {
    // Do something else
}

#[cfg(all(glutin, surfman))]
compile_error!("You cannot specify both `glutin` and `surfman` features");
```

### Attribution and Thanks

- Thanks to my God and Father who led me through figuring this out and to whome I owe everything.
- Thanks to @Yandros on the Rust forum for [showing me][sm] some crazy macro hacks!
- Thanks to @sfackler for [pointing out][po] the way to make cargo add the cfg flags.
- Thanks to the authors of the [`tectonic_cfg_support::target_cfg`] macro from which most of the cfg attribute parsing logic is taken from. Also thanks to @ratmice for [bringing it up][bip] on the Rust forum.

[`tectonic_cfg_support::target_cfg`]: https://docs.rs/tectonic_cfg_support/0.0.1/src/tectonic_cfg_support/lib.rs.html#166-298
[po]: https://users.rust-lang.org/t/any-such-thing-as-cfg-aliases/40100/2
[bip]: https://users.rust-lang.org/t/any-such-thing-as-cfg-aliases/40100/13
[sm]: https://users.rust-lang.org/t/any-such-thing-as-cfg-aliases/40100/3