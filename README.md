# BB-ACT Results Report on SO101 Robot

In this _imitation learning_ project for Robot Perception course, the VLA (Vision-Language-Action) model with the **BB-ACT** (Behavioral Cloning with Action Chunking and Transformers) algorithm is used on the SO101 robot to perform the task of moving a red block to a brown rectangular area.

With consist of group members:

1. Nur Annisa Hidayatul Masruroh (5024231025)
2. Ahmad Faiq Fawwaz (5024231031)
3. Akhmad Rizqullah Ridlohi (5024231037)
4. Moh. Wildan Risqi Maulidi (5024231058)

---

## 1. BB-ACT Overview

#### A. Behavior Cloning

**Behavior cloning** is a learning method that determines an agent's actions by imitating expert demonstrations. This approach can be solved through supervised _machine learning_ methods, such as classification.

Behavior cloning methods have proven effective for complex tasks, such as locomotion in bipedal robots. However, this method requires a large dataset and performs poorly when applied to environments outside the variations of the trained examples.

_Dataset Aggregation_ (**DAgger**) attempts to address some of these issues by appending expert demonstrations to the learned policy during the training process, so the learned policy and the expert policy complement each other. Nevertheless, DAgger is quite difficult to implement as it requires expert examples to be collected throughout the training process.

#### B. Action Chunking with Transformers

**Action Chunking with Transformers (ACT)** is a _behavior cloning_ algorithm that uses a **Conditional Variational Autoencoder (CVAE)** to model various conditions, combined with a _transformer_ to predict sequences of actions (_action chunks_) based on multimodal inputs.

By representing actions as target states that the manipulator must achieve, temporal aggregation methods can be used to combine action predictions from several previous time steps through weighted averaging. Combining these multiple predictions helps reduce compounding errors and unstable responses to states outside the training data distribution.

Although the model can still select suboptimal actions, the actions predicted in previous steps will provide a balancing effect on the final action. Furthermore, the _transformer_ network structure supports various multimodal inputs, such as _text prompts_, thereby improving the robustness of the ACT framework.

---

## 2. Dataset

The dataset used for the training process can be accessed through the following link:

**Dataset Link:**
https://huggingface.co/datasets/Kamna0321/so101_persepsi_robot_20260708_114140

This dataset was collected using the **LeRobot Dataset v3.0** framework and contains robot manipulation demonstrations for a **single-task imitation learning** problem. Each demonstration records synchronized visual observations, robot joint states, and corresponding robot actions. The dataset is intended for training **Vision-Language-Action (VLA)** models using behavior cloning.

---

### Dataset Information

| Parameter                | Value               |
| ------------------------ | ------------------- |
| Robot                    | SO-101 Follower     |
| Dataset Version          | v3.0                |
| Total Tasks              | 1                   |
| Total Episodes           | 60                  |
| Total Frames             | 35,661              |
| FPS                      | 30                  |
| Total Recording Duration | 19 minutes          |
| Split                    | Train (60 Episodes) |

---

### Data Collection

The demonstrations were collected by teleoperating the SO-101 robot arm to perform a single manipulation task. To improve the generalization capability of the VLA model, the workspace was divided into multiple regions, and the cube was placed at different locations for each episode.

In addition, the cube orientation was intentionally varied between demonstrations, including:

- **Aligned orientation**
- **Sideways orientation**

This variation allows the dataset to capture different visual appearances and robot trajectories while performing the same manipulation objective.

---

### Data Structure

The dataset consists of synchronized multimodal observations recorded at **30 FPS**.

#### 1. Action

Represents the target robot action predicted by the VLA model.

| Joint             |
| ----------------- |
| shoulder_pan.pos  |
| shoulder_lift.pos |
| elbow_flex.pos    |
| wrist_flex.pos    |
| wrist_roll.pos    |
| gripper.pos       |

- **Data Type:** `float32`
- **Shape:** `(6,)`

---

#### 2. Observation State

Represents the actual robot joint configuration at each frame.

| Joint             |
| ----------------- |
| shoulder_pan.pos  |
| shoulder_lift.pos |
| elbow_flex.pos    |
| wrist_flex.pos    |
| wrist_roll.pos    |
| gripper.pos       |

- **Data Type:** `float32`
- **Shape:** `(6,)`

---

#### 3. Observation Images

The dataset includes synchronized RGB images captured from two viewpoints.

| Camera       | Resolution |
| ------------ | ---------- |
| Front Camera | 640 × 480  |
| Wrist Camera | 640 × 480  |

Video specifications:

- Codec: **AV1**
- Pixel Format: **yuv420p**
- FPS: **30**
- Channels: **RGB (3 Channels)**

The front camera provides a global view of the workspace, while the wrist camera captures close-up observations from the robot end-effector.

---

### Episode Statistics

