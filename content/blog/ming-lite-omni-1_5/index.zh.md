---
title: "Ming-Lite-Omni V1.5 介绍"
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



## 前言
本次发布的 Ming-lite-omni V1.5 是对 Ming-lite-omni 全模态能力的一次全面升级, 在包括图文理解、文档理解、视频理解、语音理解和合成、图像生成和编辑等任务上均有明显提升。Ming-lite-omni V1.5 基于Ling-lite-1.5 构建，总参数20.3B, MoE部分激活参数为3B，在各模态基准测试中，与业界领先的模型相比展现出极具竞争力的结果。下面是我们本次更新在部分重要指标和模型架构上的提升的展示。


<div style="text-align:center">
  <img src="https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/0.webp" alt="Image description" />
  <p style="font-size:14px; color:gray;">性能对比图</p>
</div>


<div style="text-align:center">
  <img src="https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/1.webp" alt="Image description" />
  <p style="font-size:14px; color:gray;">模型架构图</p>
</div>




## 详细介绍
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
3. 引入<u>感知增强策略</u>。通过优化结构感知能力，如分割和关键点检测，提升模型对画面细节和空间关系的理解，增强编辑和生成过程的结构可控性，显著提高评测指标中与位置、结构、数量相关的得分，详见 [表A](#table1)。
4. 引入<u>多任务协同学习策略</u>。通过联合训练链路实现生成与编辑的相互促进，将分割任务转化为彩色上色编辑任务，显著提升分割指标和图像局部编辑的精度与可控性，使编辑区域边缘更光滑，详见 [图A](#image1)。
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


## Demo展示

### 图文对话

| 图片 | 问题 | Ming-lite-omni回答 | Ming-lite-omni V1.5回答 |
| --- | --- | --- | --- |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/2.jpg) | 这张图片看起来设计感很强烈，可以详细描述一下它的各个设计元素？ | 这张图片展示了一款高端游戏鼠标，具有现代感和科技感的设计。鼠标主体为黑色，表面光滑且具有金属质感。鼠标顶部有一个蓝色的发光标志，标志下方是鼠标滚轮和左右按键区域。鼠标侧面有蓝色的灯光装饰，增加了视觉吸引力。鼠标底部有防滑纹理设计，确保使用时的稳定性。整体设计简洁而富有未来感，适合游戏玩家使用。 | 这张图片展示了一款设计感强烈的游戏鼠标，具有以下设计元素： <br> 1. **颜色和材质**：   - 鼠标主体为黑色，表面光滑且具有金属质感。   - 鼠标边缘和按钮部分有蓝色LED灯带，增加了科技感和视觉吸引力。 <br> 2. **形状和结构**：   - 鼠标整体呈流线型设计，顶部略微倾斜，便于手部自然放置。   - 鼠标侧面有明显的曲线设计，符合人体工程学，提供舒适的握持感。 <br> 3. **按钮和滚轮**：   - 鼠标左侧有一个滚轮，滚轮上方有蓝色LED灯带。   - 鼠标右侧有额外的按钮，这些按钮也有蓝色LED灯带，增加了功能性和美观性。 <br> 4. **品牌标识**：   - 鼠标底部中央有一个发光的“G”标志，这是该品牌的标志性设计元素。 <br> 5. **灯光效果**：   - 鼠标上的蓝色LED灯带在暗色背景下显得尤为突出，增强了科技感和未来感。 <br> 6. **纹理和细节**：   - 鼠标顶部有纹理设计，可能是为了增加防滑效果。   - 鼠标侧面有细长的纹理设计，增加了视觉上的层次感。 <br> 整体来看，这款鼠标的设计结合了现代科技感和人体工程学设计，既美观又实用，适合游戏玩家使用。 |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/3.webp) | 图中的书法为什么好？ | 这幅书法作品展示了高超的书法技巧和深厚的文化底蕴。笔法流畅自然，字形结构严谨，布局合理，体现了书法家的深厚功力和艺术修养。 | 这幅书法作品展示了几个显著的特点，使其成为一件优秀的书法作品： <br> 1. **笔法精湛**：书法的笔法流畅自然，线条粗细变化丰富，表现出书法家的深厚功力和熟练技巧。 <br> 2. **结构严谨**：每个字的结构都经过精心设计，笔画之间的衔接自然，字与字之间保持了良好的平衡和协调。 <br> 3. **章法布局**：整幅作品的章法布局合理，行距和字距均匀，给人一种整齐有序的感觉。 <br> 4. **墨色变化**：墨色的浓淡变化自然，增添了作品的层次感和立体感。 <br> 5. **内容优美**：书法内容是一篇优美的古文，文字内容富有哲理和文学价值，增加了作品的文化内涵。 <br> 6. **印章点缀**：作品上有红色的印章，印章的使用不仅增加了作品的装饰性，还起到了点缀和平衡的作用。 <br> 这些特点共同构成了这幅书法作品的高超艺术价值，使其成为一件值得欣赏和学习的佳作。 |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/4.webp) | 图中这款手机适合给父母买吗？ | 这款手机外观简约时尚，适合追求简约风格的父母使用。相机配置也较为先进，适合喜欢拍照的用户。 | 这款手机适合给父母购买，原因如下： <br> 1. **外观设计**：手机采用简约时尚的设计，白色机身显得干净利落，符合现代审美。 <br> 2. **摄像头配置**：后置摄像头设计独特，排列整齐，可能提供良好的拍照体验。 <br> 3. **品牌**：荣耀（Honor）品牌在智能手机市场上有良好的口碑，提供稳定可靠的产品。 <br> 4. **功能**：通常这类手机会配备大容量电池和长续航能力，适合日常使用。 <br> 5. **易用性**：可能提供简洁的用户界面和易于操作的功能，适合不太熟悉复杂操作的用户。 <br> 6. **性价比**：荣耀手机通常以高性价比著称，适合预算有限但希望获得良好体验的用户。 <br> 当然，具体是否适合还需考虑父母的实际需求和偏好。 |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/5.webp) | 你能告诉我该图所示是哪类疾病的特征吗？治疗该疾病的药物有哪些？ | 该图展示的是痤疮（俗称青春痘）的特征，包括红斑、瘙痒和鳞屑。治疗痤疮的药物包括外用抗生素（如克林霉素）、维A酸类药物（如阿达帕林）、过氧化苯甲酰，以及口服药物如异维A酸等。 | 该图显示的是痤疮（俗称青春痘）的特征。痤疮是一种常见的皮肤疾病，通常出现在青春期，但也可能影响成年人。 <br> 治疗痤疮的药物包括： <br> 1. **外用药物**：   - **抗生素**：如克林霉素、红霉素等，用于减少皮肤细菌。   - **过氧化苯甲酰**：具有抗菌和抗炎作用。   - **维A酸类药物**：如维A酸乳膏，用于促进皮肤细胞更新。   - **水杨酸**：帮助去除死皮细胞和清洁毛孔。 <br> 2. **口服药物**：   - **抗生素**：如多西环素，用于严重痤疮。   - **异维A酸**：用于重度痤疮。   - **避孕药**：对于女性患者，特别是激素相关的痤疮。 <br> 3. **其他治疗**：   - **光疗**：如蓝光治疗。   - **化学剥皮**：用于去除皮肤表层。 <br> 建议在医生指导下选择适合的治疗方案。 |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/6.webp) | 如何使用图中软件购买机票？请列出具体操作步骤 | 使用图中软件购买机票的具体操作步骤如下： <br> 1. 打开携程app。   2. 点击“机票”选项。   3. 在机票页面选择出发地、目的地、出发日期和返回日期。   4. 点击搜索按钮，查看可用的航班信息。   5. 选择合适的航班后，点击“预订”按钮。   6. 填写乘客信息并选择支付方式。   7. 确认订单信息并提交支付，完成购票。 | 使用携程软件购买机票的具体操作步骤如下： <br> 1. **下载并安装携程App**：   - 如果您还没有安装携程App，可以在应用商店（如苹果App Store或安卓应用市场）搜索“携程旅行”并下载安装。 <br> 2. **打开携程App**：   - 安装完成后，打开携程App。 <br> 3. **选择“机票”选项**：   - 在首页的导航栏中，点击“机票”选项。 <br> 4. **输入出发地和目的地**：   - 在机票预订页面，输入您的出发地和目的地。 <br> 5. **选择出发日期和返回日期**：   - 选择您的出发日期和返回日期（如果是往返票）。 <br> 6. **选择乘客信息**：   - 输入乘客人数和座位类型（成人、儿童、婴儿）。 <br> 7. **选择航班**：   - 浏览并选择您喜欢的航班。 <br> 8. **填写乘客信息**：   - 输入乘客的姓名、身份证号码、联系方式等信息。 <br> 9. **选择支付方式**：   - 选择您喜欢的支付方式（如支付宝、微信支付、银行卡等）。 <br> 10. **确认订单并支付**：   - 确认订单信息无误后，点击“支付”按钮完成支付。 <br> 11. **接收电子机票**：   - 支付成功后，您将收到电子机票，可以通过App内的“我的订单”查看或下载。 <br> 通过以上步骤，您就可以使用携程App轻松购买机票了。 |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/7.webp) | 根据图表计算样本之间的 Δct 在哪一个引物下相差最小？ | 根据图表计算样本之间的 Δct 在 cGAS-qF+qR 引物下相差最小。 | 在 **EGFP-cGAS单克隆** 引物下，样本之间的 Δct 相差最小。 <br>  <br> **计算过程：** <br> **1. 理解 Δct 的含义：**   Δct 是目标基因（cGAS-qF+qR 或 EGFP-cGAS）与内参基因（cTubulin）的相对表达量差值。Δct 越小，说明目标基因与内参基因的表达量差异越小。 <br> **2. 比较不同引物下的 Δct：** <br> - **cGAS-qF+qR 引物：**   - 野生型：Δct = 8.06   - EGFP-cGAS单克隆：Δct = -0.15   - 差异：8.06 - (-0.15) = 8.21 <br> - **EGFP-cGAS单克隆 引物：**   - 野生型：Δct = 10.44   - EGFP-cGAS单克隆：Δct = -2.9   - 差异：10.44 - (-2.9) = 13.34 <br> - **EGFP-cGAS-qF+qR 引物：**   - 野生型：Δct = 21.43   - EGFP-cGAS单克隆：Δct = -13.45   - 差异：21.43 - (-13.45) = 34.88 <br> **3. 结论：**   在 **EGFP-cGAS单克隆** 引物下，样本之间的 Δct 差异最小（Δct 差异为 13.34），说明该引物下样本之间的表达量差异最小。 <br>  <br>  **总结：**   在 **EGFP-cGAS单克隆** 引物下，样本之间的 Δct 相差最小。 |


