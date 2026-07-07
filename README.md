# Laporan Hasil BB-ACT pada Robot SO101

Pada proyek _imitation learning_ ini, digunakan model VLA (Vision-Language-Action) dengan algoritma **BB-ACT** (Behavioral Cloning with Action Chunking and Transformers) pada robot SO101 untuk melakukan tugas memindahkan balok merah ke tempat persegi panjang berwarna coklat.

---

## 1. Pengertian BB-ACT

#### A. Behavior Cloning

**Behavior cloning** atau peniruan perilaku adalah metode pembelajaran yang menentukan tindakan agen dengan cara meniru contoh dari pakar (_expert demonstrations_). Pendekatan ini dapat diselesaikan melalui _machine learning_ dengan metode terawasi, seperti klasifikasi.

Metode _behavior cloning_ terbukti efektif untuk tugas yang kompleks, seperti locomotion pada robot berkaki dua. Namun, metode ini membutuhkan dataset yang besar dan kurang baik jika diterapkan pada lingkungan di luar variasi contoh yang pernah dilatih.

_Data set Aggregation_ (**DAgger**) berupaya mengatasi sebagian masalah tersebut dengan menambahkan demonstrasi pakar ke dalam kebijakan yang telah dipelajari selama proses pelatihan berlangsung, sehingga kebijakan hasil belajar dan kebijakan pakar saling melengkapi. Meski demikian, DAgger cukup sulit diterapkan karena membutuhkan contoh pakar yang dikumpulkan sepanjang proses pelatihan.

#### B. Action Chunking with Transformers

**Action Chunking with Transformers (ACT)** adalah algoritma _behavior cloning_ yang menggunakan **Conditional Variational Autoencoder (CVAE)** untuk memodelkan berbagai kondisi, lalu dikombinasikan dengan _transformer_ untuk memprediksi urutan aksi (_action chunks_) berdasarkan masukan multimodal.

Dengan merepresentasikan aksi sebagai keadaan tujuan yang harus dicapai manipulator, metode agregasi temporal dapat digunakan untuk menggabungkan prediksi aksi dari beberapa langkah waktu sebelumnya melalui rata-rata berbobot. Penggabungan beberapa prediksi ini membantu mengurangi masalah _compounding error_ dan respons yang tidak stabil terhadap keadaan di luar distribusi data pelatihan.

Walaupun model masih dapat memilih aksi yang kurang tepat, aksi yang telah diprediksi pada langkah sebelumnya akan memberikan pengaruh penyeimbang pada aksi akhir. Selain itu, struktur jaringan _transformer_ mendukung berbagai masukan multimodal, seperti _text prompt_, sehingga meningkatkan ketahanan kerangka kerja ACT.

---

## 2. Dataset

Dataset yang digunakan untuk proses training dapat diakses melalui link dibawah ini:

**Link Dataset:**
https://huggingface.co/datasets/Kamna0321/so101_persepsi_robot

Dataset ini direkam menggunakan framework **LeRobot Dataset v3.0** dan berisi demonstrasi gerakan robot untuk satu jenis tugas (single-task imitation learning). Data terdiri atas observasi kamera, state robot, serta aksi (joint command) pada setiap frame.

#### Informasi Dataset

| Parameter            | Nilai              |
| -------------------- | ------------------ |
| Robot                | SO-101 Follower    |
| Dataset Version      | v3.0               |
| Total Task           | 1                  |
| Total Episode        | 61                 |
| Total Frame          | 36.498             |
| FPS                  | 30                 |
| Total Durasi Rekaman | ±20 menit          |
| Split                | Train (61 Episode) |

---

#### Struktur Data

Dataset memiliki beberapa feature utama yang digunakan selama training.

#### 1. Action

Merupakan target output yang harus diprediksi oleh model VLA.

| Joint             |
| ----------------- |
| shoulder_pan.pos  |
| shoulder_lift.pos |
| elbow_flex.pos    |
| wrist_flex.pos    |
| wrist_roll.pos    |
| gripper.pos       |

- Tipe data : float32
- Shape : (6,)

#### 2. Observation State

