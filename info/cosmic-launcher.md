# Output (xbps-src pkg cosmic-launcher)
```
   Compiling pop-launcher-service v1.2.7 (https://github.com/pop-os/launcher/#5b868510)
   Compiling console-subscriber v0.5.0
   Compiling tracing-journald v0.3.2
   Compiling switcheroo-control v0.1.0 (https://github.com/pop-os/dbus-settings-bindings#507e342c)
warning: unexpected `cfg` condition value: `a11y`
   --> src/components/list.rs:307:11
    |
307 |     #[cfg(feature = "a11y")]
    |           ^^^^^^^^^^^^^^^^
    |
    = note: expected values for `feature` are: `console`, `default`, and `wgpu`
    = help: consider adding `a11y` as a feature in `Cargo.toml`
    = note: see <https://doc.rust-lang.org/nightly/rustc/check-cfg/cargo-specifics.html> for more information about checking conditional configuration
    = note: `#[warn(unexpected_cfgs)]` on by default

warning: field `dummy_id` is never read
   --> src/app.rs:162:5
    |
142 | pub struct CosmicLauncher {
    |            -------------- field in this struct
...
162 |     dummy_id: window::Id,
    |     ^^^^^^^^
    |
    = note: `CosmicLauncher` has a derived impl for the trait `Clone`, but this is intentionally ignored during dead code analysis
    = note: `#[warn(dead_code)]` (part of `#[warn(unused)]`) on by default

warning: variant `Surface` is never constructed
   --> src/app.rs:186:5
    |
166 | pub enum Message {
    |          ------- variant in this enum
...
186 |     Surface(surface::Action),
    |     ^^^^^^^
    |
    = note: `Message` has derived impls for the traits `Clone` and `Debug`, but these are intentionally ignored during dead code analysis

warning: `cosmic-launcher` (bin "cosmic-launcher") generated 3 warnings
    Finished `release` profile [optimized] target(s) in 6m 32s
=> cosmic-launcher-1.0.14_1: skipping check (XBPS_CHECK_PKGS is disabled) ...
=> cosmic-launcher-1.0.14_1: running pre-install hook: 00-libdir ...
=> cosmic-launcher-1.0.14_1: running pre-install hook: 02-script-wrapper ...
=> cosmic-launcher-1.0.14_1: running pre-install hook: 98-fixup-gir-path ...
=> cosmic-launcher-1.0.14_1: running do_install ...
=> ERROR: cosmic-launcher-1.0.14_1: vinstall: cannot find 'target/release/cosmic-launcher'...
=> ERROR: cosmic-launcher-1.0.14_1: do_install: 'return 1' exited with 1
=> ERROR:   in _noglob_helper() at common/environment/setup/install.sh:12
=> ERROR:   in _vbin() at common/environment/setup/install.sh:69
=> ERROR:   in _noglob_helper() at common/environment/setup/install.sh:12
=> ERROR:   in do_install() at srcpkgs/cosmic-launcher/template:25
```