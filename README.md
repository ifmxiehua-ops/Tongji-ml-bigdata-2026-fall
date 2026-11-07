# 口腔影像分析与多模态报告生成

同济大学 2026–2027 学年第一学期《机器学习与大数据计算》课程期末小组作业。

## 项目简介

本项目围绕口腔照片和牙科 X 线图像开展机器学习实验，包含口腔影像分析和中文口腔报告生成两部分。

- **口腔影像分析**使用有标注的口腔图像开展分类、目标检测和语义分割实验，任务包括口腔病变分类、龋坏位置检测和牙齿区域分割。
- **多模态报告生成**使用 COde 图文数据对 Qwen2.5-VL-3B-Instruct 进行 LoRA 微调，以口内照片、患者主诉和已有病历信息为输入，生成结构化的中文口腔报告草稿。

## 训练任务与数据集

项目首轮设置 **9 项训练任务**，涉及 **8 个独立数据集**。其中 C004 的疾病检测子包和牙齿分割子包分别用于两个任务。

| 训练任务 | 使用的数据集 | 模型结构 |
| --- | --- | --- |
| 根尖周病变二分类 | C006 根尖周病变数据集 | ResNet18 |
| 口腔黏膜四分类 | C008 SMART-OM | ResNet18 |
| 颈椎成熟度六阶段分类 | C005 Aariz | ResNet18 |
| 骨吸收相关等级三分类 | C017 BRAR | ResNet18 |
| 口内照片龋坏检测 | C001 口内龋齿图像数据集 | Faster R-CNN，ResNet50-FPN 骨干 |
| 儿童全景片疾病检测 | C004 儿童疾病检测子包 | Faster R-CNN，ResNet50-FPN 骨干 |
| 全景片牙齿语义分割 | C004 牙齿分割子包 | U-Net，ResNet18 编码器 |
| 根尖片牙齿语义分割 | C007 DenPAR | U-Net，ResNet18 编码器 |
| 中文口腔报告生成 | C018 COde | Qwen2.5-VL-3B-Instruct + LoRA |

## 实验方法

视觉部分按标注类型组织训练：图像级标签用于分类，标注框用于目标检测，像素掩膜用于语义分割。各任务分别训练模型，输出类别、检测框或分割掩膜。

报告生成部分使用 LLaMA-Factory 在单张 RTX 4090 上完成 LoRA 微调。课程实验使用测试病例，将模型生成的报告与参考病历对照，重点核对牙位、检查所见和处理记录。

**Built with Qwen**

[COde 数据集](https://huggingface.co/datasets/zirak-ai/COde) · [Qwen2.5-VL-3B-Instruct](https://huggingface.co/Qwen/Qwen2.5-VL-3B-Instruct) · [LLaMA-Factory](https://github.com/hiyouga/LlamaFactory)
