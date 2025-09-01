# Vtuberアプリケーション使用ガイド

このドキュメントは、AI技術を用いた自作Vtuberモデルの作成・使用方法を説明します。

## システム概要

3つの主要コンポーネントから構成されています：

1. **Stable Diffusion WebUI** - キャラクター画像生成
2. **rembg** - 背景除去処理
3. **talking-head-anime-3-demo** - アニメーション生成

## 前提条件

### システム要件
- Windows 11
- NVIDIA GPU (RTX 4070以上推奨)
- Docker Desktop
- Python 3.13
- CUDA 13.0

### インストール済みコンポーネント
- Python仮想環境 (`vtuber_env`)
- PyTorch 2.6.0 + CUDA 12.4
- 必要なPythonライブラリ一式
- 学習済みモデル (796MB)

## 使用方法

### 1. キャラクター画像の生成

#### Stable Diffusion WebUIの起動
```bash
cd stable-diffusion-webui-docker
docker compose --profile auto up --build
```

#### WebUIへのアクセス
1. ブラウザで `http://localhost:7860` を開く
2. プロンプト入力欄にキャラクターの特徴を記述
3. 画像生成を実行

#### 推奨プロンプト例
```
1girl, anime style, standing, front view, full body, 
simple background, high quality, detailed face, 
clean art style, solo character
```

#### 画像生成のベストプラクティス
- **解像度**: 512x512を推奨
- **アスペクト比**: 1:1（正方形）
- **構図**: 正面向き、全身が含まれる
- **背景**: シンプルな背景を選択
- **ポーズ**: 直立、手は体の両側に自然に配置

### 2. 背景除去処理

#### rembgを使用した自動背景除去
```bash
# 仮想環境をアクティベート
vtuber_env\Scripts\activate

# 背景除去の実行
python -c "
from rembg import remove
from PIL import Image

# 画像を読み込み
input_image = Image.open('generated_character.png')

# 背景除去
output_image = remove(input_image)

# 透明背景のPNGとして保存
output_image.save('character_nobg.png')
print('背景除去完了: character_nobg.png')
"
```

#### 手動での背景除去
高品質な結果が必要な場合：
1. GIMP、Photoshop等の画像編集ソフトを使用
2. アルファチャンネル付きPNG形式で保存
3. 背景部分を完全透明（アルファ値0）に設定

### 3. アニメーション生成

#### 手動ポーザーの使用
リアルタイムで表情・姿勢を調整：

```bash
# 仮想環境をアクティベート
vtuber_env\Scripts\activate

# プロジェクトディレクトリに移動
cd talking-head-anime-3-demo

# 手動ポーザーを起動
python tha3/app/manual_poser.py
```

##### 手動ポーザーの機能
- **表情制御**: 目の動き、口の形状、眉の位置
- **頭部回転**: 上下左右の頭の向き
- **体の動き**: 肩の回転、呼吸による胸の動き
- **リアルタイム調整**: スライダーによる直感的操作

#### iFacialMocap連携（iOSデバイス必要）
顔の動きをリアルタイムでキャラクターに反映：

```bash
# 仮想環境をアクティベート
vtuber_env\Scripts\activate

# プロジェクトディレクトリに移動
cd talking-head-anime-3-demo

# iFacialMocap連携ツールを起動
python tha3/app/ifacialmocap_puppeteer.py
```

##### 設定手順
1. iOSデバイスでiFacialMocapアプリを起動
2. 表示されたIPアドレスをメモ
3. PCのアプリケーション画面で「Capture Device IP」にIPアドレスを入力
4. 「START CAPTURE!」ボタンをクリック
5. 顔の動きがリアルタイムで反映されることを確認

#### Jupyter Notebook版
ブラウザベースでの操作：

```bash
# 仮想環境をアクティベート
vtuber_env\Scripts\activate

# プロジェクトディレクトリに移動
cd talking-head-anime-3-demo

# Jupyter Notebookを起動
jupyter notebook
```

1. ブラウザで開かれたページから`manual_poser.ipynb`を選択
2. 2つのセルを順番に実行
3. ページ下部にGUIが表示される

### 4. 入力画像の品質要件

#### 必須要件
- **解像度**: 512 x 512ピクセル
- **ファイル形式**: アルファチャンネル付きPNG
- **キャラクター数**: 単一のヒューマノイドキャラクターのみ
- **向き**: 正面向きで直立
- **手の位置**: 頭から離れた下方位置
- **頭部位置**: 画像上半分の中央128x128領域内
- **背景**: 完全透明（アルファ値0）

