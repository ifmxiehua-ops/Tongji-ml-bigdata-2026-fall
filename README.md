# 同济大学 机器学习与大数据计算课程作业

同济大学 2026–2027 学年第一学期《机器学习与大数据计算》课程作业仓库，包含平时作业和期末小组项目。

## 期末项目：口腔影像分析与多模态报告生成

项目分为两部分。第一部分用有标注的口腔图像训练独立的分类、检测和分割模型。第二部分使用 COde 图文数据对 Qwen2.5-VL-3B-Instruct 进行 LoRA 微调，根据口内照片、患者主诉和已知病历信息生成中文口腔报告草稿。

**Built with Qwen**

## 首轮训练计划

首轮安排 **9 项模型训练**，覆盖 **8 个独立数据集**。其中 C004 的疾病检测标注和牙齿分割标注分别用于两个任务。

| 编号 | 训练任务 | 数据集 | 模型 | 当前进度 |
| --- | --- | --- | --- | --- |
| M01 | 根尖周病变二分类 | C006 根尖周病变数据集 | ResNet18 | 首轮计划 |
| M02 | 口腔黏膜四分类 | C008 SMART-OM | ResNet18 | 首轮计划 |
| M03 | 颈椎成熟度六阶段分类 | C005 Aariz | ResNet18 | 首轮计划 |
| M04 | 骨吸收相关等级三分类 | C017 BRAR | ResNet18 | 首轮计划 |
| M05 | 口内照片龋坏检测 | C001 口内龋齿图像数据集 | Faster R-CNN（ResNet50-FPN） | 首轮计划 |
| M06 | 儿童全景片疾病检测 | C004 儿童疾病检测子包 | Faster R-CNN（ResNet50-FPN） | 首轮计划 |
| M08 | 全景片牙齿语义分割 | C004 牙齿分割子包 | U-Net（ResNet18 编码器） | 首轮计划 |
| M09 | 根尖片牙齿语义分割 | C007 DenPAR | U-Net（ResNet18 编码器） | 首轮计划 |
| M13 | 中文口腔报告生成 | C018 COde | Qwen2.5-VL-3B-Instruct + LoRA | 已完成微调和网页推理演示 |

视觉任务计划分别使用 Macro-F1、mAP、Dice 和 IoU 评估分类、检测和分割表现。报告任务将模型输出与 COde 测试病例的参考病历对照，重点核对牙位、检查所见和处理记录。

## COde 多模态微调

使用 LLaMA-Factory v0.9.3 在 AutoDL 的单张 NVIDIA GeForce RTX 4090 上完成监督微调。清洗训练数据时按患者和图像哈希检查与测试集的重叠，最终使用 **9,182 条训练样本**。

| 参数或结果 | 数值 |
| --- | --- |
| 基座模型 | Qwen2.5-VL-3B-Instruct |
| 微调方式 | LoRA，rank 8，BF16 |
| 学习率 | `5e-4` |
| 单卡批大小 / 梯度累积 | `1 / 16` |
| 最大序列长度 | `4096` |
| 训练轮数 / 优化步数 | `1 / 574` |
| 训练耗时 | 约 2 小时 12 分钟 |
| 最终训练损失 | `0.5792` |

训练产物为 `adapter_model.safetensors`（约 57.2 MiB）和 `adapter_config.json`。推理时将这两个文件与同版本的 Qwen 基座一起加载。

## 网页演示

在已安装 LLaMA-Factory v0.9.3 的 Linux GPU 环境中，准备原版 Qwen 基座和 LoRA 文件。将下面的配置保存为 `inference.yaml`，按实际位置修改两个模型路径。

```yaml
model_name_or_path: ./models/Qwen2.5-VL-3B-Instruct
adapter_name_or_path: ./weights/diagnostic_lora
template: qwen2_vl
finetuning_type: lora
infer_backend: huggingface
infer_dtype: bfloat16
image_max_pixels: 200704
trust_remote_code: true
```

```bash
GRADIO_SERVER_NAME=127.0.0.1 GRADIO_SERVER_PORT=7860 \
  llamafactory-cli webchat inference.yaml
```

打开 `http://127.0.0.1:7860`，在 **Image** 栏上传口内照片，在文字框输入主诉和已确认的病历信息，再点击 **Run**。如果服务运行在 AutoDL，可通过 SSH 端口转发从本地浏览器访问。

在一条复诊测试样本中，输入已确认的 **23 号牙**后，模型生成的「临时牙在位、软组织无异常」与参考病历一致。其他试跑中也出现了牙位写错、补写处理经过等情况。展示时将模型报告和参考病历并排对照，生成内容作为课程实验中的报告草稿，由人工核对。

## 项目来源

- 多模态训练数据：[Zirak AI COde 数据集](https://huggingface.co/datasets/zirak-ai/COde)
- 基座模型：[Qwen2.5-VL-3B-Instruct](https://huggingface.co/Qwen/Qwen2.5-VL-3B-Instruct) · [Qwen 研究许可](https://huggingface.co/Qwen/Qwen2.5-VL-3B-Instruct/blob/main/LICENSE)
- 训练和推理框架：[LLaMA-Factory v0.9.3](https://github.com/hiyouga/LLaMA-Factory/tree/v0.9.3)

本项目用于课程学习与实验评估。数据集、基座模型和相关权重的使用与分发遵循各自的许可及使用条款。
