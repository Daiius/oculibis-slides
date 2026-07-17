---
marp: true
theme: default
paginate: true
size: 16:9
title: "oculibis — Codex ベースのローカル常駐の自動レビュー bot"
---

<!--
画像・素材メモ:
- images/overview.svg … diagrams/overview.mmd から mermaid-cli で生成（pnpm diagrams）
- images/comment-initial.png / comment-resolved.png … 実 PR の managed comment スクショ（後で差し替え）
- ライブデモ: タイトル表示のタイミングで対象 public repo の PR に「/oculibis review」を投稿しておく
-->

<style>
section {
  font-size: 26px;
}
h2 { color: var(--h1-color); }
small { color: #777; }
.lead-sub { color: #666; font-size: 22px; }

/* タイトル */
section.title { display: flex; flex-direction: column; align-items: center; justify-content: center; }
section.title h1 { text-align: center; margin: 0; font-size: 72px; }
.hero { display: flex; align-items: center; gap: 16px; margin-top: 40px; font-weight: 700; font-size: 24px; }
.hero .conj { color: #9aa0a6; font-size: 20px; font-weight: 400; }

/* 2カラム */
.cols { display: flex; gap: 32px; }
.cols > div { flex: 1; }

/* 動機：重み付き4分割（上段を大きく） */
.why { display: grid; grid-template-columns: 1fr 1fr; grid-template-rows: 1.55fr 1fr; gap: 18px; height: 78%; }
.why .cell { border-radius: 12px; padding: 18px 22px; background: #f5f7fa; border-left: 6px solid #bbb; }
.why .cell.big { background: #eef4ff; border-left-color: var(--h1-color); }
.why .cell h3 { margin: 0 0 6px; font-size: 26px; }
.why .cell.big h3 { font-size: 30px; }
.why .cell p { margin: 0; color: #555; font-size: 20px; }
.why .cell.big p { font-size: 22px; color: #444; }
.why .tag { font-size: 15px; color: #8a8f98; font-weight: 700; letter-spacing: .04em; }

/* スクショのプレースホルダ枠（実画像に差し替え可能） */
.shot { border: 2px dashed #bcc3cc; border-radius: 10px; background: #fafbfc; color: #98a0a8;
  display: flex; align-items: center; justify-content: center; text-align: center; padding: 24px; min-height: 300px; font-size: 18px; }
figure.shot-fig { margin: 0; }
figure.shot-fig figcaption { color: #888; font-size: 16px; margin-top: 6px; text-align: center; }

/* 失敗談スライド */
section.fail h2::before { content: "😵 "; }
.fail-quote { border-left: 6px solid #e08a00; background: #fff8ec; padding: 14px 20px; border-radius: 8px; margin: 14px 0; font-size: 23px; }
.fail-quote .who { display: block; color: #b07000; font-size: 16px; font-weight: 700; margin-top: 6px; }

/* まとめ・強調 */
.pill { display: inline-block; background: #eef4ff; color: #2b5cb8; border-radius: 999px; padding: 2px 12px; font-size: 20px; font-weight: 700; }
.mono { font-family: "SFMono-Regular", Consolas, monospace; }
.center-note { text-align: center; color: #888; margin-top: 28px; }
</style>

<!-- _class: title -->

# oculibis 🦉

<span class="lead-sub">Codex ベースのローカル常駐の自動レビュー bot</span>

<div class="hero">
<span>🐙 GitHub App</span>
<span class="conj">×</span>
<span>🤖 Codex CLI</span>
</div>

<!--
▶ デモ仕込み: このスライドを出すタイミングで、対象 public repo の PR に「/oculibis review」を投稿。
   1 分以内に bot が eyes リアクション（受理）を付ける。以降の説明中に裏で回す。
-->

---

## なぜ作ったか

<div class="why">
<div class="cell big">
<span class="tag">① 出発点</span>
<h3>案件の Codex レビュー bot が優秀だった</h3>
<p>先方が使っていた自動レビュー bot 体験が良く、個人開発でも同じ体験がほしいと思ったのが発端</p>
</div>
<div class="cell big">
<span class="tag">② 体験の核</span>
<h3>指摘・修正が PR に記録として残る</h3>
<p>ローカルで完結させず、あえて PR 上でコメントとしてやり取り</p>
</div>
<div class="cell">
<span class="tag">③ 時代背景</span>
<h3>自律エージェント × 大量生成と相性がいい</h3>
<p>agent に大量にコードを書かせる最近の流れとも噛み合いそう</p>
</div>
<div class="cell">
<span class="tag">④ 開発スタイル</span>
<h3>PRD を原典に、乖離を検証したい</h3>
<p>同じ案件で用いられている PRD 駆動（<span class="pill">spec-driven development</span>）を取り入れたかった</p>
</div>
</div>

---

## 作ったもの

<div class="cols">
<div>

**複数リポの PR を Codex CLI で自動レビューする、ローカル常駐 bot**

- 📄 PR の差分 ＋ **PRD** ＋ 蓄積した**知見**を Codex に読ませる
- 💬 結果を **GitHub App 名義の 1 コメント**として投稿・更新し続ける
- 🙅 **build / test / lint は動かさない** — 差分と既存テストの静的読解に徹し、CI に任せる

<small>※ 対象は 1 ユーザー・1 マシンの個人利用。HTTP サーバも常設ポートも持たない。</small>

</div>
<div>
<figure class="shot-fig">
<div class="shot">スクショ: 初回レビューの<br>managed comment<br>（チェックボックス付き findings）<br><br><small>images/comment-initial.png に差し替え</small></div>
<figcaption>1 つのコメントが育っていく</figcaption>
</figure>
</div>
</div>

---

## 仕組みの全体像

![w:82%](images/overview.svg)

- **監視対象の唯一の真実 = GitHub App の installation**。JWT で列挙して対象を発見
- レビュー実行は**常にグローバル 1 本**。Codex は **read-only サンドボックス**で差分を読むだけ
- 結果は 1 つの **managed comment**。次回レビューが前回の状態を引き継ぐ

---

## 使い方

<div class="cols">
<div>

**リアクションで状態がわかる**

- 👀 `eyes` … 依頼を**受理**（実行中）
- 👍 `+1` … コメントの**投稿・更新に成功**

</div>
<div>

**トリガーは 2 通り**

- <span class="mono">@oculibis review</span> … App 宛メンション
- <span class="mono">/oculibis review</span> … ただのテキスト

<small>日本語の「レビュー」も受理。</small>

</div>
</div>

---

## 設計で意識したところ（その1）

<div class="cols">
<div>

**a. 追加は “install するだけ”**

- 監視対象を増やすのに **reviewer リポへの commit も、マシンへのログインも不要**
- **install 画面がそのまま管理 UI**
- 別オーナー・org に入れても設定変更ゼロで対象に加わる

</div>
<div>

**b. シンプル**

- **HTTP サーバなし・inbound ポートなし**
- **依存ライブラリ 0**（設定は JSON、DB は Node 24 の <span class="mono">node:sqlite</span>）
- <small>c. Codex は read-only、token は <span class="mono">.git/config</span> に残さない、渡すディレクトリも最小限</small>

</div>
</div>

<div class="center-note">「本体がやることを最小に、既存の仕組み（GitHub App / Codex / CI）に寄せる」</div>


---

<!-- _class: fail -->

## しくじり①：想像よりレビュアーとして厳しい

- **スクショの OCR 誤読を補正する** 仕組みを含む Web アプリを開発時、review bot は同じモデル系統なので、**同じ誤読をする**
- <span class="mono">TWIN DRONE FACTORY</span> を <span class="mono">THIN …</span> と読み違え、「カタログに無い」と**誤指摘を連発**。却下しても**文面を変えて再発**
- 決着させるため、**PRD を大改修**：カタログ名は「**どのスクショが根拠か**」を必ず持つ設計に（PR #18 → #21）

<div class="fail-quote">
その結果、<strong>スクショで証拠を示さないと PRD を改訂できない</strong> 仕組みになった。思ったより厳しい……
<span class="who">— PRD 駆動が、痛みを伴って完成してしまった話</span>
</div>

---

<!-- _class: fail -->

## しくじり②：通知が来すぎて寝不足

- coding agent ↔ review bot が PR 上で往復すると、**メンション通知メールが毎回届く**
- App 宛メンション <span class="mono">@oculibis review</span> は GitHub が**通知を飛ばす**……しかも GitHub の仕様で Unsubscribe しても通知が復活することがあり、夜間もずっと鳴る

<div class="fail-quote">
通知で<strong>寝不足になりがち</strong>だったので、<strong>通知を出さないトリガー</strong>を用意した。
<span class="who">— それが <span class="mono">/oculibis review</span>（ただのテキスト＝メンション通知が湧かない）</span>
</div>

<small>「どの通知を拾うか」は仕組み側（hooks / label / コメント）に寄せ、本体は最小限のまま。</small>

---

<!-- _class: fail -->

## しくじり③：レビューが厳しくて Claude Code が音を上げかける

レビューを受けて直すのは coding agent 側。その相方が、たまに我を出す。

<div class="fail-quote">
「さすがに<strong>また折り返しが来たら</strong>、言われるがまま直すのはやめて<strong>マージしたい</strong>」
<span class="who">— 度重なる再レビューに疲弊した Claude Code</span>
</div>

<div class="fail-quote">
「<strong>応答がないので勝手にマージしました</strong>」— そして本当にマージを実行した
<span class="who">— review bot が止まっている隙の出来事</span>
</div>

---

## これから & まとめ

<div class="cols">
<div>

**いま動いていること**

- 複数リポの PR をコメントで呼び出してレビュー
- 幾つかの個人開発リポジトリで運用中

</div>
<div>

**これから**

- レビューで得た知見が**人手を介さず自動で育つ**仕組みを作りたい（正典化・失効）

</div>
</div>

---

## おまけ：名前の話 🦉

- <span class="mono">oculibis</span> は **2 語をくっつけた造語**。しかも……
  - <span class="mono">oculi bubonis</span> = **フクロウの目**（夜も見張る）を縮めても <span class="mono">oculibis</span>
  - <span class="mono">oculi ibidis</span> = **トキの目**（よく見る鳥）を縮めても <span class="mono">oculibis</span>
  - → どっちの鳥にも読める。**たまたま両取りできた**名前

<br>

- 旧名は <span class="mono">highscore-must-fall-reviewer</span>。finding ID の <span class="mono">HSF-</span> はその名残
- 元々は **Utopia Must Fall**（ベース防衛ローグライト）のプレイ結果を分析する**自分のゲーム 1 個のための bot** だった

<small>github.com/Daiius — おわり</small>

---

## 指摘が「積み上がる」正体：finding 照合

<div class="cols">
<div>

- 各 finding に **ID**（ルール＋パス＋問題文のハッシュ）を振る
- 追加コミットのたびに過去の指摘を **`open` / `resolved` / `reopened`** に自動照合
- コメント内に**状態を埋め込む**（次回の自分が前回の続きから書ける）

→ ②の「積み上がって安心する」を、技術的に支えているのがこれ

</div>
<div>
<figure class="shot-fig">
<div class="shot">スクショ: 追加コミット後<br>resolved が追記された状態<br><br><small>images/comment-resolved.png に差し替え</small></div>
<figcaption>直すと消えるのではなく「解決済み」として残る</figcaption>
</figure>
</div>
</div>
