# Copilot Instructions for `direnv` (Rust/Nix)

## 概要

- **direnv** は、ディレクトリごとにシェル環境変数を管理するツールです。Rust で実装され、Nix を用いたビルド・開発環境が整備されています。
- `.envrc`/`env.sh` ファイルを bash で実行し、YAML 形式で環境変数を出力します（JSON もサポート）。
- 標準ライブラリのシェル関数群は `src/stdlib/stdlib.sh` に定義され、Rust から `include_str!` で読み込まれます。

## 主要ディレクトリ・ファイル

- `src/main.rs` … CLI エントリーポイント。コマンド/オプションのパースと標準出力。
- `src/lib.rs` … モジュールエクスポート、バージョン情報取得。
- `src/config/` … TOML 設定ファイルのロード・構造体定義。
- `src/env/` … 環境変数の管理・差分計算ロジック。
- `src/stdlib/` … シェル関数群（`stdlib.sh`）と Rust からの参照。
- `default.nix`, `shell.nix`, `nix/` … Nix によるビルド・開発環境定義。

## ビルド・開発ワークフロー

- **Nix環境推奨**: `nix-shell` で `shell.nix` を利用し、Rust toolchain（cargo, rustc, clippy, rustfmt, rls）が自動導入されます。
- **ビルド**: `cargo build` または Nix 経由で `nix-build`。
- **テスト**: Rust 標準の `cargo test` を利用。
- **依存管理**: `Cargo.toml`（Rust）、`nix/sources.nix`（Nixパッケージ）。

## コーディング・設計パターン

- **環境変数管理**: `env::Env` 型（`HashMap<OsString, OsString>`）で表現。差分計算は `env::diff` 関数を参照。
- **設定ファイル**: TOML形式。`config::load` でパス指定してロード。
- **シェル関数拡張**: `stdlib.sh` に追加し、Rust から `STDLIB` 経由で利用。
- **エラー処理**: Rust 標準の `Result` 型。CLIはエラー時に `stderr` 出力し `exit(1)`。

## 重要な慣習・注意点

- **YAML出力**: `.envrc`/`env.sh` の出力は YAML 形式。`env:` キーで環境変数を定義。
- **PATHの扱い**: `env: { PATH: "XXX" }` で PATH を上書き。`null` で削除。
- **互換性**: `direnv-export` コマンドは旧バージョン互換用。
- **ログ出力**: シェル関数 `log_status`, `log_error` で `$DIRENV_LOG_FORMAT` に従い stderr へ出力。

## 参考例

- `src/env/mod.rs` の `diff` 実装は、環境変数の差分計算ロジックの典型例。
- `src/config/mod.rs` の `Config` 構造体は TOML 設定のマッピング例。
- `src/stdlib/stdlib.sh` でシェル関数を追加・拡張可能。

## 外部連携・依存

- **Nix**: `nix/sources.nix` で外部パッケージ管理。`naersk` で Rust crate ビルド。
- **serde/serde_json/toml**: Rust のデータ構造シリアライズ/デシリアライズに利用。