Berisi kondisi aktual seluruh joint robot pada setiap frame.

| Joint             |
| ----------------- |
| shoulder_pan.pos  |
| shoulder_lift.pos |
| elbow_flex.pos    |
| wrist_flex.pos    |
| wrist_roll.pos    |
| gripper.pos       |

- Tipe data : float32
- Shape : (6,)

#### 3. Observation Images

Dataset menggunakan dua kamera RGB.

| Kamera       | Resolusi  |
| ------------ | --------- |
| Front Camera | 640 × 480 |
| Wrist Camera | 640 × 480 |

Spesifikasi video:

- Codec : AV1
- Pixel Format : yuv420p
- FPS : 30
- Channel : RGB (3 Channel)

---

#### Struktur Episode

Dataset terdiri dari **61 episode** dengan durasi yang hampir seragam.

| Statistik          | Nilai       |
| ------------------ | ----------- |
| Episode Terpendek  | 19.93 detik |
| Episode Terpanjang | 19.97 detik |
| Rata-rata Durasi   | 19.94 detik |
| Median             | 19.93 detik |
| Standar Deviasi    | 0.02 detik  |

Dengan FPS sebesar **30**, setiap episode memiliki sekitar **600 frame**.

---

#### Metadata Tambahan

Selain data utama, setiap frame juga memiliki metadata berikut:

- timestamp
- frame_index
- episode_index
- task_index
- index

Metadata tersebut digunakan untuk sinkronisasi data selama proses training dan evaluasi.

---

#### Struktur Penyimpanan Dataset

Dataset disimpan menggunakan format **LeRobot Dataset v3**.

```
data/
└── chunk-xxx/
    └── file-xxx.parquet
```

Video disimpan secara terpisah.

```
videos/
├── front/
│   └── chunk-xxx/file-xxx.mp4
└── wrist/
    └── chunk-xxx/file-xxx.mp4
```

---

### 3. Training

#### Model

- **License:** apache-2.0
- **Robot type:** so_follower
- **Cameras:** wrist, front
- **Policy type:** act

---

#### Input & Output

**Input**

| Fitur                    | Tipe   | Bentuk        |
| ------------------------ | ------ | ------------- |
| observation.state        | STATE  | (6,)          |
| observation.images.wrist | VISUAL | (3, 480, 640) |
| observation.images.front | VISUAL | (3, 480, 640) |

**Output**

| Fitur  | Tipe   | bentuk |
| ------ | ------ | ------ |
| action | ACTION | (6,)   |

---

#### Konfigurasi Training

| Konfigurasi     | Nilai |
| --------------- | ----- |
| Training steps  | 20000 |
| Batch size      | 32    |
| Optimizer       | adamw |
| Learning rate   | 1e-05 |
| Seed            | 1000  |
| LeRobot version | 0.5.2 |

---

berikut hasil training model

- Model Nur : https://huggingface.co/anisamsrh/rp_policy
- Model Wildan : https://huggingface.co/wild3005/rp_policy

---

<!--
## 4. Evaluasi Model

Isi dengan hasil evaluasi/metrik dari training di sini -->

## 4. Observasi Hasil Inference

Berikut adalah hasil observasi _inference_ pada robot SO101:
Robot berhasil mengambil balok merah dan memindahkannya ke area persegi panjang berwarna coklat, sesuai dengan gerakan yang dilatih sebelumnya. Namun, terdapat gerakan patah patah saat robot bergerak yang kemungkinan besar perlu mentuning PID gerakan robotnya, dan robot perlu dibantu sedikit saat mengambil balok dikarenakan _servo gripper_ tidak dapat menutup secara maksimal akibat adanya kerusakan kecil pada servo tersebut.

### Video Hasil

Berikut adalah dokumentasi video hasil pengujian (_inference_):

**1. Hasil Training Nur**  
<video src="./Video/HasilNur.mp4" controls width="100%"></video>

**2. Hasil Training Wildan**  
<video src="./Video/HasilWildan.mp4" controls width="100%"></video>

