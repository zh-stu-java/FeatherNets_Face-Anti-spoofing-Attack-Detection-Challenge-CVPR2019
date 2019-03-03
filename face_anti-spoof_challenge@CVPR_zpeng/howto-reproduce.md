# How to reproduce on Linux

## Prerequisites

### Install conda
Follow https://conda.io/projects/conda/en/latest/user-guide/install/index.html

###  Create conda environment and install requeirements
```
conda env create -n env_name -f env.yml
```

###  Activate conda environment
```
conda activate env_name
```

### Data preparation
We use 2 dataset for training our models. One is CASIA-SURF, another is created by ourselves, named Multi-Modality Face Dataset', abbreviated as MMFD.

Download CASIA-SURF

Download MMFD(TODO: add url here).

Uncompressed and copy them to the data directory and run these commands (assume you are already in the source directory):
```
data
├── fileList.ipynb
├── fileList.py
├── our_filelist
├── our_realsense  # MMFD dataset
├── Testing    #Test set
├── test_private_list.txt  # test set list with label
├── Training    #Train set
├── train_list.txt
├── Val        #Val set
├── val_label.txt #val set with label
└── val_public_list.txt
```


```
cd data
python fileList.py
```

If everything works well, you can see the contents of data directory like these:
```
data
├── 2depth_train.txt
├── 2ir_train.txt
├── 2label_train.txt
├── 2rgb_train.txt
├── depth_test.txt
├── depth_val.txt
├── fileList.ipynb
├── fileList.py
├── ir_test.txt
├── ir_val.txt
├── label_test.txt
├── label_val.txt
├── our_filelist
├── our_realsense  # MMFD dataset
├── rgb_test.txt
├── rgb_val.txt
├── Testing    #Test set
├── test_private_list.txt  # test set list with label
├── Training    #Train set
├── train_list.txt
├── Val        #Val set
├── val_label.txt
└── val_public_list.txt
```

## Train

