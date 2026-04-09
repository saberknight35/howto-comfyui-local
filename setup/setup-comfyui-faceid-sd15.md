# Setup ComfyUI + IP-Adapter FaceID (SD 1.5) di Windows

## Spesifikasi yang Dipakai
- OS: Windows
- GPU: NVIDIA GeForce RTX 5060 Laptop (8GB VRAM, sm_120 / Blackwell)
- Python: 3.13.x
- PyTorch: 2.12.0.dev (nightly) + cu128
- ComfyUI: v0.18.1

---

## Komponen yang Dibutuhkan

### 1. Software & Tools
| Komponen | Keterangan |
|---|---|
| Git | Untuk clone repo |
| Python | Runtime utama |
| Microsoft C++ Build Tools | Diperlukan untuk compile insightface |

### 2. Model Files

> 💡 **Kenapa ada banyak model terpisah?**
> Analoginya seperti dapur memasak: **Checkpoint** adalah chef-nya (yang punya semua keahlian masak), **VAE** adalah alat finishing (yang bikin plating jadi cantik), dan **LoRA** adalah resep tambahan (yang ngajarin chef skill baru tanpa harus ganti chef-nya). Masing-masing punya peran berbeda dan bisa dikombinasikan secara fleksibel.

---

#### 🧠 Checkpoint — "Otak Utama" AI
Checkpoint adalah model AI utama yang sudah ditraining dengan miliaran gambar. Di sinilah semua "pengetahuan" tentang cara menggambar tersimpan — anatomi manusia, pencahayaan, gaya artistik, tekstur, dll. Tanpa checkpoint, tidak ada yang bisa di-generate.

- Tiap checkpoint punya "kepribadian" berbeda (ada yang bagus untuk foto realistis, anime, landscape, dll)
- Ukurannya besar (1–7GB) karena menyimpan ratusan juta parameter
- Kamu bisa punya banyak checkpoint dan ganti-ganti sesuai kebutuhan

| File | Folder Tujuan | Keterangan | Link Download |
|---|---|---|---|
| `epicrealism_naturalSinRC1VAE.safetensors` | `models/checkpoints/` | Checkpoint SD 1.5 yang dioptimasi untuk foto realistis. Sudah include VAE bawaan (ditandai "VAE" di namanya). | CivitAI |

---

#### 🎨 VAE (Variational Autoencoder) — "Alat Finishing Warna & Ketajaman"
VAE adalah komponen yang mengubah "sketsa internal" AI (disebut latent space) menjadi gambar piksel yang bisa dilihat. Tanpa VAE, gambar yang dihasilkan akan terlihat buram, warnanya pudar, atau ada artefak aneh.

- Checkpoint biasanya sudah include VAE bawaan, tapi VAE eksternal bisa memberikan hasil yang lebih tajam dan warna lebih vivid
- Ukurannya kecil (~300MB) dibanding checkpoint
- Bisa di-swap tanpa ganti checkpoint — ini yang membuat arsitekturnya modular

| File | Folder Tujuan | Keterangan | Link Download |
|---|---|---|---|
| `vae-ft-mse-840000-ema-pruned.safetensors` | `models/vae/` | VAE standar dari Stability AI. Menghasilkan warna lebih hangat dan detail kulit lebih natural dibanding VAE bawaan beberapa checkpoint. | HuggingFace |

---

#### 👁️ CLIP Vision — "Mata" AI untuk Memahami Gambar
CLIP Vision adalah model yang bertugas "membaca" dan memahami isi sebuah gambar (bukan teks). Digunakan oleh IPAdapter untuk mengekstrak informasi visual dari foto referensi (misalnya foto wajah), lalu menginformasikannya ke proses generate.

| File | Folder Tujuan | Keterangan | Link Download |
|---|---|---|---|
| `CLIP-ViT-H-14-laion2B-s32B-b79K.safetensors` | `models/clip_vision/` | CLIP Vision encoder versi ViT-H, kompatibel dengan SD 1.5. Wajib untuk IPAdapter Plus/FaceID. Nama file harus persis seperti ini. | HuggingFace (rename dari `model.safetensors`) |

---

