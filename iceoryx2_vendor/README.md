# iceoryx2_vendor

`fibril_sim_runtime` が Fibril Simulation のプロセス境界に使う iceoryx2 の C++ ランタイムをビルドして入れます。

- 既定の iceoryx2 のリリース: `0.9.3`
- 既定で要求する Rust toolchain の最小バージョン: `1.85.0`
- Fibril 内で使う transport: `iox2::ServiceType::Ipc`

リリースは configure 時に `FIBRIL_ICEORYX2_VERSION` で変えられます。その場合はソースアーカイブの
`FIBRIL_ICEORYX2_SOURCE_SHA256` も合わせて変えてください。より新しい Rust を要求するリリースを選ぶときは
`FIBRIL_ICEORYX2_MINIMUM_RUST_VERSION` も合わせて設定します。

ビルドには `cargo` と `rustc` (1.85.0 以上) が必要です。Ubuntu 24.04 の apt の cargo は 1.75 で足りないので、
`rustup` で入れます (`apt install rustup && rustup default 1.85.0`)。

シミュレーションのプロトコルと、安定した payload の型 ID は Fibril 側で持ちます。iceoryx2 は同じホスト内の
プロセス間 transport、service discovery、共有メモリの確保と配送にだけ使います。