### 图像编辑

{{<video src="https://qianwen-res.oss-accelerate.aliyuncs.com/Qwen-VLo/head_en.mov" muted="true" width="100%" loop="true" autoplay="true">}}

### 图像生成

{{< fullwidth class="example-container" >}}
{{< example data="cases/seg.json" hide=false next=true scroll=true >}}
{{< /fullwidth >}}

<!-- 
引入<u>多任务协同学习策略</u>。通过联合训练链路实现生成与编辑的相互促进，将分割任务转化为彩色上色编辑任务，显著提升分割指标和图像局部编辑的精度与可控性，使编辑区域边缘更光滑：

<a id="image1"></a>
<div style="text-align:center">
  <img src="https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/0.webp" alt="Image description" />
  【待补充】
  <p style="font-size:14px; color:gray;">图A【占位】</p>
</div> -->

引入<u>感知增强策略</u>。通过优化结构感知能力，如分割和关键点检测，提升模型对画面细节和空间关系的理解，增强编辑和生成过程的结构可控性，显著提高评测指标中与位置、结构、数量相关的得分：
<a id="table1"></a>
|   | 1-Obj | 2-Obj | Counting | Colors | Position | Color Attr | Avg. |
|---| ---   | ---   | ---      | --- |--- |--- |--- |
|Ming-lite-omni| 0.99   | 0.77   | 0.68      | 0.78 | 0.46 |0.42 |0.64 |
|Ming-lite-omni V1.5| 0.99   | 0.93   | 0.86      | 0.87 |0.90 |0.66 |0.87 |  

