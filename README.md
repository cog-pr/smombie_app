# Smombie App - 歩きスマホ検知システム

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-1.51.0-FF4B4B?logo=streamlit&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.7.2-F7931E?logo=scikitlearn&logoColor=white)

**スマートフォンのセンサーデータを活用した歩きスマホ（Smombie）のリアルタイム検知・警告システム**

</div>

---

## 📖 概要

**Smombie App** は、スマートフォンの加速度センサーとジャイロスコープのデータを機械学習で分析し、歩きスマホ（Smartphone + Zombie = Smombie）を検知するWebアプリケーションです。

歩きスマホは重大な事故につながる危険な行動であり、本システムはその検知と警告によって安全な歩行を促進することを目的としています。

### 主な機能

-  **機械学習モデル作成** - ランダムフォレストを用いた歩行状態分類モデルの学習
-  **リアルタイム予測** - Phyphoxアプリと連携したリアルタイムセンサーデータの処理
-  **警告メール送信** - 歩きスマホ検知時の即時メール通知
-  **ユーザー管理** - アカウント登録・ログイン・設定カスタマイズ機能

---

##  システム構成

```
Smombie_App/
├── app/
│   ├── main.py                    # メインアプリケーション・ナビゲーション
│   ├── requirements.txt           # Python依存パッケージ
│   ├── .streamlit/
│   │   └── secrets.toml           # メール送信用認証情報（非公開）
│   ├── db/
│   │   └── app_db.db              # SQLiteユーザーデータベース
│   └── pages/
│       ├── top.py                 # ホームページ・ユーザー認証
│       ├── train_model.py         # モデル学習ページ
│       ├── realtime_data.py       # リアルタイム処理ページ
│       └── files/
│           └── checklist.pdf      # 実験報告書チェックリスト
├── data/
│   ├── train_data.xlsx            # 学習用データセット
│   ├── test_data.xlsx             # テスト用データセット
│   └── model.pkl                  # 学習済みモデル
└── 起動用.bat                     # Windows起動スクリプト
```

---

##  技術スタック

| カテゴリ | 技術 |
|---------|------|
| **フレームワーク** | Streamlit 1.51.0 |
| **機械学習** | scikit-learn 1.7.2 (RandomForest) |
| **データ処理** | pandas, numpy, scipy |
| **可視化** | matplotlib |
| **データベース** | SQLite (SQLAlchemy) |
| **メール送信** | yagmail |
| **センサーデータ取得** | Phyphox (外部アプリ) |

---

##  行動分類

本システムは、以下の3つの歩行状態を分類します：

| ラベル | 説明 |
|--------|------|
| **Stop** | 停止状態 |
| **Distracted Walking** | 歩きスマホ（危険状態） |
| **Not Distracted Walking** | 通常歩行 |

---

##  特徴量

センサーデータ（6軸：ax, ay, az, wx, wy, wz）から以下の統計量を抽出します：

- **基本統計量**: 平均(mean)、標準偏差(std)、最小値(min)、最大値(max)、中央値(median)
- **範囲**: レンジ(range)
- **四分位**: 第1四分位(q1)、第3四分位(q3)、四分位範囲(iqr)
- **分布形状**: 歪度(skew)、尖度(kurt)

合計 **66個の特徴量** を使用してモデルを学習します。

---

##  セットアップ

### 前提条件

- Python 3.10以上
- pipパッケージマネージャー

### インストール手順

1. **リポジトリをクローン**
   ```bash
   git clone https://github.com/cog-pr/Smombie_App.git
   cd Smombie_App
   ```

2. **依存パッケージをインストール**
   ```bash
   pip install -r app/requirements.txt
   ```

