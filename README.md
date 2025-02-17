# Enhancing-Medical-Image-Segmentation-through-Auto-Generated-Weak-Labels

# Auto-Generate-WLs
Code repository supporting the paper "Auto-Generating Weak Labels for Real & Synthetic Data to Improve Label-Scarce Medical Image Segmentation". For reproducibility, we have included all datasets + generated weak labels used in the paper.

## Using the code:
### 1. Prepare codebase and data
- Clone this repository:

```
git clone https://github.com/stanfordmlgroup/Auto-Generate-WLs
cd Auto-Generate-WLs
```

- Create a conda environment:
```
conda env create -f environment.yml
conda activate auto_wl
```
- Place the gold-standard (GS) dataset and unlabeled dataset under `pytorch-nested-unet/inputs` in the following format:
```
inputs
└── <dataset name>
    ├── images
    |   ├── 001.png
    │   ├── 002.png
    │   ├── 003.png
    │   ├── ...
    └── masks
        ├── 0
        |   ├── 001.png
        |   ├── 002.png
        |   ├── 003.png
        |   ├── ...
        └── 
```
We also provide the gold-standard dataset and weak labels used in this paper for reproducibility.

### 2. Train a model on the gold-standard dataset

Next, train an initial model on the gold-standard dataset. This will be used to generate prompts for MedSAM and generate the weak labels:

    python train.py --dataset <gs dataset> --arch NestedUNet 
	    --img_ext .png --mask_ext .png --batch_size 4 
		--input_w 256 --input_h 256

For example:
```
python train.py --dataset busi-25-small --arch NestedUNet 
	    --img_ext .png --mask_ext .png --batch_size 4 
		--input_w 256 --input_h 256
```
Then, generate predictions on the unlabeled dataset using this model:

    python eval.py --name <gs dataset>_NestedUNet_256_woDS 
		--dataset <unlabeled dataset>

For example:

    python eval.py --name busi-25-small_NestedUNet_256_woDS 
    		--dataset busi-25-aug-25

This will output the predictions to a `outputs/<model name>/<dataset name>/0` (the prediction path).
### 3. Generate predictions on the unlabeled dataset

Run the notebook `generate-masks.ipynb`, which will generate the weak labels and provide a visualization of the coarse labels, prompts, and weak labels. 

### 4. Train a model on the augmented dataset

Now, we are ready to train a model on the augmented dataset, as follows:

    python train.py --dataset <augmented dataset> --arch NestedUNet 
	    --img_ext .png --mask_ext .png --batch_size 4 
		--input_w 256 --input_h 256

For example:
```
python train.py --dataset busi-box-aug-50 --arch NestedUNet 
	    --img_ext .png --mask_ext .png --batch_size 4 
		--input_w 256 --input_h 256
```

Then, we can evaluate both our base model trained on the gold-standard dataset as well as the model trained on the augmented dataset, as follows:

    python eval.py --name <gs dataset>_NestedUNet_256_woDS 
		--dataset <test dataset>
	python eval.py --name <augmented dataset>_NestedUNet_256_woDS
		-- dataset <test dataset>
 and compare the resulting DICE and IOU. You can also compare the predictions in the respective output folders of the models.
 
 ### Acknowledgements
 This codebase uses code segments from [UNet++](https://github.com/4uiiurz1/pytorch-nested-unet) and [MedSAM](https://github.com/bowang-lab/MedSAM).