<!-- <div style="text-align:center">
  <p style="font-size:14px; color:gray;">表A</p>
</div> -->

| 原图 | 生成的深度图 | 生成的检测框 | 生成的边缘轮廓 |
| --- | --- | --- | --- |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/gen/1752466889319-bd19acce-c07d-4664-9890-41e4dff1ba8d.webp) | ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/gen/1752466903529-996bcd35-a9a0-484b-98bf-2f2468f4df42.webp) | ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/gen/1752466895795-1955ead5-6d94-4142-8d7b-e265352d2bcb.webp) | ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/gen/1752467020122-ad8b436c-bb33-4ef0-85b8-cf45ae8c9be1.webp) |





### 语音合成
| 输入文本 | Ming-lite-omni | Ming-lite-omni V1.5 |
| --- | --- | --- |
| Tyler, Lucy, Michelle, we're going to space! | <audio controls><source src="https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/tts_en1_v1.wav" type="audio/wav"></audio>  | <audio controls><source src="https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/tts_en1_v1.5.wav" type="audio/wav"></audio>  |
| what is the forecast for California for rain | <audio controls><source src="https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/tts_en2_v1.wav" type="audio/wav"></audio>  | <audio controls><source src="https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/tts_en2_v1.5.wav" type="audio/wav"></audio>  |
| Television can make you dumb, but it can also be good education. | <audio controls><source src="https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/tts_en3_v1.wav" type="audio/wav"></audio>  | <audio controls><source src="https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/tts_en3_v1.5.wav" type="audio/wav"></audio>  |