3. **メール設定（オプション）**
   
   `app/.streamlit/secrets.toml` ファイルを作成し、以下の形式で設定します：
   ```toml
   [email]
   address = "your-email@gmail.com"
   app_key = "your-app-password"

   [connections.my_db]
   url = "sqlite:///./app/db/app_db.db"
   ```

   > ⚠️ **注意**: Gmailを使用する場合は[アプリパスワード](https://support.google.com/accounts/answer/185833)を生成してください。

---

##  使い方

### アプリケーションの起動

#### ローカル環境で起動

```bash
streamlit run ./app/main.py
```

または、Windowsの場合は `起動用.bat` をダブルクリックしてください。

#### Dockerで起動

Docker Composeを使用する場合（推奨）：

```bash
# secrets.tomlを作成
cp app/.streamlit/secrets.toml.example app/.streamlit/secrets.toml
# 中身を編集してメール設定を入力

# ビルド＆起動
docker-compose up --build

# バックグラウンドで起動する場合
docker-compose up -d --build
```

Dockerだけで起動する場合：

```bash
# イメージをビルド
docker build -t smombie-app .

# コンテナを起動
docker run -p 8501:8501 smombie-app
```

起動後、ブラウザで `http://localhost:8501` にアクセスしてください。

### 基本的なワークフロー

#### 1. ユーザー登録・ログイン
- ホームページでアカウントを作成
- ログイン後、各種設定をカスタマイズ可能

#### 2. モデル作成ページ
1. 学習用Excelファイル（train_data.xlsx）をアップロード
   - 必須列: `time`, `ax`, `ay`, `az`, `wx`, `wy`, `wz`, `class`
2. テスト用Excelファイル（test_data.xlsx）をアップロード
3. 「実行する」ボタンでモデル学習・評価を実行
4. 混同行列とアニメーションGIFで結果を確認
5. 学習済みモデル（.pkl）をダウンロード

#### 3. リアルタイム処理ページ
1. スマートフォンで[Phyphox](https://phyphox.org/)アプリを起動
2. リモートアクセスを有効化してIPアドレスを取得
3. アプリにIPアドレスを入力
4. 学習済みモデル（.pkl）をアップロード
5. 「実行する」ボタンでリアルタイム検知を開始

---

## 設定可能なパラメータ

ログイン後、設定画面から以下のパラメータをカスタマイズできます：

### スライディングウィンドウ
| パラメータ | デフォルト値 | 説明 |
|-----------|-------------|------|
| ウィンドウサイズ | 60 | 一度に分析するデータポイント数 |
| ストライド | 30 | ウィンドウのスライド幅 |

### ランダムフォレスト
| パラメータ | デフォルト値 | 説明 |
|-----------|-------------|------|
| n_estimators | 400 | 決定木の数 |
| max_depth | 6 | 決定木の最大深さ |

### メール通知
| オプション | 説明 |
|-----------|------|
| 即時送信 | 歩きスマホ検知時に即座に警告メールを送信 |
| 計測終了後 | 計測終了後にまとめて通知 |

---

## Phyphox連携

[Phyphox](https://phyphox.org/) は、スマートフォンの各種センサーデータを取得できる無料アプリです。

### 対応デバイス
- **iPhone**: iOS版 Phyphox
- **Android**: Android版 Phyphox

### 使用するセンサー
- 線形加速度センサー（X, Y, Z軸）
- ジャイロスコープ（X, Y, Z軸）

---

## 参考文献

本システムは以下の研究・文献を参考にしています：

- 京都大学（2024）「歩きスマホによる内因性転倒リスクの増大」
- 東京消防庁「歩きスマホ等に係る事故に注意！」
- J-STAGE「歩行中のスマートフォン操作と視線の危険性」
- モバイル社会研究所（2024）歩きスマホに関する実態調査

---

## プロジェクト背景

本プロジェクトは、**認知情報科学実験** の一環として開発されました。歩きスマホの危険性を科学的に検証し、センサーデータを活用した検知システムの実現可能性を探求しています。

---

## スクリーンショット

| ホームページ | モデル作成 | リアルタイム処理 |
|-------------|-----------|-----------------|
| ログイン・歩きスマホの危険性についての解説 | 学習済みモデルの作成・評価 | センサーデータのリアルタイム分析 |

---

## コントリビューション

バグ報告や機能提案は、Issueを通じてお知らせください。プルリクエストも歓迎します。

---

## お問い合わせ

質問やフィードバックがありましたら、Issueを作成してください。

---

<div align="center">

**⚠️ 歩きスマホは危険です。スマホを見る前に、まず足を止めましょう。**

</div>
