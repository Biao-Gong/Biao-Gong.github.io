---
title: "Ming-Lite-Omni V1.5"
date: 2025-07-17T00:00:03+08:00
weight: 1
math: true
show_reading_time: true
show_bread_crumbs: true
show_post_nav_links: false # the prev/next after the content
show_code_copy_buttons: true
show_word_count: true
---

{{< button href="https://github.com/inclusionAI/Ming " label="GITHUB" external=true >}} 🤗 <a href="https://huggingface.co/inclusionAI/Ming-Lite-Omni-1.5">Hugging Face</a>｜ 🤖 <a href="https://www.modelscope.cn/models/inclusionAI/Ming-Lite-Omni-1.5">ModelScope</a>



# 前言
本次发布的 Ming-lite-omni V1.5 是对 Ming-lite-omni 全模态能力的一次全面升级, 在包括图文理解、文档理解、视频理解、语音理解和合成、图像生成和编辑等任务上均有明显提升。Ming-lite-omni V1.5 基于Ling-lite-1.5 构建，总参数20.3B, MoE部分激活参数为3B，在各模态基准测试中，与业界领先的模型相比展现出极具竞争力的结果。下面是我们本次更新在部分重要指标和模型架构上的提升的展示。


<div style="text-align:center">
  <img src="https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/0.webp" alt="Image description" />
  <p style="font-size:14px; color:gray;">性能对比图</p>
</div>


<div style="text-align:center">
  <img src="https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/1.webp" alt="Image description" />
  <p style="font-size:14px; color:gray;">模型架构图</p>
</div>




# 详细介绍
为了实现这样的提升，我们将自研方案与学术界/开源社区的最新进展相结合，在以下几个部分做了有效尝试，并取得多个重要结论。

**图像/文本/视频/语音理解**

1. MRoPE 时空感知位置编码。引入了MRoPE，通过时间、高度、宽度三维分块位置编码，赋予模型时空感知能力，实现高效跨模态联合建模，提升对视频、复杂图像场景的理解精度。
2. 高效全参数训练策略。优化学习率与多模态数据配比，将理解阶段需<u>分步冻结/解冻 LLM 的预训练流程</u>，升级为<u>高效全参数训练</u>，训练周期缩短 26.5%，保持性能无损。
3. 针对视频理解任务，采用从短视频到长视频的课程学习策略，逐步提升模型处理长视频的复杂度。
4. 针对复杂文档理解任务，引入 Chain-of-Thought 策略分步骤构建结构化推理路径，有效提升模型对复杂问题的解决能力。
5. 全方位数据升级  
    - 预训练阶段
      - 新增文本实体结构化数据，补全图谱盲区。
      - 扩充高质量商品数据，提升通识能力。
    - 指令微调阶段
      - 提升细粒度视觉感知（目标计数/颜色/场景识别）数据精准性。
      - 提高垂类识别（动植物/车辆/食材等）数据深度。
      - 从数据角度优化跨学科复杂图文推理能力。
      - 针对语音理解任务，将领域、主题、语种（包括方言）等信息引入到语音理解任务的指令文本中，增强模型的理解表现，实现对中英文，粤语，四川话，上海话，闽南语等方言的全面支持。

**图像/语音生成**

1. 图像生成侧采用<u>双分支解耦策略</u>提升模型对参考图的可学习参数量。具体来说，在图像进入 DiT 之前，我们使用不同的权重对参考图像与噪声图像分别进行处理，并增加额外两层transformer layers作为refiner进一步增强这一效果。
2. 为了解决图像编辑时的人物ID及场景ID一致性问题，我们新增了<u>ID & Scene Consistency Loss</u>，增大目标图编辑区域的权重、增大参考图非编辑区域的参考强度、降低参考图编辑区域的参考强度。
3. 引入<u>感知增强策略</u>。通过优化结构感知能力，如分割和关键点检测，提升模型对画面细节和空间关系的理解，增强编辑和生成过程的结构可控性，显著提高评测指标中与位置、结构、数量相关的得分，**<font style="background-color:#FBDE28;">详见表X</font>**。
4. 引入<u>多任务协同学习策略</u>。通过联合训练链路实现生成与编辑的相互促进，将分割任务转化为彩色上色编辑任务，显著提升分割指标和图像局部编辑的精度与可控性，使编辑区域边缘更光滑，**<font style="background-color:#FBDE28;">详见图X</font>**。
5. 语音生成解码器方面，我们实现了全新的音频解码器，直接接受来自LLM的输出特征实现上下文感知。
6. 语音生成效率方面，为了提高韵律性能和实时生成能力，我们将离散的Audio codec token进行BPE编码，使得音频帧率降低了35%。
7. 全方位数据升级
    - 获取高质量人物图像数据，标准包括：图像分辨率/质量、人脸细粒度、人脸大小等。
    - 采集名人数据，并做质量筛选和人脸裁剪获取名人图像数据。
    - 构建边缘图、分割图、文字图、人物表情图等训练子集，扩充模型能力边界。

**用户偏好对齐**

为了保证我们模型的真实使用体验与常用Benchmark上的提升一致，我们构建了体验评测集，在内部进行多模型的人工对抗评分。得益于高质量的对齐偏好数据构建， Ming-lite-omni v1.5 在图文问答的内容准确性（低幻觉率）、相关性、格式美观性以及表述流畅性方面相比领先模型展现出一定优势， Ming-lite-omni v1.5在内部对抗评测集上相比Ming-lite-omni v1 胜和率为 87.07%, 使用体验得到了明显优化。  


| 评测基准 | 评测维度          | Qwen2.5-VL-7B | Ming-Omni-Lite V1.5 |
|------|---------------|---------------|----------------|
| 体验   | 均分            | 4.274         | 4.365          |
|      | 自建体验评测集_相关性   | 4.308         | 4.5            |
|      | 自建体验评测集_流畅性   | 4.765         | 4.91           |
|      | 自建体验评测集_内容丰富性 | 3.828         | 3.69           |
|      | 自建体验评测集_格式合理性 | 4.727         | 4.8            |
|      | 自建体验评测集_正确性   | 3.741         | 3.92           |


# Demo展示

## 图文对话

## 图像编辑

## 图像检测分割

## 图像生成

## 语音合成

## OCR文档理解



# 开始使用 Ming-lite-omni v1.5

Ming-lite-omni v1.5的模型和代码已开源，欢迎大家试用、反馈和交流。后续我们会持续优化Ming-lite-omni，持续提升在全模态的效果同时，让Ming-lite-omni更加轻量化，同时强化Ming-lite-omni的多模推理能力和生成能力。
  - Github: https://github.com/inclusionAI/Ming
  - Hugging Face: https://huggingface.co/inclusionAI/Ming-Lite-Omni-1.5 
  - ModelScope: https://www.modelscope.cn/models/inclusionAI/Ming-Lite-Omni-1.5