#### 🎭 IPAdapter — "Jembatan" antara Foto Referensi dan Generate
IPAdapter memungkinkan AI untuk mengambil informasi visual dari gambar referensi (misalnya wajah seseorang) dan menerapkannya ke hasil generate. Berbeda dari LoRA yang mengubah "gaya", IPAdapter bekerja secara real-time per-generate.

| File | Folder Tujuan | Keterangan | Link Download |
|---|---|---|---|
| `ip-adapter-faceid-plusv2_sd15.bin` | `models/ipadapter/` | Model IPAdapter khusus FaceID versi Plus V2 untuk SD 1.5. Mengekstrak identitas wajah dari foto referensi dan menerapkannya ke gambar baru. Lebih akurat dari versi V1. | HuggingFace h94/IP-Adapter-FaceID |

---

#### 🧬 LoRA (Low-Rank Adaptation) — "Resep Tambahan" untuk Checkpoint
LoRA adalah file kecil yang "menempel" di atas checkpoint dan mengubah atau menambah kemampuannya tanpa mengganti checkpoint itu sendiri. Analoginya: checkpoint adalah chef berbintang, LoRA adalah buku resep khusus yang diberikan ke chef tersebut.

- Ukurannya sangat kecil (50MB–300MB) dibanding checkpoint (1–7GB)
- Bisa dikombinasikan beberapa LoRA sekaligus
- Ada LoRA untuk skin realism, gaya artistik tertentu, pose spesifik, dll
- LoRA FaceID ini khusus dipasangkan dengan model IPAdapter FaceID — keduanya harus dipakai bersamaan

| File | Folder Tujuan | Keterangan | Link Download |
|---|---|---|---|
| `ip-adapter-faceid-plusv2_sd15_lora.safetensors` | `models/loras/` | LoRA pendamping wajib untuk IPAdapter FaceID Plus V2. Bertugas membantu checkpoint memahami "instruksi wajah" yang dikirim oleh IPAdapter. Tanpa LoRA ini, FaceID tidak akan bekerja dengan baik. | HuggingFace h94/IP-Adapter-FaceID |

---

#### 🔍 InsightFace buffalo_l — "Detektor Wajah"
buffalo_l bukan model Stable Diffusion — ini adalah model deteksi wajah dari library InsightFace. Tugasnya: mendeteksi, menganalisis, dan mengekstrak fitur wajah dari foto yang kamu upload, sebelum informasinya dikirim ke IPAdapter.

| File | Folder Tujuan | Keterangan | Link Download |
|---|---|---|---|
| `buffalo_l` (folder berisi 5 file .onnx) | `C:\Users\<user>\.insightface\models\` | Model deteksi wajah resolusi tinggi. Versi "large" — lebih akurat dari buffalo_m dan buffalo_s tapi lebih lambat. Berisi beberapa sub-model: deteksi landmark wajah, gender/age estimation, dan face recognition. | GitHub InsightFace releases |

### 3. Custom Nodes
| Node | Repo |
|---|---|
| ComfyUI_IPAdapter_plus | https://github.com/cubiq/ComfyUI_IPAdapter_plus |

### 4. Python Packages
| Package | Keterangan |
|---|---|
| insightface | Face detection & recognition |
| onnxruntime | Dependency insightface |

---

## Langkah-Langkah Setup

### Step 1 — Clone ComfyUI
```bash
cd D:\Projects
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI
```

### Step 2 — Buat Virtual Environment
```bash
python -m venv env-mycomfyui
env-mycomfyui\Scripts\activate
```

### Step 3 — Install Dependencies
```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### Step 4 — Install PyTorch (CUDA / Nightly untuk RTX 50 series)
Untuk GPU generasi lama (RTX 30/40):
```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126 --force-reinstall
```

Untuk GPU RTX 5060 (Blackwell / sm_120) — wajib nightly:
```bash
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu128 --force-reinstall
```

Verifikasi CUDA aktif:
```bash
python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
# Harus output: True
```

### Step 5 — Install Custom Node IPAdapter Plus
```bash
cd custom_nodes
git clone https://github.com/cubiq/ComfyUI_IPAdapter_plus.git
cd ..
```

### Step 6 — Install insightface
Pastikan sudah install Microsoft C++ Build Tools dulu (Desktop development with C++), lalu:
```bash
pip install insightface
pip install onnxruntime
```

### Step 7 — Download & Tempatkan Semua Model
Sesuaikan nama file dengan tabel komponen di atas, taruh di folder yang benar.

