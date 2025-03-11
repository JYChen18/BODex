# BODex

A GPU-based efficient pipeline for dexterous grasp synthesis built on [cuRobo](https://github.com/NVlabs/curobo/tree/main), proposed in *BODex: Scalable and Efficient Robotic Dexterous Grasp Synthesis Using Bilevel Optimization [ICRA 2025]*.

[Project page](https://pku-epic.github.io/BODex/) ｜ [Paper](https://arxiv.org/abs/2412.16490) ｜ [Dataset](https://huggingface.co/datasets/JiayiChenPKU/BODex) ｜ [Benchmark code](https://github.com/JYChen18/DexGraspBench) ｜ [Learning code (TODO)](:TODO)

## Introduction
### Main Features
- **Grasp Synthesis**: Generate force-closure grasps for floating dexterous hands, such as the Shadow, Allegro, and Leap Hand.
- **Trajectory Planning**: Plan collision-free approaching trajectories with the table for hands mounted on robotic arms, e.g., UR10e + Shadow Hand systems.

### Highlights
- **Efficient**: Capable of synthesizing millions of grasps per day using a single NVIDIA 3090 GPU.
- **Generalizable**: Supports different hands and a wide range of objects.

## Getting Started
1. **Install git lfs**: Before `git clone` this repository, please make sure that the git lfs has been installed by `sudo apt install git-lfs`.

2. **Install the Python environment**:
```
conda create -n bodex python=3.10
conda activate bodex

conda install pytorch==2.2.2 pytorch-cuda=12.1 -c pytorch -c nvidia #or use pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118 for cuda 11.8

pip install -e . --no-build-isolation  

pip install usd-core 
conda install pytorch-scatter -c pyg
conda install coal -c conda-forge   # https://github.com/coal-library/coal

pip uninstall numpy
pip install numpy==1.26.4

cd src/curobo/geom/cpp
python setup.py install    # install coal_openmp_wrapper
conda install -c conda-forge hpp-fcl
```

3. **Prepare object assets**: Download our pre-processed object assets `DGN_obj_processed.zip` and `DGN_obj_split.zip` from [here](https://huggingface.co/datasets/JiayiChenPKU/BODex) and organize the unzipped folders as below. Alternatively, new object assets can be pre-processed using [MeshProcess](https://github.com/JYChen18/MeshProcess).
```
src/curobo/content/assets/object/DGN_obj
|- processed_data
|  |- core_bottle_1a7ba1f4c892e2da30711cdbdbc73924
|  |_ ...
|- valid_split
|  |- all.json
|  |_ ...
```
   
4. **Synthesize grasp poses**: Each synthesized grasping data point includes a pre-grasp, a grasp, and a squeeze pose. 
```
# Single GPU version
CUDA_VISIBLE_DEVICES=7 python example_grasp/plan_batch_env.py -c sim_shadow/fc.yml -w 40 

# Debugging. The saved USD file has the whole optimization process, while the intermediate gradient on each contact point is denoted by the purple line.
CUDA_VISIBLE_DEVICES=7 python example_grasp/plan_batch_env.py -c sim_shadow/fc.yml -w 1 -m usd -debug -d all -i 0 1

# Multiple GPU version
python example_grasp/multi_gpu.py -c sim_shadow/fc.yml -t grasp -g 0 1 2 3 
```
We can also **synthesize approaching trajectories** that are collision-free with the table for hands mounted on arms, e.g., UR10e+Shadow Hand systems.
 ```
# Single GPU version
CUDA_VISIBLE_DEVICES=7 python example_grasp/plan_mogen_batch.py -c sim_shadow/tabletop.yml -t grasp_and_mogen

# Multiple GPU version
python example_grasp/multi_gpu.py -c sim_shadow/tabletop.yml -t grasp_and_mogen -g 0 1 2 3 
```
On a single GPU, the grasp synthesis supports parallizing different objects, but the motion planning only supports parallizing different trajectories for the same object.

5. **(Optional) Visualize synthesized poses**:
```
python example_grasp/visualize_npy.py -c sim_shadow/fc.yml -p debug -m grasp
```

6. **Evaluate grasp poses and filter out bad ones**: please see [DexGraspBench](https://github.com/JYChen18/DexGraspBench).

## Project using BODex
Some projects *make modifications* on our pipeline to quickly synthesize large-scale datasets of grasping poses, such as 
- [DexGraspNet2.0](https://pku-epic.github.io/DexGraspNet2.0/)
<!-- - [GraspVLA (coming soon)]() -->

## Citation

If you found this repository useful, please consider to cite the following works,

```
@article{chen2024bodex,
  title={BODex: Scalable and Efficient Robotic Dexterous Grasp Synthesis Using Bilevel Optimization},
  author={Chen, Jiayi and Ke, Yubin and Wang, He},
  journal={arXiv preprint arXiv:2412.16490},
  year={2024}
}

@misc{curobo_report23,
      title={cuRobo: Parallelized Collision-Free Minimum-Jerk Robot Motion Generation},
      author={Balakumar Sundaralingam and Siva Kumar Sastry Hari and Adam Fishman and Caelan Garrett
              and Karl Van Wyk and Valts Blukis and Alexander Millane and Helen Oleynikova and Ankur Handa
              and Fabio Ramos and Nathan Ratliff and Dieter Fox},
      year={2023},
      eprint={2310.17274},
      archivePrefix={arXiv},
      primaryClass={cs.RO}
}
```