The dataset contains **60 demonstrations** with highly consistent durations.

| Statistic          | Value   |
| ------------------ | ------- |
| Shortest Episode   | 19.80 s |
| Longest Episode    | 19.83 s |
| Average Duration   | 19.81 s |
| Median Duration    | 19.80 s |
| Standard Deviation | 0.01 s  |

With a recording rate of **30 FPS**, each episode contains approximately **594 frames**, resulting in **35,661 frames** across the entire dataset.

The low standard deviation indicates a highly consistent demonstration length, which is beneficial for sequence-based imitation learning methods such as ACT and BB-ACT.

---

### Additional Metadata

Besides images and robot states, each frame includes additional metadata for synchronization and indexing.

| Metadata      | Description                      |
| ------------- | -------------------------------- |
| timestamp     | Recording timestamp of the frame |
| frame_index   | Frame number within an episode   |
| episode_index | Episode identifier               |
| task_index    | Task identifier                  |
| index         | Global frame index               |

---

### Dataset Storage Structure

The dataset follows the **LeRobot Dataset v3.0** storage format.

Robot state and action data are stored as **Apache Parquet** files:

```text
data/
└── chunk-xxx/
    └── file-xxx.parquet
```

Video observations are stored separately for each camera:

```text
videos/
├── observation.images.front/
│   └── chunk-xxx/
│       └── file-xxx.mp4
│
└── observation.images.wrist/
    └── chunk-xxx/
        └── file-xxx.mp4
```

Each chunk contains synchronized robot states, actions, and corresponding video files.

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

| Feature                  | Type   | Shape         |
| ------------------------ | ------ | ------------- |
| observation.state        | STATE  | (6,)          |
| observation.images.wrist | VISUAL | (3, 480, 640) |
| observation.images.front | VISUAL | (3, 480, 640) |

**Output**

| Feature | Type   | Shape |
| ------- | ------ | ----- |
| action  | ACTION | (6,)  |

---

#### Training Configuration

| Configuration   | Value |
| --------------- | ----- |
| Training steps  | 20000 |
| Batch size      | 16    |
| Optimizer       | adamw |
| Learning rate   | 1e-05 |
| Seed            | 1000  |
| LeRobot version | 0.6.1 |

---

Here is the model training results:

- https://huggingface.co/FawzQi/so101-BBact-policy

Here is the model training metric result:

- https://wandb.ai/ahamdfaiqfawwaz-institut-teknologi-sepuluh-nopember/lerobot-so101?nw=nwuserahamdfaiqfawwaz

---

## 4. Inference Observation Results

Here are the inference observation results on the SO101 robot:
The robot successfully picks up the red block and moves it to the brown rectangular area even in different positions of the block.

### Video Results

Here is the video documentation of the inference tests:

**[Video Results on Google Drive](https://drive.google.com/drive/folders/1kwrHN5rREQnvqet9mAZuFPpuiiKF5_q-?usp=sharing)**

---

## Steps to Run the Project

Here is the installation and usage guide to run the _policy_ on the robot.

### 1. Prerequisites Installation

**Install Miniforge**

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

**Setup Conda Environment**

```bash
conda create -y -n lerobot python=3.12
conda activate lerobot
```

**Install FFmpeg**

```bash
conda install ffmpeg -c conda-forge
```

### 2. LeRobot Installation

**Clone Repository and Install**

```bash
git clone https://github.com/huggingface/lerobot.git
cd lerobot
pip install lerobot
pip install 'lerobot[feetech]' # Required for SO-101 servo motors
```

After installing LeRobot on the operating system, the next step is to run commands in the terminal to use the SO101 robot. For a more complete explanation, please refer to the `AGENT_GUIDE.md` file located in the downloaded LeRobot repository.

### 3. Preparation Before Running the Robot

Before sending commands to LeRobot, it is necessary to _login_ to a Hugging Face account in the terminal and install _Git LFS_ (Large File Storage) to pull models or datasets.

```bash
huggingface-cli login
# Or using the command: hf auth login

git lfs install && git lfs pull
```

### 4. Execution on Robot

**Finding the Robot Port**

```bash
lerobot-find-port
```

> **Note:** macOS typically uses `/dev/tty.usbmodem...`; Linux uses `/dev/ttyACM0` (may require access using `sudo chmod 666 /dev/ttyACM0`).

**Robot Calibration**

```bash
lerobot-calibrate --robot.type=so101_follower --robot.port=/dev/ttyACM0 --robot.id=my_followerclear
lerobot-calibrate --teleop.type=so101_leader  --teleop.port=/dev/ttyACM1   --teleop.id=my_leader
```

**Running the Trained Policy (Inference)**

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

## Additional Commands (Recording Custom Datasets)

If you want to record your own dataset, run the following command:

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
