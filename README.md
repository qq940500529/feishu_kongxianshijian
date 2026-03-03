# feishu_kongxianshijian

飞书机器人（Python）示例：
- 被 @ 时接收消息事件
- 刚收到消息先添加 `OneSecond` 表情回复
- 查询消息内被 @ 成员（自动排除机器人）的共同空闲时间
- 空闲条件：工作日 `08:30-17:00`，连续空闲不少于 `15` 分钟
- 结果通过 `post` 富文本中的 `md` 标签返回，时段以代码块表格展示
- 在“用户进入与机器人会话”“机器人进群”事件触发时发送查询指引

## 目录结构

- `main.py`：长连接事件入口
- `bot/config.py`：配置加载
- `bot/feishu_client.py`：飞书 API 封装
- `bot/scheduler.py`：共同空闲时间计算

## 环境准备

1. Python 3.10+
2. 安装依赖：

```bash
pip install -r requirements.txt
```

3. 配置环境变量（已支持从 `.env` 读取）：

```env
FEISHU_APP_ID=你的应用ID
FEISHU_APP_SECRET=你的应用密钥
# 可选：若你已知机器人 open_id，可配置以更稳定地排除机器人
FEISHU_BOT_OPEN_ID=ou_xxx
```

可先复制模板文件再填写：

```bash
cp .env.example .env
```

## 飞书后台配置

### 1) 开启能力

- 开启机器人能力并发布版本

### 1.1) 机器人名称建议

- 建议将机器人名称设置为：`共同空闲时间查询`
- 原因：项目默认按该名称在被 @ 成员中识别并排除机器人自身
- 若你使用其他名称，请同步修改 [main.py](main.py) 中的 `BOT_DISPLAY_NAME`

### 2) 订阅事件

- 订阅：`im.message.receive_v1`（接收消息 v2.0）
- 订阅：`im.chat.access_event.bot_p2p_chat_entered_v1`（用户进入与机器人会话）
- 订阅：`im.chat.member.bot.added_v1`（机器人进群）
- 推荐使用：长连接方式接收事件

### 3) 权限（通过 JSON 批量导入）

- 导入位置：应用配置 -> 开发配置 -> 权限管理 -> 批量导入/导出权限 -> 导入
- 导入以下 JSON：

```json
{
	"scopes": {
		"tenant": [
			"calendar:calendar",
			"calendar:calendar.free_busy:read",
			"calendar:calendar:readonly",
			"contact:user.employee_id:readonly",
			"im:chat:readonly",
			"im:message",
			"im:message.group_at_msg:readonly",
			"im:message.group_msg",
			"im:message.p2p_msg:readonly",
			"im:message.reactions:write_only",
			"im:message:recall",
			"im:message:send_as_bot"
		],
		"user": [
			"calendar:calendar",
			"calendar:calendar.free_busy:read",
			"calendar:calendar:readonly",
			"contact:user.employee_id:readonly",
			"im:message"
		]
	}
}
```

## 运行

```bash
python main.py
```

## 默认查询规则

- 查询窗口：从当前时间开始，向后 7 天
- 输出范围：7 天窗口内的所有工作日（周一到周五）
- 工作时间：`08:30-17:00`
- 最小连续空闲时长：15 分钟
- 输出条数：展示全部符合条件的时段

可在 `bot/config.py` 的 `Settings` 中调整。

## 说明

- 共同空闲时间按“参与成员忙碌时间并集的补集”计算。