#### 品質向上のためのガイドライン
- **顔の詳細**: 目、鼻、口が明確に描かれている
- **髪型**: 複雑すぎない髪型を推奨
- **服装**: シンプルな服装が安定した結果を生む
- **ライティング**: 均等な照明、強い影は避ける

## モデルバリアント

システムには4つの精度・速度バリアントがあります：

| バリアント | サイズ | 速度 | 精度 | 推奨用途 |
|-----------|--------|------|------|----------|
| `standard_float` | 最大 | 最遅 | 最高 | 最終品質重視 |
| `separable_float` | 中 | 中 | 高 | バランス型 |
| `standard_half` | 中 | 中 | 中 | 軽量版 |
| `separable_half` | 最小 | 最速 | 標準 | リアルタイム用 |

### バリアント選択例
```bash
# 高速処理用
python tha3/app/manual_poser.py --model separable_half

# 最高品質用
python tha3/app/manual_poser.py --model standard_float
```

## パフォーマンス最適化

### GPU使用率の最適化
1. **CUDA使用確認**
   ```bash
   nvidia-smi
   python -c "import torch; print(torch.cuda.is_available())"
   ```

2. **メモリ管理**
   - 不要なアプリケーションを終了
   - VRAMの使用状況を監視
   - 必要に応じて軽量モデルを選択

### Docker最適化
```bash
# Docker Desktopリソース設定
# 設定 > Resources > Advanced
# Memory: 8GB以上
# CPUs: 4コア以上
# GPU: 有効化
```

## トラブルシューティング

### よくある問題

#### 1. キャラクターが正常にアニメーションしない
**症状**: 顔や体の動きが不自然、または動かない

**解決法**:
- 入力画像の品質要件を再確認
- 画像の解像度を512x512に調整
- 背景が完全に透明になっているか確認
- キャラクターの顔が指定領域内にあるか確認

#### 2. GPU認識エラー
**症状**: CUDA not available、GPU使用率0%

**解決法**:
```bash
# ドライバー更新確認
nvidia-smi

# PyTorch CUDA確認
python -c "import torch; print(torch.cuda.is_available()); print(torch.version.cuda)"

# 仮想環境の再構築（必要時）
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124 --force-reinstall
```

#### 3. Dockerコンテナ起動エラー
**症状**: Docker compose失敗、ポート競合

**解決法**:
```bash
# ポート使用状況確認
netstat -an | findstr :7860

# Docker完全クリーンアップ
docker system prune -a
docker volume prune

# 再ビルド
docker compose --profile auto up --build --force-recreate
```

#### 4. Python仮想環境エラー
**症状**: ModuleNotFoundError、パッケージ認識不可

**解決法**:
```bash
# 仮想環境の再有効化
vtuber_env\Scripts\deactivate
vtuber_env\Scripts\activate

# パッケージ再インストール
pip install --upgrade pip
pip install -r requirements.txt  # もしくは個別インストール
```

#### 5. iFacialMocap接続エラー
**症状**: iOS デバイスとの接続失敗

**解決法**:
- 同一Wi-Fiネットワークに接続確認
- ファイアウォール設定の確認
- iPho×neの画面スリープ無効化
- アプリケーションの再起動

### ログの確認方法
```bash
# Python アプリケーションのログ
python tha3/app/manual_poser.py > app.log 2>&1

# Docker ログ
docker compose logs

# システムリソース監視
# タスクマネージャー > パフォーマンス > GPU
```

## 応用的な使用方法

### カスタムキャラクターの作成
1. **オリジナルアートワーク作成**
   - 専用の画像生成プロンプトを開発
   - LoRAやDreamBoothでの追加学習
   - 一貫したキャラクターデザインの維持

2. **表情パターンの記録**
   - 手動ポーザーでの設定を保存
   - よく使う表情のプリセット作成
   - JSON形式での設定バックアップ

### 配信・録画での活用
1. **OBS Studio連携**
   - Window Captureでアプリケーションを取り込み
   - Green Screen (Chroma Key) での背景透過
   - 複数シーンでの切り替え

2. **リアルタイム配信**
   - iFacialMocapでの表情追跡
   - separable_halfモデルでの軽量動作
   - 低遅延設定での配信最適化

## ライセンス・著作権

- **talking-head-anime-3-demo**: MIT License
- **学習済みモデル**: Creative Commons Attribution 4.0 International License
- **Stable Diffusion**: 各モデルのライセンスに従う
- **生成画像**: 使用したモデルおよび元画像の権利に準拠