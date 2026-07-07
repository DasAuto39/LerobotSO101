# Laporan Hasil BB-ACT pada Robot SO101

Pada proyek *imitation learning* ini, digunakan model VLA (Vision-Language-Action) dengan algoritma **BB-ACT** (Behavioral Cloning with Action Chunking and Transformers) pada robot SO101 untuk melakukan tugas memindahkan balok merah ke tempat persegi panjang berwarna coklat.

---

## Bagian BB-ACT

Tulung

### 1. Pengertian
<!-- Isi dengan penjelasan mengenai algoritma BB-ACT di sini -->


### 2. Dataset yang Dipakai
<!-- Isi dengan deskripsi dataset yang digunakan di sini -->


### 3. Training
<!-- Isi dengan proses/parameter training di sini -->


### 4. Hasil Training
<!-- Isi dengan hasil evaluasi/metrik dari training di sini -->


### 5. Hasil Inference
<!-- Isi dengan penjelasan analitis hasil inference di sini -->


---

## Observasi Hasil Inference

Berikut adalah hasil observasi *inference* pada robot SO101: 
Robot berhasil mengambil balok merah dan memindahkannya ke area persegi panjang berwarna coklat, sesuai dengan gerakan yang dilatih sebelumnya. Namun, terdapat gerakan patah patah saat robot bergerak yang kemungkinan besar perlu mentuning PID gerakan robotnya, dan robot perlu dibantu sedikit saat mengambil balok dikarenakan *servo gripper* tidak dapat menutup secara maksimal akibat adanya kerusakan kecil pada servo tersebut.

### Video Hasil

Berikut adalah dokumentasi video hasil pengujian (*inference*):

**1. Hasil Training Nur**  
<video src="./Video/HasilNur.mp4" controls width="100%"></video>

**2. Hasil Training Wildan**  
<video src="./Video/HasilWildan.mp4" controls width="100%"></video>

Atau video dapat diakses melalui tautan Google Drive berikut:  
 **[Video Hasil di Google Drive](https://drive.google.com/drive/folders/1kwrHN5rREQnvqet9mAZuFPpuiiKF5_q-?usp=sharing)**
---

## Langkah-Langkah Menjalankan Proyek

Berikut adalah panduan instalasi dan penggunaan untuk menjalankan *policy* pada robot.

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

Sebelum menjalankan perintah ke LeRobot, diperlukan untuk *login* akun Hugging Face pada terminal, dan menginstal *Git LFS* (Large File Storage) untuk menarik model atau dataset.

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
