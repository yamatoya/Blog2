# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

https://poop.jp 用の静的サイトジェネレーター（C# / .NET 6）。`articles/` の Markdown を HTML に変換し、GitHub Actions（`.github/workflows/build.yml`）が master への push 時に `dotnet run` を実行して `gh-pages` ブランチへ公開する。

## コマンド

```bash
# サイト生成（入力ディレクトリと出力ディレクトリを引数で渡す）
dotnet run --project ./src/Blog2/Blog2.csproj -c Release -- ./articles ./publish

# ビルドのみ
dotnet build Blog2.sln
```

テストプロジェクトは存在しない。リンターの設定もない。

## 記事の書き方

- `articles/` に `YYYY-MM-DD<suffix>.md` という名前で配置する（例: `2022-09-05_flare01.md`）。ファイル名がこの形式に一致しないと生成時に例外が投げられる。
- 1行目は必ず `# タイトル` で始める（`#` で始まらないと例外）。1行目がタイトル、2行目以降が本文（Markdig で HTML 変換、生 HTML の埋め込み可）。
- ファイル名から URL が決まる: `YYYY-MM-DD_foo.md` → `https://poop.jp/YYYY/MM/DD_foo.html`
- `drafts/` は下書き置き場（生成対象外）。

## アーキテクチャ

- `src/Blog2/Program.cs` — 全ロジックが入った単一ファイルのトップレベルプログラム。処理の流れ:
  1. 入力ディレクトリの全 `.md` を並列で読み込み `Article` レコードに変換
  2. ルート / 年別（`YYYY/`）/ 月別（`YYYY/MM/`）の index ページを 15 記事ごとにページング生成（2ページ目以降は `2.html`, `3.html`...）
  3. 各記事の個別 HTML を生成
  4. 最新10件から RSS（`/feed/index.xml`）を生成
- HTML テンプレートはコード内の文字列（`BuildHtml` 関数）。サイドバー・フッター・OGP タグもここで組み立てる。
- CSS は `src/Blog2/style.css`。ビルド出力にコピーされ（csproj の `CopyToOutputDirectory`）、生成時に出力ルートへ `style.css` として書き出される。ライト/ダーク両対応（`prefers-color-scheme`）。コードハイライトは Prism（CDN, tomorrow テーマ）。
- `tools/WordPressExporter/` — WordPress の `wp_posts` ダンプ XML を記事 Markdown に変換する一回限りの移行ツール。通常は触らない。
