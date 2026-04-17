# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ

- **GitHub**: https://github.com/halsistm/UFO
- **push先**: `git push origin main`（originは上記URLに設定済み）

## プロジェクト概要

**UFO Abductor - OPEN WORLD** は、ブラウザで動作する単一HTMLファイルのゲームです。Three.js r128 を CDN から読み込み、サーバー不要でローカルファイルとして開けます（ただし音声ファイルのフェッチに HTTP サーバーが必要）。

## 起動方法

```bash
# ローカルサーバーで起動（音声が必要な場合）
python3 -m http.server 8080
# → ブラウザで http://localhost:8080/index.html を開く

# 音声不要ならファイルを直接ブラウザにドロップするだけでも動作する
```

## ファイル構成

すべてのゲームロジックは `index.html` 1ファイルに収まっている。外部ファイルは音声 (.mp3) と 3D モデル (.glb) のみ。

| ファイル | 内容 |
|---|---|
| `index.html` | ゲーム本体（HTML + CSS + JS 全込み、約2500行） |
| `Helicopter.glb` | ヘリコプター敵モデル |
| `F22_shape_only_decimated.glb` | 戦闘機敵モデル |
| `taikuho.glb` | 対空砲敵モデル |
| `*.mp3` | SE・BGM |

## コードアーキテクチャ（index.html 内の構成）

`window.load` イベント内にすべてのコードがある。セクションはコメント `// ======` で区切られている。

```
SOUND SYSTEM         SoundManager クラス（Web Audio API）
WORLD / CHUNK SYSTEM チャンク単位のプロシージャル地形生成
  - getBiome()       fBm ノイズでバイオームを決定（11種類）
  - buildChunk()     チャンクごとに建物・地形・ターゲットを生成
3D MODEL FACTORIES   建物・木・乗り物などのプロシージャルメッシュ
UFO                  プレイヤー機（Three.js Group、コード内で直接構築）
ENEMY GLB LOADING    Helicopter / F22 / taikuho の GLB プリロードとスケール設定
TARGET SYSTEM        誘拐ターゲット（人間・動物・乗り物）の定義と生成
  - TARGET_DEFS[]    全ターゲット種別（type, title, score, カラー）
  - createHuman()    等、タイプ別メッシュ生成関数
CHUNK UPDATE         UFO 位置に追従してチャンクの動的ロード/アンロード
ANIMATE              requestAnimationFrame ループ（UFO 操作・物理・UI 更新）
```

## 重要な数値・定数

- **CHUNK_SIZE = 64** / **BLOCK_SIZE = 8**（地形グリッドの基本単位）
- **UFO 直径 ≈ 13 units**（CylinderGeometry radius 6.5）、初期高度 y=55
- **敵スケール**（GLB の元サイズが大きいため小さい値を設定）
  - Helicopter: `setScalar(0.18)`
  - Fighter:    `setScalar(0.25)`
  - Taikuho:    `setScalar(0.1)`
- **スポーン距離**: ヘリ 180–240u、戦闘機 220–300u、対空砲 100–180u（すべて UFO からの水平距離）

## ゲームプレイ仕様

- **操作**: WASD / 矢印キー or 仮想スティック（モバイル）で UFO 移動
- **ABDUCT ボタン**: 真下のターゲットをトラクタービームで吸い込む
- **ATTACK ボタン**: 攻撃ビームを発射して敵を撃破
- **バイオーム**: 11 種類（OCEAN / BEACH / GRASSLAND / FARMLAND / FOREST / MOUNTAIN / SUBURB / URBAN / INDUSTRIAL / RIVER / MILITARY）。MILITARY ゾーンでは敵が増援スポーン
- **Fast モード**: 1.2 秒以上移動継続で速度が最大 20.8 unit/frame まで加速

## よくある修正箇所

- **敵のサイズ変更** → `loadEnemyGLBs()` 関数内の `gltf.scene.scale.setScalar()`
- **ターゲット追加** → `TARGET_DEFS` 配列にオブジェクトを追記し、`buildTargetMesh()` に case を追加
- **バイオーム調整** → `getBiome()` 関数内の閾値
- **敵のHP・ダメージ** → `spawnHelicopter()` / `spawnFighter()` / `spawnTaikuho()` の `hp` フィールドと `animate()` 内の `damage` 変数
