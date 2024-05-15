
## 前言

视频生成文字，文字生成视频的

要求：找到应用的功能，出md，录屏和泳道图（主要体现和大模型的交互，目前看至少涉及两个部分：视频流的理解和如何生成视频）


sora
https://openai.com/index/video-generation-models-as-world-simulators/

sora依然无法解决视频内容生产的全链路问题？




ppt总结：

Sora

市场需求分析/竞品分析

YAYI2.0大模型

1. **YAYI2.0 大模型深度赋能**：
    
    - 介绍了YAYI2.0的智能写作和智能视频生产能力，包括文生图、图生视频、AI口型驱动等技术。
2. **YAYI2.0 大模型深度赋能-视频无中生有**：
    
    - 展示了AI一键成片技术，能够基于文本和场景自动生成专业化AI合成影片。
3. **YAYI2.0 大模型深度赋能-智能编目**：
    
    - 介绍了基于MoE轻量级多模态大模型的视频智能拆条和编目技术。
4. **YAYI2.0 大模型深度赋能-视频有中生优**：
    
    - 描述了AI智能一键剪辑成片的功能，结合了多模态大模型的感知与认知能力。


智能媒资库

### open sora
https://github.com/hpcaitech/Open-Sora
试用： https://huggingface.co/spaces/hpcai-tech/open-sora




### mora
https://github.com/lichao-sun/Mora
通过多代理框架实现通用视频生成

![test image](https://github.com/lichao-sun/Mora/raw/main/image/method.jpg)
### open-sora plan
https://github.com/PKU-YuanGroup/Open-Sora-Plan
基于转换器的文本到视频扩散系统，使用 T5 的文本嵌入进行训练。

### dynamiCrafter
https://github.com/Doubiiu/DynamiCrafter
线上试体验： https://huggingface.co/spaces/Doubiiu/DynamiCrafter

静态图像 转为 动态视频
核心：双流图像注入机制，通过结合文本信息和图像内容，使用先进的AI模型和机制，将静态图像转换成动态视频
- **文本对齐和动态置信度计算**：使用GPT-4来分析文本描述，并将其与视频内容进行匹配，确保生成的视频与文本描述在语义上是一致的。
- **扩散模型**：利用扩散模型生成视频帧，这是一种生成模型，能够生成高质量的图像和视频内容。
- **视觉细节指导**：在生成视频时，使用视觉细节指导来确保生成的视频帧与输入图像在视觉上保持一致性。
- **帧插值**：通过在不同的输入图像之间进行帧插值，生成平滑的视频过渡效果。
- **GPU加速**：使用高性能的GPU（如RTX 4090）来加速模型的推理过程，减少内存消耗。




产品设计/思路