Atau video dapat diakses melalui tautan Google Drive berikut:  
 **[Video Hasil di Google Drive](https://drive.google.com/drive/folders/1kwrHN5rREQnvqet9mAZuFPpuiiKF5_q-?usp=sharing)**

---

## Langkah-Langkah Menjalankan Proyek

Berikut adalah panduan instalasi dan penggunaan untuk menjalankan _policy_ pada robot.

### 1. Instalasi Prasyarat

**Install Miniforge**

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

**Setup Environment Conda**

```bash
conda create -y -n lerobot python=3.12
conda activate lerobot
```

**Install FFmpeg**

```bash
conda install ffmpeg -c conda-forge
```

### 2. Instalasi LeRobot

**Clone Repository dan Install**

```bash
git clone https://github.com/huggingface/lerobot.git
cd lerobot
pip install lerobot
pip install 'lerobot[feetech]' # Diperlukan untuk motor servo SO-101
```

Setelah menginstal LeRobot pada sistem operasi, langkah selanjutnya adalah menjalankan perintah pada terminal untuk menggunakan robot SO101. Untuk penjelasan yang lebih lengkap, silakan merujuk pada file `AGENT_GUIDE.md` yang terdapat pada repositori LeRobot yang telah diunduh.

### 3. Persiapan Menjalankan Robot

Sebelum menjalankan perintah ke LeRobot, diperlukan untuk _login_ akun Hugging Face pada terminal, dan menginstal _Git LFS_ (Large File Storage) untuk menarik model atau dataset.

```bash
huggingface-cli login
# Atau dengan perintah: hf auth login

git lfs install && git lfs pull
```

### 4. Eksekusi pada Robot

**Menemukan Port Robot**

```bash
lerobot-find-port
```

> **Catatan:** macOS biasanya menggunakan `/dev/tty.usbmodem...`; Linux menggunakan `/dev/ttyACM0` (mungkin memerlukan akses dengan `sudo chmod 666 /dev/ttyACM0`).

**Kalibrasi Robot**

```bash
lerobot-calibrate --robot.type=so101_follower --robot.port=/dev/ttyACM0 --robot.id=my_followerclear
lerobot-calibrate --teleop.type=so101_leader  --teleop.port=/dev/ttyACM1   --teleop.id=my_leader
```

**Menjalankan Policy yang Telah Dilatih (Inference)**

```bash
lerobot-rollout \
  --strategy.type=base \
  --robot.type="so101_follower" \
  --robot.id="my_followerclear" \
  --robot.port="/dev/ttyACM0" \
  --robot.cameras="{ wrist: {type: opencv, index_or_path: /dev/v4l/by-id/usb-Sonix_Technology_Co.__Ltd._USB2.0_CAM1_USB2.0_CAM1-video-index0, width: 640, height: 480, fps: 30}, front: {type: opencv, index_or_path: /dev/v4l/by-id/usb-Etron_Technology__Inc._USB2.0_Camera-video-index0, width: 640, height: 480, fps: 30}}" \
  --policy.path=username/repo \
  --task="namatask" \
  --fps=30 \
  --display_data=true
```

---

## Perintah Tambahan (Merekam Dataset Sendiri)

Jika Anda ingin merekam dataset sendiri, jalankan perintah berikut:

```bash
lerobot-record \
  --robot.type=so101_follower --robot.port=/dev/ttyACMX --robot.id=my_followerclear \
  --teleop.type=so101_leader  --teleop.port=/dev/ttyACMX  --teleop.id=my_leader \
  --robot.cameras="{ wrist: {type: opencv, index_or_path: /dev/v4l/by-id/usb-Sonix_Technology_Co.__Ltd._USB2.0_CAM1_USB2.0_CAM1-video-index0, width: 640, height: 480, fps: 30}, front: {type: opencv, index_or_path: /dev/v4l/by-id/usb-Etron_Technology__Inc._USB2.0_Camera-video-index0, width: 640, height: 480, fps: 30}}" \
  --dataset.repo_id="username/repository" \
  --dataset.single_task="move red cube to the brown area" \
  --dataset.num_episodes=16 \
  --dataset.episode_time_s=20 \
  --dataset.reset_time_s=2 \
  --display_data=true \
  --resume=true
```
