# oculibis-slides

[oculibis](../oculibis)（複数リポジトリの Pull Request を Codex CLI で自動レビューし、GitHub App 名義の 1 コメントを更新し続けるローカル常駐 bot）を社内向けに紹介する [Marp](https://marp.app/) スライドです。

スライド本体は [`slides.md`](./slides.md) です。構成は「前半＝動機の物語 / 後半＝設計判断（＋失敗談）」の二段構え。発表の冒頭で実リポジトリに `/oculibis review` を投げ、レビューが返ってくる様子をライブで挟む段取りを前提にしています。

## セットアップ

```sh
pnpm install
```

## ビルド / プレビュー

```sh
pnpm preview     # ブラウザでライブプレビュー（Marp の -p）
pnpm watch       # build/slides.html を変更監視つきで生成
pnpm build       # build/slides.html を生成（画像も build/ にコピー）
pnpm build:pdf   # build/slides.pdf を生成
```

Marp for VS Code 拡張を使う場合は `slides.md` をそのままプレビューできます。

> PDF / 画像出力時に `slides.md` のローカル画像（`images/` 配下）を読み込むため、Marp CLI には `--allow-local-files` を渡しています（`build:pdf` に設定済み）。

## 図（mermaid）について

Marp は標準では mermaid を描画できないため、`diagrams/*.mmd` を [mermaid-cli](https://github.com/mermaid-js/mermaid-cli) で SVG に事前レンダリングして `images/` に置き、スライドからは画像として参照しています。

```sh
pnpm diagrams
```

図を編集したら再生成します（Chromium が必要。`diagrams/puppeteer.json` の `executablePath` で参照する Chrome を指定。環境に合わせて書き換えてください）。

## 差し替えが必要な素材（プレースホルダ）

`images/` 配下に以下を用意すると本番用になります（未配置でもビルドは通り、代替のテキスト枠が出ます）。

- `comment-initial.png` … 初回レビューの managed comment スクショ
- `comment-resolved.png` … 追加コミット後に resolved が追記された状態のスクショ
- `overview.svg` … `pnpm diagrams` で生成（コミット対象）

## 発表当日のデモ段取り（メモ）

1. タイトル表示のタイミングで、対象の public repo の PR に `/oculibis review` コメントを投稿
2. 1 分以内に bot が `eyes` リアクション（受理）を付けるのを確認
3. スライドの切れ目で「レビュー来ましたね」とライブをチラ見。`+1` = publish 完了
4. トラブル時は `images/comment-*.png` のスクショで代替