### OCR文档理解
| 图片 | 问题 | Ming-lite-omni回答 | Ming-lite-omni V1.5回答 |
| --- | --- | --- | --- |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/9.webp) | 在判断题第95题中，考生答题的手写内容是什么？若考生的手写答案为√则输出对，若考生的手写答案为×则输出错，请输出对或错 | 对 | 错 |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/10.webp) | Output the content of the CAPTCHA directly. Characters are only numbers and uppercase/lowercase letters. | 9743 | 9742 |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/11.webp) | What is the Latex tag for mathematical expression in images? | A C = C B \therefore C D / / B E | A C = C B \because C D / / B E |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/12.webp) | 从图中提取: 日期 | 日期：2019年12月03日05时08分   从图中提取: 里程   里程：23.9公里   从图中提取: 金额   金额：52.6元   从图中提取: 状态   状态：   从图中提取: 客票印务   客票印务2019年12月印900000卷   从图中提取: 号码   号码从00000001至90000000止 | 日期: 2021年03月05日 |
| ![](https://gcore.jsdelivr.net/gh/biao-gong/static@main/0715/13.webp) | What is the difference in value between Lamb and Corn?' You FIRST think about the reasoning process as an internal monologue and then provide the final answer. The reasoning process MUST BE enclosed within <think> </think> tags. The final answer MUST BE put in \\boxed{}.' | 0.37 | <think><br/>The value for Lamb is 103.7 and for Corn is 103.13. To find the difference, subtract the smaller value from the larger value: 103.7 - 103.13 = 0.57.<br/></think><br/>\boxed{0.57} |


## 开始使用 Ming-lite-omni v1.5

Ming-lite-omni v1.5的模型和代码已开源，欢迎大家试用、反馈和交流。后续我们会持续优化Ming-lite-omni，持续提升在全模态的效果同时，让Ming-lite-omni更加轻量化，同时强化Ming-lite-omni的多模推理能力和生成能力。
  - Github: https://github.com/inclusionAI/Ming
  - Hugging Face: https://huggingface.co/inclusionAI/Ming-Lite-Omni-1.5 
  - ModelScope: https://www.modelscope.cn/models/inclusionAI/Ming-Lite-Omni-1.5