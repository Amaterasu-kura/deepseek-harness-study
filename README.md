# 从一个 LLM 调用，到一个能交付的 Agent

**以 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 源码为教材，把 agent 开发的全链路拆开讲一遍。**

写给「LLM / agent 开发零基础、但想真正看懂一个 agent 产品」的人。不要求你会写 TypeScript，也不要求手搓代码——目标是读完以后，拿到任何一个 agent 框架，都能在半小时内画出它的结构图。

<p>
  <a href="https://amaterasu-kura.github.io/deepseek-harness-study/dsh-agent-learning.html"><b>📖 在线阅读 HTML 版（推荐）</b></a>
  &nbsp;·&nbsp;
  <a href="dsh-agent-learning.md">📄 Markdown 版</a>
</p>

---

## 两个版本

| 版本 | 文件 | 说明 |
|---|---|---|
| **HTML（推荐）** | [`dsh-agent-learning.html`](dsh-agent-learning.html) · [在线阅读](https://amaterasu-kura.github.io/deepseek-harness-study/dsh-agent-learning.html) | 单文件，下载后双击打开。带目录导航、折叠卡片、示意图与条形图，无服务端依赖（仅字体与首图走网络） |
| Markdown | [`dsh-agent-learning.md`](dsh-agent-learning.md) | 纯文本，GitHub 可直接阅读，便于检索与二次编辑 |

## 这本手册里有什么

- **27 节正文**，按「地基 → 骨架 → 主链路 → 长任务 → 迁移」推进，不是按源码目录罗列。
- **42 个最小示例**：一个代码块只说明一个机制。比如 20 行伪代码的 agent 循环、一行拦截器实现「危险命令要问人」、压缩时的一次 `surfaceOp: replace`。
- **27 张「产品经理视角」折叠卡片**（HTML 版）：每一节都回答「谁的痛 / 怎么解 / 代价是什么 / 能迁移到你项目上的判断」，默认收起，点一下才展开。
- **真实数据而非估算**：53 万行源码的分布、220 行 YAML 组装出整个产品、压缩在默认配置下的实际触发点是窗口的 **69.3%**（不是直觉里的 80%）、一次会话日志的真实事件序列。

### 目录

<details>
<summary>展开全部 27 节</summary>

| # | 章节 |
|---|---|
| 0 | 先给结论：agent 开发到底在开发什么 |
| 1 | 地基：LLM 是一个无状态函数 |
| 2 | 最小 agent 循环：20 行伪代码 |
| 3 | Harness 到底要补哪十件事 |
| 4 | 教材概览：DSH 是个什么样的项目 |
| 5 | 地基框架 Cordis：五个概念撑起整个项目 |
| 6 | 装配：产品是被一份份 YAML 拼出来的 |
| 7 | 主链路总览：一次回车之后发生了什么 |
| 8 | 会话日志：整个系统的唯一真相 |
| 9 | 轮次与步骤：循环的状态机 |
| 10 | 系统提示词：给模型看的「岗位说明书」 |
| 11 | 模型调用层：一个「能力缝」的完整例子 |
| 12 | 工具：agent 的双手 |
| 13 | 结果回灌：上下文是这样被撑爆的 |
| 14 | 上下文管理：压缩不是「删除」，是「换书」 |
| 15 | 持久化与恢复：崩溃之后还能接着干 |
| 16 | 安全边界：三道护栏，和一条诚实的边界 |
| 17 | 委派：让 agent 学会「叫别人干」 |
| 18 | 扩展面：能力是怎么长出来的 |
| 19 | 前端：为什么 agent 产品都需要一个「壳」 |
| 20 | 迁移清单：任何 agent 项目都有的 12 个模块 |
| 21 | 怎么读这类代码：四条方法论 |
| 22 | 两周学习路线（不用写代码） |
| — | 术语表 / 十条工程纪律 / 写在最后 |

</details>

## 适合谁

- 想从「会调 API」走到「看得懂 agent 产品」的开发者。
- 正在选型 agent 框架、想知道「一个成熟实现到底要考虑什么」的工程师或产品经理。
- 需要给团队做一次 agent 架构入门分享的人（可以直接拿这份材料讲）。

**不适合**：想找一份「照着敲就能跑」的教程的人。文中的代码块是最小示意，不是可直接编译的完整实现；真实签名请对照仓库源码（每节都给了路径）。

## 关于数据与结论

- 分析对象：`deepseek-harness` **0.2.2-rc.1**（2026 年 11 月）。这个项目仍在 developer preview，接口会变——**请学它的设计判断，而不是记类名**。
- 文中数字均来自源码阅读、文档与实测（统计口径写在对应图表下）。
- 这是一份**非官方**学习材料，与 DeepSeek AI 无隶属关系。DeepSeek Harness 的版权归其作者所有，本项目仅做学习性解读。

## 致谢

- [**DeepSeek Harness**](https://github.com/deepseek-ai/deepseek-harness)：本手册的教材与分析对象，感谢 DeepSeek AI 开源。
- [**歸藏设计（op7418/dsh-guizang-design）**](https://github.com/op7418/dsh-guizang-design)：本页面的 HTML 排版、图示组件与折叠卡片样式，都来自这个 DSH 插件。没有它，这份文档只是一堆 Markdown。

## 反馈与勘误

发现问题、有不同理解、或者希望补充某个子系统——**欢迎提 [Issue](../../issues/new)**，我会尽量逐条回复与勘误。

如果这份材料帮你省下了几天摸索时间，欢迎点个 ⭐。

## License

[MIT](LICENSE) © 2026 Fan Xiuyuan