### Download pretrained models
Download [fishnet150](https://pan.baidu.com/s/1uOEFsBHIdqpDLrbfCZJGUg) pretrained model from [FishNet150](https://github.com/kevin-ssy/FishNet)(Model trained without tricks )

Download [mobilenetv2](https://drive.google.com/open?id=1jlto6HRVD3ipNkAl1lNhDbkBp7HylaqR) pretrained model from [MobileNet V2](https://github.com/tonylins/pytorch-mobilenet-v2)

**move them to checkpoints/pre-trainedModels/**

### 1. Train FishNet150
```
nohup python main.py --config="cfgs/fishnet150-32-5train.yaml" --b 32 --lr 0.01 --every-decay 30 --fl-gamma 2 >> fishnet150-train.log &
```

### 2. Train MobileNet V2
```
nohup python main.py --config="cfgs/mobilenetv2.yaml" --b 32 --lr 0.01 --every-decay 40 --fl-gamma 2 >> mobilenetv2-bs32-train.log &
```

### 3. Train FeatherNet54
```
nohup python main.py --config="cfgs/FeatherNet54-32.yaml" --every-decay 60 -b 32 --lr 0.01 --fl-gamma 3 >>FNet54-bs32-train.log &
```

### 4. Train FeatherNet54-SE
```
nohup python main.py --config="cfgs/FeatherNet54-se-64.yaml" --b 64 --lr 0.01  --every-decay 60 --fl-gamma 3 >> FNet54-se-bs64-train.log &
```

### 5. Train MobileLiteNetA
```
nohup python main.py --config="cfgs/MobileLiteNetA-32.yaml" --b 32 --lr 0.01  --every-decay 60 --fl-gamma 3 >> MobileLiteNetA-bs32-train.log &
```

### 6. Train MobileLiteNetB
```
nohup python main.py --config="cfgs/MobileLiteNetB-32.yaml" --b 32 --lr 0.01  --every-decay 60 --fl-gamma 3 >> MobileLiteNetB-bs32--train.log &
```

### How to create a submission file for val set
```
python main.py --config="cfgs/mobilenetv2.yaml" --resume ./checkpoints/mobilenetv2_bs32/_4_best.pth.tar --val True --val-save True
```
### How to create a submission file for test set
```
python main.py --config="cfgs/mobilenetv2.yaml" --resume ./checkpoints/mobilenetv2_bs32/_4_best.pth.tar --phase-test True --val True --val-save True
```

## Test
### Predict the test set with several models
By running the following commands, the performance results of test set are store in the submission/ directory.
choose best checkpoints to resume
we choose these checkpoints to ensemble.

|model name | ACER|TPR@FPR=10E-2|TPR@FPR=10E-3|FP|FN|epoch|params|FLOPs|
| ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
|FishNet150| 0.00144|0.999668|0.998330|19|0|27|24.96M|6452.72M|
|FishNet150| 0.00181|1.0|0.9996|24|0|52|24.96M|6452.72M|
|FishNet150| 0.00496|0.998664|0.990648|48|8|16|24.96M|6452.72M|
|MobileNet v2|0.00228|0.9996|0.9993|28|1|5|2.23M|306.17M
|MobileNet v2|0.00387|0.999433|0.997662|49|1|6|2.23M|306.17M
|MobileNet v2|0.00402|0.9996|0.992623|51|1|7|2.23M|306.17M
|FeatherNet54|0.00242|1.0|0.99846|32|0|41|0.57M|270.91M|
|FeatherNet54-se|0.00242|1.0|0.996994|32|0|69|0.57M|270.91M|
|MobileLiteNetA|0.00261|1.00|0.961590|19|7|51|0.35M|79.99M|
|MobileLiteNetB|0.00168|1.0|0.997662|20|1|48|0.35M|83.05M|
|**Ensembled all**|0.0000|1.0|1.0|0|0|-|-|-|


```
python main.py --config="cfgs/fishnet150-32.yaml" --resume ./checkpoints/fishnet150_bs32/_15_best.pth.tar --phase-test True --val True --val-save True
python main.py --config="cfgs/fishnet150-32.yaml" --resume ./checkpoints/fishnet150_bs32/_51_best.pth.tar --phase-test True --val True --val-save True
python main.py --config="cfgs/fishnet150-32.yaml" --resume ./checkpoints/fishnet150_bs32/_10_best.pth.tar --phase-test True --val True --val-save True
python main.py --config="cfgs/FeatherNet54-32.yaml" --resume ./checkpoints/FeatherNet54/_40_best.pth.tar --phase-test True --val True --val-save True
python main.py --config="cfgs/FeatherNet54-se-64.yaml" --resume ./checkpoints/FeatherNet54-se/_68_best.pth.tar --phase-test True --val True --val-save True
python main.py --config="cfgs/mobilenetv2.yaml" --resume ./checkpoints/mobilenetv2_bs32/_4_best.pth.tar --phase-test True --val True --val-save True
python main.py --config="cfgs/mobilenetv2.yaml" --resume ./checkpoints/mobilenetv2_bs32/_5.pth.tar --phase-test True --val True --val-save True
python main.py --config="cfgs/mobilenetv2.yaml" --resume ./checkpoints/mobilenetv2_bs32/_6.pth.tar --phase-test True --val True --val-save True
python main.py --config="cfgs/MobileLiteNetA-32.yaml" --resume ./checkpoints/mobilelitenetA_bs32/_50_best.pth.tar --phase-test True --val True --val-save True
python main.py --config="cfgs/MobileLiteNetB-32.yaml" --resume ./checkpoints/mobilelitenetB_bs32/_47_best.pth.tar --phase-test True --val True --val-save True
```

### Generate the final submission by assemble results from above models
repalce file_dir1,file_dir2,.... to your generate submission files in gen_final_submission.py.
like 
```
file_dir1='submission/2019-01-28_15:45:05_fishnet150_52_submission.txt'
```
then run these commands

```

python gen_final_submission.py
```
finally you will see the final submission file(final_submission.txt) in the submission/ directory.