简体中文 | [English](README.en.md)

# 螺壳 Luoke

**完全在你的 Mac 上运行的个人 AI。** 螺壳管着本机的模型，在它上面跑邮件助手、输入法这些应用；它们帮你做事，同时把你自己的东西整理成一份只在本机的记忆，让每个应用都更懂你。不经过任何云服务，没有账号，没有上传。

> Luoke is a personal AI that runs entirely on your Mac. One app manages the local models; mail, input method and other apps run on top of it and share a private on-device memory. No cloud service, no account, no upload.

螺壳由**螺壳实验室 Luoke Labs** 出品，现在处于封闭测试（beta1）。下载和介绍见 **[luoke.ai](https://luoke.ai)**。

## 有什么

| 名称 | 是什么 | 仓库 |
|---|---|---|
| **螺壳** | 用户装的那个应用：管本机所有模型推理（按内存分档挑模型）、各应用的安装和更新；螺壳记忆也在里面 | [luoke-core](https://github.com/luokelabs/luoke-core) |
| **螺壳记忆** | 各应用共用的本机记忆：记下发生了什么，归纳出对你的理解；每条都有依据，能看、能改、能删 | [luoke-mem](https://github.com/luokelabs/luoke-mem) |
| **螺壳邮件助手** | 用本机模型整理整个邮箱：分拣新邮件、清理旧邮件、订阅体检、精选、起草回复 | [luoke-mail](https://github.com/luokelabs/luoke-mail) |
| **螺壳输入法** | 小鹤双拼和全拼，本机模型按你前面写的内容挑首选、往下联想 | [luoke-ime](https://github.com/luokelabs/luoke-ime) |

在做、还没对外的：螺壳的主动提醒和对话（agent）、深度搜索（本机文件和照片）、体检（空闲时查这台 Mac 的垃圾和安全问题）。

这四个组件已经开源（10-28），代码、文档和许可见各自的仓库；在做的几个准备好以后照同样的方式公开。

## 底线

- **本机就是边界**：模型、记忆、你的邮件和打的字都只在这台 Mac 上。螺壳只在下载模型和应用、检查更新时联网；各应用什么时候联网写在各自的说明和[系统文档](https://luokelabs.github.io/luoke-atlas/)里。
- **记忆归你**：任何一条理解都能看到依据，能否定、能删；本机的应用都能读，删除只在记忆自己的界面里做。
- **能撤回的才自动做**：归档、标已读这类可以撤回的事，确认一次以后可以自动做并告诉你；删除这类撤回不了的，永远要你自己点。
- **照实说**：每个应用在不同内存、不同芯片上能做到什么，照评测结果写，做不到的就说做不到。

## 这个仓库

这里是螺壳的顶层文档：各组件怎么分工、怎么协作、共用哪些词，以及涉及多个组件的决定。各组件的代码在各自的仓库里，作为子模块挂在这里。

- [系统文档](https://luokelabs.github.io/luoke-atlas/)：各组件现在怎么工作、接口、任务和提示词、文件位置
- [顶层设计](ARCHITECTURE.md)：分工、数据怎么流、组件之间的约定
- [术语](CONTEXT.md)：跨组件共用的词，以及对外怎么称呼
- [决定记录](docs/adr/)：涉及不止一个组件的决定
- [组件注册页](projects/)：每个组件对外提供什么、依赖谁、对别的组件有什么需求
- [新增一个组件](docs/new-component.md)

## 下载和反馈

- 下载：[luoke.ai](https://luoke.ai)，或 [luokelabs/luoke-releases](https://github.com/luokelabs/luoke-releases/releases/latest) 里最新的测试版。要求 Apple 芯片的 Mac、macOS 26.2 以上、内存 16 GB 起。
- 封闭测试的反馈走测试者自己的渠道；安全问题见 [SECURITY.md](SECURITY.md)。

## 参与开发（团队成员）

```bash
git clone --recurse-submodules https://github.com/luokelabs/luoke.git
cd luoke
git submodule foreach 'git switch main && git pull'
```

子模块指向团队开发用的私有仓库（`<组件>-private`），只看代码的话直接克隆上面表里的公开仓库就行。第三行让每个组件停在 main 的最新提交上（不然子模块停在顶层记下的提交上，是 detached HEAD）。在哪个组件里改，就在那个目录里提交、推那个组件的仓库。顶层记下的各组件提交只在出版本（内部版 `internal-*`、测试版、正式版）时更新并打标签，用来还原"那一版是哪几份代码"。

---

© 2026 螺壳实验室 Luoke Labs。各组件的许可见各自仓库。
