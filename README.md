# SpatialScope
A unified approach for integrating spatial and single-cell transcriptomics data by leveraging deep generative models


## Installation
``` shell
$ git clone https://github.com/YangLabHKUST/SpatialScope-Beta.git
$ cd SpatialScope-Beta
$ conda env create -f environment.yml
$ conda activate SpatialScope
```
check the installation status
```shell
$ python ./src/Cell_Type_Identification.py -h
```

## Quick start for Visium data

We illustrate the usage of SpatialScope using a single slice of 10x Visium human heart data:
- Spatial data: ./demo_data/V1_Human_Heart_spatial.h5ad
- Image data: ./demo_data/V1_Human_Heart_image.tif
- scRNA reference data: ./Ckpts_scRefs/Heart_D2/Ref_Heart_sanger_D2.h5ad
- Pretrained model checkpoint: ./Ckpts_scRefs/Heart_D2/model_600001.pt


### Step1: Nuclei segmentation

```
python ./src/Nuclei_Segmentation.py --tissue heart --out_dir  ./output  --ST_Data ./demo_data/V1_Human_Heart_spatial.h5ad --Img_Data  ./demo_data/V1_Human_Heart_image.tif
```

Input:

- --out_dir: output directory
- --tissue: output sub-directory
- --ST_Data: ST data file path
- --Img_Data: H&E stained image data file path (require raw H&E image with high resolution, about 10000x10000 resolution)

This step will make `./output/heart` directory, and generate two files:

- Visualization of nuclei segmentation results: nuclei_segmentation.png
- Preprocessed ST data for cell type identification: sp_adata_ns.h5ad (**cell_locations** that contains spatial locations of segmented cells will be added to .uns)

### Step2: Cell type identification

```
python ./src/Cell_Type_Identification.py --tissue heart --out_dir  ./output  --ST_Data ./output/heart/sp_adata_ns.h5ad --SC_Data ./Ckpts_scRefs/Heart_D2/Ref_Heart_sanger_D2.h5ad --cell_class_column cell_type
```

Input:

- --out_dir: output directory
- --tissue: output sub-directory
- --ST_Data: ST data file path (generated in Step 1)
- --SC_Data: single-cell reference data file path (When using your own scRef file, we recommend adding a **Marker** column to the .var to pre-select several thousand marker or highly variable genes as in "./Ckpts_scRefs/Heart_D2/Ref_Heart_sanger_D2.h5ad")
- --cell_class_column: cell class label column in scRef file
  
This step will generate three files:

- Visualization of cell type identification results: estemated_ct_label.png
- Cell type identification results: CellTypeLabel_nu10.csv
- Preprocessed ST data for gene expression decomposition: sp_adata.h5ad


### Step3: Gene expression decomposition

```
python ./src/Decomposition.py --tissue heart --out_dir  ./output --SC_Data ./Ckpts_scRefs/Heart_D2/Ref_Heart_sanger_D2.h5ad --cell_class_column cell_type  --ckpt_path ./Ckpts_scRefs/Heart_D2/model_600001.pt --spot_range 0,100 --gpu 0,1,2,3
```

Input:

- --out_dir: output directory
- --tissue: output sub-directory
- --SC_Data: single-cell reference data file path 
- --cell_class_column: cell class label column in scRef file
- --ckpt_path: model checkpoint file path (As the model checkpoint was trained on scRef file, the checkpoint and scRef file much be matched)
- --spot_range: limited by GPU memory, we can only handle at most about 1000 spots in 4 GPUs at a time. e.g., 0,1000 means 0 to 1000-th spot
- --gpu: Visible GPUs

This step will generate one file:

- Single-cell resolution ST data generated by SpatialScope for spot 0-100: generated_cells_spot0-100.h5ad


## Contact information

Please contact Xiaomeng Wan (xwanaf@connect.ust.hk), Jiashun Xiao (jxiaoae@connect.ust.hk) or Prof. Can Yang (macyang@ust.hk) if any enquiry.
