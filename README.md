# SECOND for KITTI/NuScenes object detection (1.6.0 Alpha)
SECOND detector.

The original readme is [here](README_origin.md).

ONLY support python 3.6+, pytorch 1.0.0+. Tested in Ubuntu 16.04/18.04/Windows 10.

If you want to train nuscenes dataset, see [this](NUSCENES-GUIDE.md).

## Install

### 1. Clone code (branch for BUSS)

```bash
git clone -b devel_buss https://github.com/BUSS-DeeCamp/second.pytorch.git
cd ./second.pytorch/second
```

### 2. Install dependencies

* basic packages
```bash
pip install -r requirements.txt
```

* spconv
```bash
sudo apt-get install libboost-all-dev
git clone https://github.com/traveller59/spconv.git --recursive
cd spconv && git checkout a6ae896
python setup.py bdist_wheel
cd ./dist && pip install *
```

* apex

If you want to train with fp16 mixed precision (train faster in RTX series, Titan V/RTX and Tesla V100), you need to install [apex](https://github.com/NVIDIA/apex).

```bash
git clone https://github.com/NVIDIA/apex
cd apex
pip install -v --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" ./
```

* nuscenes-devkit

```bash
git clone https://github.com/poodarchu/nuscenes.git
cd nuscenes
python setup.py install
```

### 3. Add project folder to PYTHONPATH

Add the `second.pytorch` folder to PYTHONPATH in your IDE.

## Prepare dataset

* KITTI Dataset preparation

Download KITTI dataset and create some directories first:

```plain
└── KITTI_DATASET_ROOT
       ├── training    <-- 7481 train data
       |   ├── image_2 <-- for visualization
       |   ├── calib
       |   ├── label_2
       |   ├── velodyne
       |   └── velodyne_reduced <-- empty directory
       └── testing     <-- 7580 test data
           ├── image_2 <-- for visualization
           ├── calib
           ├── velodyne
           └── velodyne_reduced <-- empty directory
```

Then run
```bash
python create_data.py kitti_data_prep --root_path=KITTI_DATASET_ROOT
```

* [NuScenes](https://www.nuscenes.org) Dataset preparation

Download NuScenes dataset:
```plain
└── NUSCENES_TRAINVAL_DATASET_ROOT
       ├── samples       <-- key frames
       ├── sweeps        <-- frames without annotation
       ├── maps          <-- unused
       └── v1.0-trainval <-- metadata and annotations
└── NUSCENES_TEST_DATASET_ROOT
       ├── samples       <-- key frames
       ├── sweeps        <-- frames without annotation
       ├── maps          <-- unused
       └── v1.0-test     <-- metadata
```

Then run
```bash
python create_data.py nuscenes_data_prep --root_path=NUSCENES_TRAINVAL_DATASET_ROOT --version="v1.0-trainval" --max_sweeps=10
python create_data.py nuscenes_data_prep --root_path=NUSCENES_TEST_DATASET_ROOT --version="v1.0-test" --max_sweeps=10
--dataset_name="NuscenesDataset"
```

* Modify config file

There is some path need to be configured in config file:

```bash
train_input_reader: {
  ...
  database_sampler {
    database_info_path: "/path/to/dataset_dbinfos_train.pkl"
    ...
  }
  dataset: {
    dataset_class_name: "DATASET_NAME"
    kitti_info_path: "/path/to/dataset_infos_train.pkl"
    kitti_root_path: "DATASET_ROOT"
  }
}
...
eval_input_reader: {
  ...
  dataset: {
    dataset_class_name: "DATASET_NAME"
    kitti_info_path: "/path/to/dataset_infos_val.pkl"
    kitti_root_path: "DATASET_ROOT"
  }
}
```

## Usage

### A quick start

* [simple_inference.py](second/simple-inference.py)
    1. Change the [config](https://github.com/BUSS-DeeCamp/second.pytorch/blob/devel_buss/second/simple-inference.py#L18) and corresponding [model](https://github.com/BUSS-DeeCamp/second.pytorch/blob/devel_buss/second/simple-inference.py#L31) path in the script
    2. Select a [index](https://github.com/BUSS-DeeCamp/second.pytorch/blob/devel_buss/second/simple-inference.py#L60) in the dataset
    3. Run the script
    ```python
    python simple_inference.py
    ```

* [inference_from_bin_file.py](second/inference_from_bin_file.py)
    1. Change the [config](https://github.com/BUSS-DeeCamp/second.pytorch/blob/devel_buss/second/inference_from_bin_file.py#L200) and corresponding [model](https://github.com/BUSS-DeeCamp/second.pytorch/blob/devel_buss/second/inference_from_bin_file.py#L201) path in the script
    2. Run the script (several bin files for testing can be found [here](second/test_data))
    ```python
    python inference_from_bin_file.py /path/to/cloud.bin
    ```