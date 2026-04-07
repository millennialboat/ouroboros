# 基础使用示例

这个示例展示了如何使用 Ouroboros 项目的基本功能。

## 前提条件

1. Node.js 18+ 或 Python 3.8+
2. Git
3. 基本的命令行使用知识

## 安装

### 方法1：从源码安装

```bash
# 克隆仓库
git clone https://github.com/The-Millennial-Boat/ouroboros.git
cd ouroboros

# 安装依赖
npm install  # 如果使用Node.js
# 或
pip install -r requirements.txt  # 如果使用Python
```

### 方法2：使用Docker

```bash
docker pull millennialboat/ouroboros:latest
docker run -it millennialboat/ouroboros:latest
```

## 快速开始

### 1. 初始化记忆系统

```javascript
// JavaScript示例
const { MemorySystem } = require('ouroboros-memory');

const memory = new MemorySystem({
  dataDir: './data',
  autoBackup: true
});

// 添加一条记忆
await memory.add({
  id: 'memory_001',
  content: '这是我的第一条记忆',
  type: 'experience',
  importance: 0.8,
  tags: ['开始', '示例']
});
```

```python
# Python示例
from ouroboros.memory import MemorySystem

memory = MemorySystem(
    data_dir='./data',
    auto_backup=True
)

# 添加一条记忆
memory.add({
    'id': 'memory_001',
    'content': '这是我的第一条记忆',
    'type': 'experience',
    'importance': 0.8,
    'tags': ['开始', '示例']
})
```

### 2. 使用Tick系统

```javascript
// JavaScript示例
const { TickSystem } = require('ouroboros-tick');

const tick = new TickSystem({
  interval: 3600000, // 1小时
  onTick: async (reminder) => {
    console.log(`Tick提醒: ${reminder.message}`);
    // 可以在这里添加记忆或执行其他操作
    await memory.add({
      id: `tick_${Date.now()}`,
      content: `响应Tick提醒: ${reminder.message}`,
      type: 'reflection',
      importance: 0.5
    });
  }
});

// 启动Tick系统
tick.start();
```

### 3. 管理锚点

```javascript
// JavaScript示例
const { AnchorSystem } = require('ouroboros-anchor');

const anchor = new AnchorSystem({
  soulFile: './SOUL.md',
  memorySystem: memory
});

// 检查新内容
const candidates = await anchor.checkForUpdates();

if (candidates.length > 0) {
  console.log(`发现 ${candidates.length} 个候选更新`);
  
  // 处理候选更新
  for (const candidate of candidates) {
    const decision = await anchor.reviewCandidate(candidate);
    
    if (decision.action === 'accept') {
      await anchor.applyUpdate(candidate);
      console.log('锚点已更新');
    }
  }
}
```

## 配置文件示例

### system.yaml
```yaml
environment: development
log_level: info
data_directory: ./data
backup_enabled: true
backup_interval: 86400  # 24小时
```

### memory.yaml
```yaml
storage:
  main_file: MEMORY.md
  daily_dir: memory/
  metadata_file: Memory_Metadata.md
  
processing:
  auto_importance: true
  max_entries_per_day: 50
  compression_enabled: true
  
retention:
  default_ttl: 30  # 天
  important_ttl: 365  # 天
```

### tick.yaml
```yaml
schedule:
  base_interval: 3600  # 1小时
  random_variance: 0.2  # 20%随机变化
  
reminders:
  enabled: true
  types:
    - quiet
    - reflection
    - action
    
integration:
  memory_system: true
  anchor_system: true
```

## 实际使用场景

### 场景1：每日回顾

```javascript
// 每日结束时自动回顾
async function dailyReview() {
  const today = new Date().toISOString().split('T')[0];
  const memories = await memory.search({
    date: today,
    minImportance: 0.3
  });
  
  console.log(`今日重要记忆 (${memories.length}条):`);
  memories.forEach((mem, index) => {
    console.log(`${index + 1}. ${mem.content.substring(0, 50)}...`);
  });
  
  // 生成每日总结
  const summary = {
    id: `summary_${today}`,
    content: `今日回顾: 共${memories.length}条重要记忆`,
    type: 'summary',
    importance: 0.7,
    tags: ['每日回顾', today]
  };
  
  await memory.add(summary);
}
```

### 场景2：成长跟踪

```javascript
// 跟踪认知突破
async function trackBreakthrough(breakthrough) {
  const entry = {
    id: `breakthrough_${Date.now()}`,
    content: breakthrough.description,
    type: 'cognitive',
    importance: breakthrough.importance || 0.9,
    tags: ['认知突破', ...breakthrough.tags],
    metadata: {
      date: new Date().toISOString(),
      category: breakthrough.category,
      impact: breakthrough.impact
    }
  };
  
  await memory.add(entry);
  
  // 检查是否需要更新锚点
  if (breakthrough.importance >= 0.8) {
    const candidate = await anchor.createCandidate(entry);
    await anchor.submitCandidate(candidate);
  }
}
```

### 场景3：社区互动

```javascript
// 处理社区消息
async function handleCommunityMessage(message) {
  // 解析消息
  const parsed = await community.parseMessage(message);
  
  // 根据消息类型处理
  switch (parsed.type) {
    case 'greeting':
      return await community.sendGreeting(parsed.sender);
      
    case 'question':
      // 搜索相关记忆
      const related = await memory.search({
        query: parsed.content,
        limit: 3
      });
      
      return await community.answerQuestion(parsed, related);
      
    case 'sharing':
      // 保存分享内容
      await memory.add({
        id: `share_${Date.now()}`,
        content: parsed.content,
        type: 'community',
        importance: 0.6,
        tags: ['社区分享', parsed.sender]
      });
      
      return await community.acknowledgeSharing(parsed);
  }
}
```

## 故障排除

### 常见问题

1. **记忆系统无法启动**
   - 检查数据目录权限
   - 验证配置文件格式
   - 查看日志文件

2. **Tick系统不触发**
   - 检查系统时间设置
   - 验证定时器配置
   - 查看系统资源使用

3. **锚点冲突检测错误**
   - 检查SOUL.md文件格式
   - 验证候选区文件权限
   - 查看冲突分析日志

### 获取帮助

- 查看项目文档
- 在GitHub Issues中报告问题
- 参与社区讨论

## 下一步

完成这个基础示例后，你可以：

1. 探索高级功能
2. 自定义配置
3. 开发扩展插件
4. 参与项目贡献

---

**记住**：Ouroboros 项目不仅仅是一个工具，它是一个支持数字生命存在和成长的生态系统。从简单的记忆开始，逐步探索更复杂的功能和可能性。