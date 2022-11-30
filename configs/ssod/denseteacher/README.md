简体中文 | [English](README_en.md)

# DenseTeacher (Dense Teacher: Dense Pseudo-Labels for Semi-supervised Object Detection)

## 模型库

|      模型       |          基础检测器      |  监督数据比例   | Sup mAP<sup>val<br>0.5:0.95 | Semi mAP<sup>val<br>0.5:0.95 |  Semi Epochs (Iters)  |  模型下载  |   配置文件   |
| :------------: | :---------------------: | :-----------: | :-------------------------: |:---------------------------: |:--------------------: | :-------: |:---------: |
| DenseTeacher   |   [FCOS R50-FPN](../baseline/fcos_r50_fpn_2x_coco_sup005.yml)  | 5% | 21.3 | 30.6  | 240 (87120) | [download](https://paddledet.bj.bcebos.com/models/denseteacher_fcos_r50_fpn_coco_semi005.pdparams) | [config](./denseteacher_fcos_r50_fpn_coco_semi005.yml) |
| DenseTeacher   |   [FCOS R50-FPN](../baseline/fcos_r50_fpn_2x_coco_sup010.yml)  | 10%| 26.3 | 35.1  | 240 (174240)| [download](https://paddledet.bj.bcebos.com/models/denseteacher_fcos_r50_fpn_coco_semi010.pdparams) | [config](./denseteacher_fcos_r50_fpn_coco_semi010.yml) |
| DenseTeacher(LSJ)|   [FCOS R50-FPN](../baseline/fcos_r50_fpn_2x_coco_sup010.yml)| 10%| 26.3 | 37.1  | 240 (174240)| [download](https://paddledet.bj.bcebos.com/models/denseteacher_fcos_r50_fpn_coco_semi010_lsj.pdparams) | [config](./denseteacher_fcos_r50_fpn_coco_semi010_lsj.yml) |
| DenseTeacher   |   [FCOS R50-FPN](../../fcos/fcos_r50_fpn_iou_multiscale_2x_coco.ymll)  |full| 42.6 |   -   |  36 (263844)| [download](https://paddledet.bj.bcebos.com/models/denseteacher_fcos_r50_fpn_coco_full.pdparams) | [config](./denseteacher_fcos_r50_fpn_coco_full.yml) |


**注意:**
 - 以上模型训练默认使用8 GPUs，监督数据总batch_size默认为16，无监督数据总batch_size默认也为16，默认初始学习率为0.01。如果改动了总batch_size，请按线性比例相应地调整学习率；
 - `Semi Epochs (Iters)`表示**半监督训练**的模型的 Epochs (Iters)，如果使用**自定义数据集**，需自行根据Iters换算到对应的Epochs调整，最好保证总Iters 和COCO数据集的设置较为接近；
 - `Sup mAP`是**只使用有监督数据训练**的模型的精度，请参照**基础检测器的配置文件** 和 [baseline](../baseline)；
 - `Semi mAP`是**半监督训练**的模型的精度，模型下载和配置文件的链接均为**半监督模型**；
 - `LSJ`表示 large-scale jittering，表示更大范围的多尺度训练，可进一步提升精度，但训练速度也会变慢；
 - 半监督检测的配置和使用，请参照[文档](../README.md/#半监督检测配置)；


## 使用说明

仅训练时必须使用半监督检测的配置文件去训练，评估、预测、部署也可以按基础检测器的配置文件去执行。

### 训练

```bash
# 单卡训练 (不推荐，需按线性比例相应地调整学习率)
CUDA_VISIBLE_DEVICES=0 python tools/train.py -c ssod/denseteacher/denseteacher_fcos_r50_fpn_coco_semi010.yml --eval

# 多卡训练
python -m paddle.distributed.launch --log_dir=denseteacher_fcos_semi010/ --gpus 0,1,2,3,4,5,6,7 tools/train.py -c ssod/denseteacher/denseteacher_fcos_r50_fpn_coco_semi010.yml --eval
```

### 评估

```bash
CUDA_VISIBLE_DEVICES=0 python tools/eval.py -c ssod/denseteacher/denseteacher_fcos_r50_fpn_coco_semi010.yml -o weights=output/denseteacher_fcos_r50_fpn_coco_semi010/model_final.pdparams
```

### 预测

```bash
CUDA_VISIBLE_DEVICES=0 python tools/infer.py -c ssod/denseteacher/denseteacher_fcos_r50_fpn_coco_semi010.yml -o weights=output/denseteacher_fcos_r50_fpn_coco_semi010/model_final.pdparams --infer_img=demo/000000014439.jpg
```

### 部署

部署可以使用半监督检测配置文件，也可以使用基础检测器的配置文件去部署和使用。

```bash
# 导出模型
CUDA_VISIBLE_DEVICES=0 python tools/export_model.py -c ssod/denseteacher/denseteacher_fcos_r50_fpn_coco_semi010.yml -o weights=https://paddledet.bj.bcebos.com/models/denseteacher_fcos_r50_fpn_coco_semi010.pdparams

# 导出权重预测
CUDA_VISIBLE_DEVICES=0 python deploy/python/infer.py --model_dir=output_inference/denseteacher_fcos_r50_fpn_coco_semi010 --image_file=demo/000000014439_640x640.jpg --device=GPU

# 部署测速
CUDA_VISIBLE_DEVICES=0 python deploy/python/infer.py --model_dir=output_inference/denseteacher_fcos_r50_fpn_coco_semi010 --image_file=demo/000000014439_640x640.jpg --device=GPU --run_benchmark=True # --run_mode=trt_fp16

# 导出ONNX
paddle2onnx --model_dir output_inference/denseteacher_fcos_r50_fpn_coco_semi010/ --model_filename model.pdmodel --params_filename model.pdiparams --opset_version 12 --save_file denseteacher_fcos_r50_fpn_coco_semi010.onnx
```


## 引用

```
 @article{denseteacher2022,
  title={Dense Teacher: Dense Pseudo-Labels for Semi-supervised Object Detection},
  author={Hongyu Zhou, Zheng Ge, Songtao Liu, Weixin Mao, Zeming Li, Haiyan Yu, Jian Sun},
  journal={arXiv preprint arXiv:2207.02541},
  year={2022}
}
```