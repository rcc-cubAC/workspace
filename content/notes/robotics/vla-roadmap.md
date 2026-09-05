---
title: 机器人与 VLA 入门路线
date: 2026-09-05T17:00:00+08:00
draft: false
tags:
  - 机器人
  - VLA
  - 入门路线
description: 面向已有深度学习与三维图形基础的读者，从机器人学基础、模仿学习到视觉语言动作模型的分阶段学习路线，所有链接均已核对．
---

## 这条路线的前提

默认读者已经熟悉深度学习的基本功，包括 Transformer、扩散模型与流匹配，也熟悉三维几何与图形学．缺的是三样东西，机器人学的基本概念、模仿学习这一套方法论，以及从数据采集到真机部署的完整流程．路线按这三样东西的依赖关系排列，每个阶段都给出要读的材料与要动手做的事．全部走完大约需要三个月，前两个阶段可以压缩，第三阶段不要压缩．

## 先看整体图景

VLA 是 Vision-Language-Action 模型的简称．它的输入是相机图像、语言指令与机器人本体状态，例如关节角与夹爪开合度；输出是接下来一小段时间的动作序列，例如未来若干步的关节目标位置或末端位姿增量．它本质上是一个以视觉语言模型为主干、用机器人示范数据微调出来的策略网络．

这条技术路线的演进大致如下．

