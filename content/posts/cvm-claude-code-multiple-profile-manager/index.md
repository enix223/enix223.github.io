---
title: '告别手动配置！CVM一键切换Claude Code 多套环境'
date: 2026-06-03T15:03:39+08:00
draft: false
---

# 1. 前言

随着LLM大模型越来越能打，日常开发使用**vibe coding**是势在必行的，而使用LLM进行开发，**claude code**是绕不过的神器。

claude code恰好支持使用第三方模型，同时各个AI大厂都陆续支持**coding plan**，各个厂商的模型能力各异，我希望可以在不同厂商之间方便的切换，但是claude code只允许设置一套模型参数，cc配置样例如下:

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-XXXXXX",
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_MODEL": "deepseek-v4-pro",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-v4-pro",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-pro",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-flash",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-v4-flash",
    "CLAUDE_CODE_EFFORT_LEVEL": "max"
  }
}
```

出于上述目的，我用claude code的vibe coding开发了一个快速切换cc配置的cli程序`cvm`。

# 2. 设计思路

claude code安装后，首次运行会在用户目录创建一个`.claude`文件夹，里面会存放claude code相关文件，其中`settings.json`可以让用户修改全局的cc的配置，同时工作目录也支持`.claude`文件夹，用户可以根据需要在不同的工作目录中设置特定的设置。

`cvm` cli程序就是通过修改用户目录或工作目录的`.claude`中的`settings.json`实现切换配置的目的。

# 3. 如何使用

cvm目前支持的命令包括:

```
cvm profile add <name>       添加新配置（交互式提示）
cvm profile update <name>    更新已有配置（字段选择器）
cvm profile list             列出所有配置（* 表示当前激活）
cvm profile activate <name>  切换到指定配置（全局）
cvm profile show <name>      查看配置详情
cvm profile current          打印当前激活的配置名称
cvm profile delete <name>    删除配置
cvm use <name>               将配置合并到 .claude/settings.local.json（本地）
```

## 3.1 安装

使用`cvm`需要先安装nodejs环境，nodejs安装教程请看官网: https://nodejs.org/zh-cn/download

安装好nodejs后，打开命令行，输入如下指令安装`cvm`

```
npm install -g @cloudesk/cvm
```

## 3.2 添加配置

添加配置命令： 

```
cvm profile add <name>
```

例如，添加deepseek的配置

```
cvm profile add deepseek
```

```shell
cvm profile add deepseek
? ANTHROPIC_AUTH_TOKEN — Anthropic API key
  e.g. sk-ant-api03-xxxx sk-XXXX （从模型后台中获取，并填写API key）
? ANTHROPIC_BASE_URL — Custom API endpoint
  e.g. https://api.anthropic.com https://api.deepseek.com/anthropic
? ANTHROPIC_MODEL — Default model
  e.g. claude-sonnet-4-6 deepseek-v4-pro
? ANTHROPIC_DEFAULT_OPUS_MODEL — Opus model override
  e.g. claude-opus-4-7 deepseek-v4-pro
? ANTHROPIC_DEFAULT_SONNET_MODEL — Sonnet model override
  e.g. claude-sonnet-4-6 deepseek-v4-pro
? ANTHROPIC_DEFAULT_HAIKU_MODEL — Haiku model override
  e.g. claude-haiku-4-5-20251001 deepseek-v4-flash
? CLAUDE_CODE_SUBAGENT_MODEL — Model for subagents
  e.g. claude-haiku-4-5-20251001 deepseek-v4-flash
? CLAUDE_CODE_EFFORT_LEVEL — Reasoning effort level
  e.g. max max
Profile "deepseek" saved to /Users/enix/.claude/settings-deepseek.json
```

主流厂商的claude code配置可参考附录。

## 3.2 列出可用配置

```
cvm profile list
```

输出类似如下结果:

```
 * deepseek
   glm
   xm

* = active
```

## 3.3 更换配置

我们可以使用如下命令激活全局配置

```
cvm profile activate <name>
```

例如，要激活刚才添加的`deepseek`配置，可以执行:

```
cvm profile activate deepseek
```

如果需要在当前工作目录激活某个特定的配置，可以使用如下命令

```
cvm use <name>
```

## 3.4 修改配置

我们需要修改某个配置的设置时，可以通过`profile update`命令实现:

```
cvm profile update <name>
```

运行该命令后，我们需要选择需要修改哪个配置项，并按空格选中（可多选），然后确定按提示修改。

```
cvm profile update glm                                                                                                                                                           ✔  10:13:20 
? Select fields to update: (Press <space> to select, <a> to toggle all, <i> to invert selection, and <enter> to proceed)
 ◉ ANTHROPIC_AUTH_TOKEN (current: fjdkalf)
❯◉ ANTHROPIC_BASE_URL (current: http://123.com)
 ◯ ANTHROPIC_MODEL (current: glm-5)
 ◯ ANTHROPIC_DEFAULT_OPUS_MODEL (current: glm-5)
 ◯ ANTHROPIC_DEFAULT_SONNET_MODEL (current: glm-5)
 ◯ ANTHROPIC_DEFAULT_HAIKU_MODEL (current: glm-5)
 ◯ CLAUDE_CODE_SUBAGENT_MODEL (current: glm-5)
```

## 3.5 删除配置

当我们需要删除配置时，可以使用如下命令删除：

```
cvm profile delete <name>
```

# 附录

* deepseek cc配置: https://api-docs.deepseek.com/zh-cn/quick_start/agent_integrations/claude_code
* 小米MIMO cc配置: https://platform.xiaomimimo.com/docs/zh-CN/integration/claudecode
* Minimax cc配置: https://platform.minimaxi.com/docs/token-plan/claude-code
* 智谱glm cc配置: https://docs.bigmodel.cn/cn/guide/develop/claude
* 千问 cc配置: https://www.alibabacloud.com/help/zh/model-studio/claude-code
* kimi cc配置: https://www.kimi.com/code/docs/third-party-tools/other-coding-agents.html#claude-code

# 结语

希望这个工具能帮助到您，cvm源代码可以查看我的github地址: http://github.com/enix223/cvm

❤️ Happy vibe coding 🤖