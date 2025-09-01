# 自作Vtuberモデル環境構築ガイド

[Talking Head(?) Anime from A Single Image 3: Now the Body Too](https://tech-blog.cloud-config.jp/2023-08-24-vtuber-with-ai-first-part)の記事を参考にした、自作Vtuberモデル作成環境です。

## 構築済み環境

### システム要件
- Windows 11
- NVIDIA GPU (RTX 4070以上推奨)
- Docker Desktop
- Python 3.13
- CUDA 13.0

### インストール済みツール

1. **Python仮想環境** (`vtuber_env`)
   - PyTorch 2.6.0 + CUDA 12.4
   - SciPy, Matplotlib, wxPython
   - rembg (背景除去ツール)

2. **talking-head-anime-3-demo**
   - アニメーション生成ツール
   - 学習済みモデル一式 (796MB)

3. **Stable Diffusion WebUI Docker**
   - キャラクター画像生成環境

## 使用方法

### 1. キャラクター画像の生成

Stable Diffusion WebUIを起動：
```bash
cd stable-diffusion-webui-docker
docker compose --profile auto up --build
```

ブラウザで `http://localhost:7860` にアクセスしてキャラクター画像を生成

### 2. 背景除去

生成した画像の背景を除去：
```bash
vtuber_env\Scripts\activate
python -c "
from rembg import remove
from PIL import Image
import io

# 画像を読み込み
input_image = Image.open('input.png')

# 背景除去
output = remove(input_image)

# 保存
output.save('output_nobg.png')
"
```

### 3. アニメーション生成

#### 手動ポーザー
表情や動きを手動で調整：
```bash
vtuber_env\Scripts\activate
cd talking-head-anime-3-demo
python tha3/app/manual_poser.py
```

#### iFacialMocap連携（iOSデバイス必要）
リアルタイム表情追跡：
```bash
vtuber_env\Scripts\activate
cd talking-head-anime-3-demo
python tha3/app/ifacialmocap_puppeteer.py
```

### 4. Jupyter Notebook版
ブラウザで手動ポーザーを使用：
```bash
vtuber_env\Scripts\activate
cd talking-head-anime-3-demo
jupyter notebook
```
`manual_poser.ipynb` を開いて実行

## 入力画像の要件

アニメーションが正常に動作するための画像条件：

- 解像度: 512 x 512
- アルファチャンネル必須（PNG形式）
- 単一のヒューマノイドキャラクター
- 正面向きで直立
- 手は頭から離れた下方位置
- 頭部は画像上半分の中央128x128領域内
- 背景は完全透明（アルファ値0）

## システム構成の詳細

### モデルバリアント
talking-head-anime-3-demoには4つのモデルが利用可能：

- `standard_float`: 最大・最遅・最高精度（デフォルト）
- `separable_float`: 分離可能版（float）
- `standard_half`: 標準版（half precision）
- `separable_half`: 分離可能版（half precision）

使用例：
```bash
python tha3/app/manual_poser.py --model separable_half
```

### ディレクトリ構成
```
vtuber/
├── vtuber_env/                          # Python仮想環境
├── talking-head-anime-3-demo/           # アニメーションツール
│   ├── data/
│   │   ├── images/                      # サンプル画像
│   │   └── models/                      # 学習済みモデル
│   │       ├── standard_float/
│   │       ├── standard_half/
│   │       ├── separable_float/
│   │       └── separable_half/
│   └── tha3/                           # メインプログラム
└── stable-diffusion-webui-docker/       # Stable Diffusion環境
```

## トラブルシューティング

### Docker関連
- Docker Desktopが起動していない場合は手動で起動
- WSL2が必要な場合は `wsl --install` で有効化

### GPU関連
- CUDA対応GPUが必要
- ドライバーが最新であることを確認
- `nvidia-smi` でGPU認識を確認

### Python環境
- 仮想環境の有効化を忘れずに
- Windows版wxPythonはPython 3.10で問題があるため3.8-3.9または3.11以降を使用

## 参考リンク

- [元記事](https://tech-blog.cloud-config.jp/2023-08-24-vtuber-with-ai-first-part)
- [Talking Head Anime 3 プロジェクト](https://pkhungurn.github.io/talking-head-anime-3/)
- [stable-diffusion-webui-docker](https://github.com/AbdBarho/stable-diffusion-webui-docker)
- [talking-head-anime-3-demo](https://github.com/pkhungurn/talking-head-anime-3-demo)