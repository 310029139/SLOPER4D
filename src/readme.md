# Data processing pipeline


## **Data structure**
```bash
# the data structure for every sequence
├── root_folder
   ├── lidar_data/
   |  ├── lidar_frames_rot/        
   |  |   └── '*.pcd'              # undistorted n frames point clouds in global coordinates
   |  ├── 'lidar_trajectory.txt'   # everyline: framenum X Y Z qx qy qz qw timestamp
   |  └── 'tracking_traj.txt'      # everyline: X Y Z framenum timestamp
   ├── mocap_data/
   |  └── '*_second.bvh'           # mocap data
   ├── rgb_data/
   |  └── '*.mp4'
   ├── '*_labels.pkl'              # all 2D/3D labels and origin human data
   └── 'dataset_params.json'       # meta info
```


## **Environment**
- Python 3.8.12
- PyTorch 1.7.0
- CUDA 11.0
- Ubuntu 18.04

## **Installation**
1. Clone the repository:
```bash
git clone https://github.com/climbingdaily/SLOPER4D.git
```
2. Install the required packages:
```bash
conda create --name sloper4d python==3.8 -y
conda activate sloper4d
conda install pytorch==1.7.0 torchvision==0.8.0 torchaudio==0.7.0 cudatoolkit=11.0 -c pytorch
pip install -r requirements.txt
```
## **Dependencies**
- **SMPL**: Download v1.0.0 version SMPL models `SMPL_NEUTRAL.pkl`, `SMPL_FEMALE.pkl`, `SMPL_MALE.pkl` and `J_regressor_extra.npy` from http://smpl.is.tue.mpg.de and put them in `smpl` directory.
- [**CSF**](https://github.com/jianboqi/CSF) (Optional)
- **FFmpeg** (version >= 3.4.11)

## **Data loader**
coming soon...


## **Processing**
```bash
root_folder=/path/to/root_folder
```

- ### **Mocap data** 
1. Convert the `bvh` file to `csv` files
   ```bash
   # pip install bvh-converter 
   bvhconverter -r "/path/to/bvh"
   ```

2. Jumping peak detection. Used to double check the synchronization time in `dataset_params.json`
   ```bash
   python tools/detect_jumps.py -M "/path/to/bvh" 
   ```

- ### **LiDAR data** 

1. Scene Mesh reconstruction (With TSDF fusion) and human data generation
   ``` bash
   python src/process_raw_data.py --root_folder $root_folder --tsdf --sync 
   ```
   optional arguments:
   ```
   --root_folder          The data's root directory
   --traj_file  
   --params_file  
   -S, --start_idx        The start frame index in LiDAR for processing, 
                           specified when sychronization time is too late
   -E, --end_idx          The end frame index in LiDAR for processing, 
                           specified when sychronization time is too early.
   -VS, --voxel_size      The voxel filter parameter for TSDF fusion
   --skip_frame           The everay n frame used for mapping
   --tsdf                 Use VDB fusion to build the scene mesh 
   --sdf_trunc            The trunction distance for SDF funtion
   --sync                 Synced all data and save them in a pkl based on the params_file
   ```

2. Human point clouds cropping
   ```bash
   python src/process_human_points.py -R $root_folder [--scene <scene path>]
      ```
- ### **RGB data ** 
   Under `processing`, convert videos to images by:

    ```shell
    python vdo2imgs.py --save_path "path to dataset"
    ```


## **Visualization**

- ### **RGB visualization**

    Under `processing`: 

1.  Download and place `SMPL_*.pkl` models in `processing/smpl_models`

2.  Install [detectron2](https://github.com/facebookresearch/detectron2.git), [pypenGL](https://github.com/mcfletch/pyopengl.git), [pypcd](https://github.com/klintan/pypcd.git)

3.  Change the project path and dataset path in `render_sence_base.sh`, then use `render_sence_all.sh` to generate videos under `rgb_datas`

    ```shell
    # in render_sence_base.sh
    PROJ_PATH="path to code"
    DATA_PATH="path to dataset"
    ...
    --draw_coco17 \				# visualize COCO17 skeleton
    --draw_coco17_kps \		    # visualize COCO17 keypoints
    --draw_smpl \				# visualize SMPL
    --draw_human_pc \			# visualize human point cloud
    --draw_scene_pc \			# visualize scene point cloud
    ```




- ### **SMPL visualization**
   Please refer this visualization tool [SMPL-Scene Viewer](https://github.com/climbingdaily/SMPL-Scene-Viewer),
   or [aitviewer](https://github.com/climbingdaily/aitviewer)


## License
The SLOPER4D dataset is published under the Creative [Commons Attribution-NonCommercial-ShareAlike 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) License. You must attribute the work in the manner specified by the authors, you may not use this work for commercial purposes and if you alter, transform, or build upon this work, you may distribute the resulting work only under the same license. Contact us if you are interested in commercial usage.

