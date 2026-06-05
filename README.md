# Supervised Job Application Skill

这是一个监督式求职投递 Skill，用于帮助 agent 在招聘平台上筛选岗位、读取 JD、匹配简历、生成打招呼话术，并在用户逐个确认后执行投递或沟通动作。

它不是无确认海投工具。

## 适用场景

- 按岗位、城市、薪资、经验范围筛选机会
- 读取岗位 JD，并提取职责、硬性要求、加分项和风险点
- 对照用户简历判断岗位匹配度
- 生成基于 JD 和简历内容的打招呼话术
- 在用户明确确认后，辅助点击投递、立即沟通或发送消息
- 记录投递结果、默认话术发送情况、验证阻塞和跳过原因

## 安全边界

这个 Skill 把投递、发送消息、上传简历、点击立即沟通等动作都视为外部副作用。Agent 必须在每个岗位操作前向用户说明公司、岗位、动作和话术，并获得明确确认。

它不应该用于：

- 未经确认的批量海投
- 绕过验证码、短信、二维码、人脸验证或平台风控
- 自动处理账号密码、验证码或敏感身份信息
- 伪造简历经历或夸大用户能力
- 向无关岗位大量发送低质量申请

## 初次使用时应询问

Agent 第一次使用时，需要先确认：

- 目标岗位关键词
- 目标城市
- 最低可接受薪资
- 期望经验范围
- 学历、工作方式等限制
- 不想投递的行业或公司
- 每日投递上限
- 使用哪份简历作为匹配依据
- 只做筛选，还是允许逐个确认后协助投递

## 验证机制

正式投递前，Agent 应主动建议做一次验证流程：找一个与当前工作经验不相关的岗位，只验证搜索、读取 JD、识别按钮状态和页面跳转，不执行真实投递。

如果用户坚持要用无关岗位做真实投递测试，Agent 必须先说明这会产生低质量投递记录，并且只有在用户明确指定公司、岗位和动作后才能继续。

## BOSS 直聘经验

本 Skill 记录了一次 BOSS 直聘自动化过程中遇到的问题和解决方案：

- Codex 内置浏览器可能空白或卡在安全验证
- 普通 Chrome 和 agent 控制的 Chrome 可能不是同一个 profile
- 应通过 `chrome://version` 核对 Profile Path
- agent-browser 可能卡在旧标签，需要用 Chrome DevTools Protocol 查找真实页面
- BOSS 薪资数字可能使用特殊字体，需要还原
- JS 假点击可能无效，必要时使用真实鼠标事件
- `立即沟通` 不是预览，可能会直接发送平台默认招呼

详细流程见 [`references/boss-zhipin.md`](references/boss-zhipin.md)。

## 文件结构

```text
supervised-job-application/
├─ SKILL.md
├─ README.md
├─ agents/
│  └─ openai.yaml
└─ references/
   └─ boss-zhipin.md
```

## 安装使用

将本仓库克隆到支持本地 skills 的 agent 环境中，然后让 agent 读取 `SKILL.md`。

对于 Codex，可将目录放到：

```text
C:\Users\<用户名>\.codex\skills\supervised-job-application
```

其他 agent，例如 Claude Code，也可以直接阅读 `SKILL.md` 和 `references/` 中的操作规范后复用。