- **单任务**策略．[ACT](https://www.roboticsproceedings.org/rss19/p016.pdf) 与 [Diffusion Policy](https://www.roboticsproceedings.org/rss19/p026.pdf) 都发表于 RSS 2023，从零训练，只用几十到几百条示范就能学会一个任务．它们确立了两个沿用至今的设计，动作分块与用生成模型建模多峰动作分布．
- **多任务**策略．[RT-1](https://www.roboticsproceedings.org/rss19/p025.pdf) 同样发表于 RSS 2023，用 Transformer 在十几万条真机示范上训练，把连续动作离散成词元，证明了数据规模能换来任务泛化．
- **视觉语言模型**作主干．[RT-2](https://proceedings.mlr.press/v229/zitkovich23a.html) 发表于 CoRL 2023，直接把动作当成文本词元，与网络图文数据一起微调视觉语言模型，语义泛化能力明显好于从零训练．VLA 这个名字就来自这篇论文．
- **跨本体**数据．[Open X-Embodiment](https://arxiv.org/abs/2310.08864) 汇集了二十多种机器人的上百万条轨迹，在其上训练的 RT-X 模型说明不同机器人的数据可以互相帮助．
- **开源**复现．[Octo](https://www.roboticsproceedings.org/rss20/p090.pdf) 发表于 RSS 2024，[OpenVLA](https://proceedings.mlr.press/v270/kim25c.html) 发表于 CoRL 2024，两者都开放权重与训练代码，是入门时最容易上手的两个基线．
- **流匹配**动作头．[π0](https://www.roboticsproceedings.org/rss21/p010.html) 发表于 RSS 2025，在视觉语言模型之后接一个流匹配的动作专家，输出高频连续动作块，配套代码 [openpi](https://github.com/Physical-Intelligence/openpi) 开源．后续的 [π0.5](https://www.physicalintelligence.company/blog/pi05) 强调开放环境泛化．
- **通用**与人形．[GR00T N1](https://github.com/NVIDIA/Isaac-GR00T)、[Gemini Robotics](https://deepmind.google/models/gemini-robotics/) 与 [Helix](https://www.figure.ai/news/helix) 代表 2025 年之后的方向，多为双系统结构，慢速的视觉语言推理配合高频的动作生成．[SmolVLA](https://arxiv.org/abs/2506.01844) 则走小模型路线，用社区数据训练，消费级显卡就能微调．

读论文时，抓住下面几个设计维度，每篇论文都可以放进这张表里比较．

| 维度 | 主要选项 |
| --- | --- |
| 主干 | 从零训练的 Transformer，或预训练的视觉语言模型 |
| 动作表示 | 离散词元，或用扩散、流匹配生成的连续动作块 |
| 观测输入 | 单目或多目 RGB 图像，或点云等三维表示 |
| 训练数据 | 单机器人示范，或跨本体的混合数据集 |
| 训练方式 | 从零训练，或先预训练再针对目标机器人微调 |
| 推理形式 | 逐步输出单个动作，或一次输出一个动作块再开环执行 |

## 第一阶段，机器人学基础

目标是看懂机器人示范数据里每一列的含义，知道策略输出的动作会被怎样执行．内容不多，两到三周足够．

要掌握的概念包括刚体位姿与 $\mathrm{SE}(3)$ 变换、关节空间与任务空间、正运动学、雅可比矩阵、微分逆运动学，以及位置控制与阻抗控制的区别．核心关系只有一条，末端速度与关节速度由雅可比矩阵联系，

$$
\dot{\mathbf{x}} = \mathbf{J}(\mathbf{q})\,\dot{\mathbf{q}}, \qquad \dot{\mathbf{q}} = \mathbf{J}(\mathbf{q})^{+}\,\dot{\mathbf{x}}.
$$

策略网络输出的往往是末端位姿增量或关节目标位置，底层控制器再把它变成电机指令．理解这一层，才能判断一个数据集的动作空间是否与自己的机器人兼容．

读的材料按优先级排列．

- [Robotic Manipulation](https://manipulation.csail.mit.edu/)，MIT 的操作课程讲义，前六章讲位姿、基本抓取与微分逆运动学，例子全部可以在浏览器里运行．这是最贴近 VLA 需要的教材．
- [Modern Robotics](https://hades.mech.northwestern.edu/index.php/Modern_Robotics)，Lynch 与 Park 的教材，第二章到第六章讲构型空间、刚体运动、正逆运动学，推导比前者严格．
- [Underactuated Robotics](https://underactuated.mit.edu/) 讲动力学与控制，对操作任务的 VLA 不是必需，可以放到最后．

动手做一件事．在 [MuJoCo](https://mujoco.readthedocs.io/en/stable/overview.html) 里加载一个机械臂场景，用微分逆运动学写一段抓取并放置的脚本，观察关节角、末端位姿与夹爪状态随时间的变化．做完这件事，后面看数据集就不会陌生．

## 第二阶段，模仿学习与单任务策略

目标是在仿真里从零训练出能完成一个任务的策略，理解模仿学习的失效方式与常用补救，三到四周．

行为克隆的目标就是在示范数据上做监督学习，

$$
\min_\theta \; \mathbb{E}_{(\mathbf{o}, \mathbf{a}) \sim \mathcal{D}} \left[ -\log \pi_\theta(\mathbf{a} \mid \mathbf{o}) \right].
$$

它的两个经典问题是误差累积与多峰分布．误差累积指策略稍有偏差就会进入示范里没有的状态，之后越错越远；多峰分布指同一个观测下示范者可能采取几种不同的合理动作，用均方误差回归会得到它们的平均值，而平均值本身往往不合理．动作分块让策略一次输出未来 $H$ 步的动作 $\mathbf{a}_{t:t+H}$，减少决策次数，从而缓解误差累积；扩散或流匹配的动作头则能直接建模多峰分布．

要读的两篇论文．

- [ACT](https://www.roboticsproceedings.org/rss19/p016.pdf) 用条件变分自编码器加 Transformer 输出动作块，并用时间集成平滑相邻块的重叠部分．项目页 [ALOHA](https://tonyzhaozh.github.io/aloha/) 同时给出了一套低成本的双臂遥操作硬件．
- [Diffusion Policy](https://diffusion-policy.cs.columbia.edu/) 把动作块当作扩散模型的生成目标，以观测为条件去噪，用滚动时域的方式执行．注意它对视觉编码器、动作块长度与执行步数的消融实验，这些结论在 VLA 里仍然成立．

动手用 [LeRobot](https://github.com/huggingface/lerobot) 训练．它把数据集格式、策略实现与仿真评测统一到一个代码库里，官方文档有[仿真环境下的模仿学习教程](https://huggingface.co/docs/lerobot/il_sim)．建议顺序是，先在 PushT 上训练 Diffusion Policy，再在 ALOHA 仿真任务上训练 ACT，最后到 [LIBERO](https://libero-project.github.io/) 上跑一遍多任务评测．每次训练都记录成功率随示范数量的变化，这是理解数据效率最直接的方式．

这一阶段与三维背景的衔接点是以点云为输入的策略．[3D Diffusion Policy](https://www.roboticsproceedings.org/rss20/p067.pdf) 发表于 RSS 2024，用极简的点云编码器替换图像编码器，在少量示范下的泛化明显更好；[3D Diffuser Actor](https://3d-diffuser-actor.github.io/) 则在三维场景表示上做动作扩散．两者都值得读，因为三维表示怎样进入 VLA 目前还没有定论．

## 第三阶段，视觉语言动作模型

目标是读懂主要 VLA 的设计取舍，并在仿真基准上完成一次微调与评测，四到六周．

论文按下面的顺序读，每篇附上要重点看的地方．

- [RT-1](https://robotics-transformer1.github.io/)，看动作离散化的做法与数据集的构成，理解为什么多任务需要这么多数据．
- [RT-2](https://robotics-transformer2.github.io/)，看动作如何编码成文本词元，以及与图文数据一起微调时的比例，注意它的语义泛化实验．
- [Open X-Embodiment](https://robotics-transformer-x.github.io/)，看跨本体数据混合的正负效果，以及各数据集观测与动作空间的差异．
- [Octo](https://octo-models.github.io/)，看它怎样用可插拔的观测与动作头兼容不同机器人，以及扩散头与离散头的对比．
- [OpenVLA](https://openvla.github.io/)，看视觉编码器的选择、参数高效微调的效果，以及推理速度的瓶颈．
- [π0](https://arxiv.org/abs/2410.24164)，看流匹配动作专家的设计与训练目标．对动作块 $\mathbf{a}$ 与噪声 $\boldsymbol{\epsilon}$ 做线性插值 $\mathbf{a}^\tau = \tau \mathbf{a} + (1 - \tau)\boldsymbol{\epsilon}$，网络学习预测速度场 $\boldsymbol{\epsilon} - \mathbf{a}$，推理时从噪声出发积分几步即可．与扩散策略相比，它的采样步数更少，适合高频控制．
- [FAST](https://www.roboticsproceedings.org/rss21/p012.html)，看用离散余弦变换压缩动作块后再做自回归的思路，这是离散词元路线的改进．
- [OpenVLA-OFT](https://openvla-oft.github.io/)，看并行解码、动作分块与连续动作回归三项改动分别带来多少速度与成功率提升，这篇论文很适合建立微调 VLA 的默认配置．
- [π0.5](https://arxiv.org/abs/2504.16054)，看异构数据共同训练与子任务预测怎样带来开放环境的泛化．
- [SmolVLA](https://huggingface.co/blog/smolvla)，看小模型的取舍与异步推理，这是消费级显卡上最现实的微调对象．

两篇综述可以在读完上述论文之后用来查漏补缺，[A Survey on Vision-Language-Action Models for Embodied AI](https://arxiv.org/abs/2405.14093) 与 [Vision-Language-Action Models: Concepts, Progress, Applications and Challenges](https://arxiv.org/abs/2505.04769)．论文列表可以看 [Awesome-VLA](https://github.com/yueen-ma/Awesome-VLA)．

动手做三件事．

1. 跑通推理．用 LeRobot 加载 [SmolVLA 的预训练权重](https://huggingface.co/lerobot/smolvla_base)，或用 [openpi](https://github.com/Physical-Intelligence/openpi) 加载 π0，在仿真里观察它对语言指令的响应．
2. 微调一次．在 LIBERO 上微调 SmolVLA 或 π0，openpi 与 [OpenVLA 的代码库](https://github.com/openvla/openvla)都自带 LIBERO 的微调与评测脚本．对照 OpenVLA-OFT 报告的成功率，检查自己的流程有没有明显问题．
3. 评测泛化．用 [SimplerEnv](https://simpler-env.github.io/) 评测在 Open X-Embodiment 数据上预训练的策略，它把几个真机评测场景搬进了仿真，专门用来衡量真机策略．

评测时务必多次重复并报告方差，机器人策略的成功率波动很大，单次评测的结论不可靠．

## 第四阶段，真机

这一阶段可选，但如果研究方向要落到真机，越早开始越好．硬件推荐 [SO-101](https://huggingface.co/docs/lerobot/so101) 机械臂，它是 [SO-ARM100](https://github.com/TheRobotStudio/SO-ARM100) 的改进版，成本在几百美元量级，LeRobot 有完整的组装、标定、遥操作与训练教程．完整流程见官方的[真机模仿学习教程](https://huggingface.co/docs/lerobot/il_robots)，顺序是遥操作采集几十条示范，训练 ACT 或微调 SmolVLA，再部署回机械臂．Hugging Face 的[机器人课程](https://huggingface.co/learn/robotics-course/)把这条流程讲了一遍，配合文档看即可．

想做双臂或移动操作再看 [Mobile ALOHA](https://mobile-aloha.github.io/)，成本高一个量级．

## 仿真与工具清单

| 工具 | 用途 |
| --- | --- |
| [MuJoCo](https://mujoco.org/) | 物理引擎，多数操作基准的底层，第一阶段练习用它 |
| [LeRobot](https://huggingface.co/docs/lerobot/index) | 数据集格式、策略实现、真机接口一体化，全程主力 |
| [LIBERO](https://github.com/Lifelong-Robot-Learning/LIBERO) | 多任务操作基准，VLA 微调最常用的对照 |
| [SimplerEnv](https://simpler-env.github.io/) | 真机场景的仿真复刻，评测跨本体预训练策略 |
| [ManiSkill](https://maniskill.readthedocs.io/en/latest/) | GPU 并行仿真，任务多，适合大规模生成数据 |
| [robosuite](https://robosuite.ai/) 与 [RoboCasa](https://robocasa.ai/) | 前者是模块化仿真框架，后者在其上构建了大规模厨房场景 |
| [Isaac Lab](https://isaac-sim.github.io/IsaacLab/) | NVIDIA 的仿真框架，与 GR00T 配套，渲染质量高，安装较重 |
| [Genesis](https://genesis-world.readthedocs.io/) | 新的通用物理引擎，可微，值得关注但生态尚不成熟 |
| [数据集可视化](https://huggingface.co/spaces/lerobot/visualize_dataset) | 在浏览器里查看 LeRobot 格式数据集的每一条轨迹 |

安装建议是，第二阶段只装 LeRobot 与它依赖的仿真环境，第三阶段再加 LIBERO 与 openpi，其余按需安装．ROS 不是入门必需，做真机时再学．

## 从三维生成背景切入的方向

三维图形与 AIGC 3D 生成的经验在这个领域有几个直接的用武之地．

- **三维表示**进入策略．前面提到的 3D Diffusion Policy 与 3D Diffuser Actor 只是起点，怎样让 VLA 高效利用深度、点云或场景重建，仍是开放问题．
- **仿真资产**生成．RoboCasa 一类的大规模仿真场景依赖生成的物体与布局，三维生成模型可以直接产出可用于仿真的资产，缓解仿真数据的多样性瓶颈．
- **真实到仿真**的重建．把真实场景重建为可交互的仿真环境，用于评测与数据增广，这与几何处理的关系最近．
- **抓取位姿**生成．[GraspNet](https://graspnet.net/) 一类工作直接在点云上预测抓取位姿，是几何与操作结合的经典问题．
- **具身**基准．[BEHAVIOR](https://behavior.stanford.edu/) 这类家务级基准需要大量高质量的三维资产与场景，是三维生成研究者能直接贡献的地方．

## 常见误区

- 不要从强化学习开始．当前的 VLA 几乎全部基于模仿学习，强化学习是后期用于策略改进的手段，不是入门的门槛．
- 不要从 ROS 开始．ROS 解决的是真机系统集成问题，与学习算法无关，学它会消耗大量时间而不产生理解．
- 不要先买硬件．先在仿真里跑通完整流程，再决定要不要真机，以及需要什么规格的真机．
- 不要相信单次评测．同一策略在同一任务上的成功率可以相差二三十个百分点，报告结果时至少重复若干次．
- 不要指望仿真成功率直接迁移到真机．仿真用来验证流程与比较方法，真机表现要单独评测．