Struktur folder akhir:
```
ComfyUI/
├── models/
│   ├── checkpoints/
│   │   └── epicrealism_naturalSinRC1VAE.safetensors
│   ├── vae/
│   │   └── vae-ft-mse-840000-ema-pruned.safetensors
│   ├── clip_vision/
│   │   └── CLIP-ViT-H-14-laion2B-s32B-b79K.safetensors
│   ├── ipadapter/
│   │   └── ip-adapter-faceid-plusv2_sd15.bin
│   └── loras/
│       └── ip-adapter-faceid-plusv2_sd15_lora.safetensors
└── custom_nodes/
    └── ComfyUI_IPAdapter_plus/

C:\Users\<user>\.insightface\models\
└── buffalo_l\
    ├── 1k3d68.onnx
    ├── 2d106det.onnx
    ├── det_10g.onnx
    ├── genderage.onnx
    └── w600k_r50.onnx
```

### Step 8 — Jalankan ComfyUI
```bash
python main.py
# Buka browser: http://127.0.0.1:8188
```

---

## Workflow Node Graph (FaceID Plus V2)

```
Load Checkpoint ──────────────────────────────► model ─┐
                                                        │
Load IPAdapter Model (faceid-plusv2_sd15.bin) ─► ipadapter ─┐
                                                              │
Load Image (foto wajah) ──────────────────────► image ──────►│ IPAdapterFaceID ──► MODEL ──► KSampler ──► VAE Decode ──► Save Image
                                                              │
Load CLIP Vision (CLIP-ViT-H) ────────────────► clip_vision ─┤
                                                              │
IPAdapterInsightFaceLoader (provider: CUDA) ──► insightface ─┘

CLIP Text Encode (positive prompt) ──────────────────────────────────────► positive ─┐
CLIP Text Encode (negative prompt) ──────────────────────────────────────► negative ─┤ KSampler
Empty Latent Image (512x768 atau 768x1024) ──────────────────────────────► latent ───┘
```

---

## Node yang Dipakai di ComfyUI

| Node | Fungsi |
|---|---|
| `CheckpointLoaderSimple` | Load model utama (checkpoint) |
| `IPAdapterModelLoader` | Load model ip-adapter-faceid |
| `IPAdapterInsightFaceLoader` | Load insightface untuk deteksi wajah (pilih provider: CUDA) |
| `Load CLIP Vision` | Load CLIP Vision encoder |
| `Load Image` | Upload foto wajah referensi |
| `IPAdapterFaceID` | Node utama FaceID, gabungkan semua input |
| `CLIPTextEncode` | Positive & negative prompt |
| `EmptyLatentImage` | Ukuran canvas output |
| `KSampler` | Engine generate gambar |
| `VAEDecode` | Decode latent ke gambar |
| `SaveImage` | Simpan hasil |

---

## Tips & Catatan

- **CFG Scale:** Gunakan 5–7 untuk hasil lebih natural. Nilai tinggi (10+) bikin gambar plastik.
- **Steps:** 20 cukup untuk draft, 30–40 untuk kualitas lebih baik.
- **Weight FaceID:** Default 1.0, turunkan ke 0.7–0.8 jika wajah terlalu dominan mengubah komposisi.
- **Resolusi:** Minimal 512x768, lebih baik 768x1024. Makin besar makin lambat.
- **Model SD 1.5** punya batas realistis — untuk hasil lebih fotorealistik pertimbangkan upgrade ke SDXL atau Flux.
- **buffalo_l** harus ada isinya (5 file .onnx), bukan folder kosong.
- File `ip-adapter-faceid-plusv2_sd15.bin` **tidak di-rename** — biarkan tetap `.bin`.
- File CLIP Vision **harus** bernama persis `CLIP-ViT-H-14-laion2B-s32B-b79K.safetensors`.

---

## Next Steps (Advanced)
- [ ] Upgrade ke SDXL untuk kualitas lebih tinggi
- [ ] Coba Flux + PuLID untuk face swap generasi terbaru
- [ ] Eksplor LoRA tambahan untuk skin texture & realism
- [ ] Install ControlNet untuk kontrol pose & komposisi
- [ ] Coba upscaling (4x UltraSharp, RealESRGAN) untuk resolusi lebih tinggi
