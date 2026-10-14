# External Repository

このレポジトリにあるdependency.reposにより得られたパッケージを、部内aptリポジトリに登録するためのスクリプトです。

このレポジトリに直接置いている vendor パッケージも、同じワークフローでビルドして登録します。

| パッケージ | 中身 | 使う側 |
| --- | --- | --- |
| `fibril_mujoco_vendor` | MuJoCo（`opt/fibril_mujoco_vendor` に入る） | fibril_simulation |
| `iceoryx2_vendor` | iceoryx2 の C++ ランタイム（`opt/iceoryx2_vendor` に入る）。ビルドに Rust 1.85 以上が要る | fibril_simulation |

ワークフローは、`dependency.repos` か、これらのディレクトリを変えた main への push で動きます。
