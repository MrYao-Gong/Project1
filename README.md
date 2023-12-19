# DDPM Based on The Dual Diffusion Implicit Bridges

This project experiment ultimately achieved the results in the paper: Dual Diffusion Implicit Bridges (ICLR 2023)

Code Running Comments:

This code need imagenet file to run.

! python download.py --exp imagenet

The exact file and folder locations are shown in the image：

![image](https://github.com/MrYao-Gong/Project1/assets/154312757/c345a681-73c1-49ad-b255-1aa3edbc9481)



### Training Synthetic Models

`python scripts/synthetic_train.py --num_res_blocks 3 --diffusion_steps 4000 --noise_schedule linear --lr 1e-4 --batch_size 20000 --task 0`

Task is an integer in {0, 1, 2, 3, 4, 5} corresponding to one of the synthetic types.

Training each model probably takes only a few (3-4) hours on a GPU.

The models and logs are saved to the directory at `OPENAI_LOGDIR`. If you want to save the model files to your desired
folder, modify the variable via `export OPENAI_LOGDIR=...`

### Cycle Consistency

`python scripts/synthetic_cycle.py --num_res_blocks 3 --diffusion_steps 4000 --batch_size 30000 --source 0 --target 1`

The above command runs the cycle-consistent translation experiment in the paper: between datasets Moons (0) and
Checkerboards (1). The generated experiment plots are saved under the new directory `experiments/images`.

### Synthetic Translation

`python scripts/synthetic_translation.py --num_res_blocks 3 --diffusion_steps 4000 --batch_size 30000 --source 0 --target 3`

The above command performs translation between the two synthetic domains and saves the resulting plots
to `experiments/images`.

### Sample from Synthetic Models

`python scripts/synthetic_sample.py --num_res_blocks 3 --diffusion_steps 4000 --batch_size 20000 --num_samples 80000 --task 1`

### Miscellaneous

- If you encounter `ModuleNotFoundError: No module named 'guided_diffusion'`, here is one possible
  solution: https://stackoverflow.com/a/23210066/3284573.

## ImageNet Translation

### Installation

**Download the model weights**. Similarly, run `python download.py --exp imagenet` to download the pretrained,
class-conditional ImageNet models from [guided-diffusion](https://github.com/openai/guided-diffusion). The script will
create a directory `models/imagenet` and put the classifier and diffusion model weights there.

**Copy the validation dataset**. We use the ImageNet validation set for domain translation. Two steps:

- Download the ImageNet validation set from ILSVRC2012. Unzip it. This will create a folder containing image files named
  like "ILSVRC2012_val_000XXXXX.JPG".
- Remember the path to the folder containing the validation set as `val_dir`, as we'll need it for the next command.

We are now ready to translate the images.

### Translation between ImageNet Classes

```commandline
MODEL_FLAGS="--attention_resolutions 32,16,8 --class_cond True --diffusion_steps 1000 --image_size 256 --learn_sigma True --noise_schedule linear --num_channels 256 --num_head_channels 64 --num_res_blocks 2 --resblock_updown True --use_fp16 True --use_scale_shift_norm True"
python scripts/imagenet_translation.py $MODEL_FLAGS --classifier_scale 1.0 --source 260,261,282,283 --target 261,262,283,284 --val_dir path_to_val_set
```

The above command copies the validation images for the source classes to `./experiments/imagenet`. Translated images are
placed in the same folder, with the target class appended in the filename.

We can update `source` and `target` to translate between other ImageNet classes. The corresponding val images are copied
automatically.

We translate the domain pairs in the specified order. For example, in the above command, we translate from class 260 to
261, 283 to 284, etc.

We can experiment with `classifier_scale` to guide the denoising process towards the target class with different
strengths.

We can prepend the Python command with `mpiexec -n N` to run it over multiple GPUs. For details, refer
to [guided-diffusion](https://github.com/openai/guided-diffusion).

