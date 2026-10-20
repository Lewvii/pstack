# 安全

请通过 [GitHub 的私密漏洞报告表单](https://github.com/michael-denyer/pstack-claude/security/advisories/new)报告漏洞，本仓库已启用该功能。请不要为漏洞开公开 issue，因为 issue 追踪器在你按下提交的那一刻就是公开的。私密公告能让细节只留在你我之间，直到修复发布。

以下范围都在受理之列：技能树、hook、`plugins/pstack/skills/poteto-mode/scripts` 下的 vendored bun 脚本，以及 CI 工作流。任何能让 agent 运行用户并未打算运行的命令、外泄密钥、或越出它被指向的仓库的行为，都算在内；本仓库在固定和安装自身依赖方式上的供应链弱点也算。Claude Code 本身的发现属于 Anthropic，本仓库所同步的上游项目中的发现属于上游。

只有 `main` 上的最新发行版会收到修复。插件自动更新按版本号安装，所以修复以新版本发布，而不是给旧版本打补丁，也没有需要回移的维护分支。
