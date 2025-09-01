# 自作Vtuberモデル作成プロジェクト

AI技術を用いた自作Vtuberモデル作成環境です。

## ドキュメント

- **[Vtuberアプリケーション使用ガイド](./docs/VTUBER_APPLICATION_GUIDE.md)** - メインアプリケーションの詳細な使用方法

## クイックスタート

### 初回セットアップ

1. **リポジトリクローン**:
   ```bash
   git clone https://github.com/NakamuMomoka/vmodel-generator.git
   cd vmodel-generator
   ```

2. **Python仮想環境作成**:
   ```bash
   python -m venv vtuber_env
   
   # Windows
   vtuber_env\Scripts\activate
   
   # macOS/Linux
   source vtuber_env/bin/activate
   ```

3. **依存関係インストール**:
   ```bash
   pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
   pip install scipy matplotlib wxpython rembg
   ```

### 基本的な使用手順

1. **画像生成**: Stable Diffusion WebUIでキャラクター画像を作成
2. **背景除去**: rembgで背景を透明化  
3. **アニメーション**: talking-head-anime-3-demoで動きを追加

## システム要件

### 推奨環境
- **OS**: Windows 11, macOS, Ubuntu 20.04+
- **GPU**: NVIDIA GPU (RTX 4070以上推奨、CUDA対応)
- **Python**: 3.8 - 3.11 (3.10推奨)
- **Docker**: Docker Desktop (Stable Diffusion用)
- **メモリ**: 16GB以上
- **ストレージ**: 10GB以上の空き容量

### 主要コンポーネント

1. **Python仮想環境** (`vtuber_env`)
   - PyTorch + CUDA対応
   - SciPy, Matplotlib, wxPython
   - rembg (背景除去ツール)

2. **talking-head-anime-3-demo**
   - アニメーション生成ツール
   - 学習済みモデル一式 (796MB)

3. **Stable Diffusion WebUI Docker**
   - キャラクター画像生成環境

## 使用方法

詳細な手順については[Vtuberアプリケーション使用ガイド](./docs/VTUBER_APPLICATION_GUIDE.md)を参照してください。

## プロジェクト構成

```
vmodel-generator/
├── vtuber_env/                          # Python仮想環境
├── talking-head-anime-3-demo/           # アニメーション生成ツール
├── stable-diffusion-webui-docker/       # 画像生成環境
├── docs/                               # ドキュメント
└── .gitignore                          # Git設定
```

## ライセンス

- **talking-head-anime-3-demo**: MIT License
- **学習済みモデル**: Creative Commons Attribution 4.0 International License  
- **その他コンポーネント**: 各プロジェクトのライセンスに従う