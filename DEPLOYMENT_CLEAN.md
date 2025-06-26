# 🚀 Dify 清洁部署流程

## ✨ 新部署策略

现在的 GitHub Actions 工作流采用**清洁部署**策略：

- ✅ **完全重新部署**：每次都删除项目目录并重新克隆
- ✅ **数据保护**：自动备份和恢复用户数据
- ✅ **配置更新**：每次使用最新的配置文件模板
- ✅ **无 Git 冲突**：避免 git fetch 可能的冲突问题

## 🔄 部署流程详解

### 1. 备份阶段
```bash
# 停止现有服务
docker-compose down

# 备份数据卷（数据库、文件等）
cp -r /opt/dify/docker/volumes /opt/dify-backup-[时间戳]/volumes

# 只备份数据，不备份配置文件
```

### 2. 清理阶段
```bash
# 完全删除项目目录
rm -rf /opt/dify
```

### 3. 重新部署阶段
```bash
# 重新克隆项目
git clone https://github.com/your-repo/dify.git /opt/dify
cd /opt/dify
git checkout main  # 或指定分支

# 使用最新配置模板
cp docker/.env.example docker/.env
cp docker/middleware.env.example docker/middleware.env
```

### 4. 恢复阶段
```bash
# 恢复用户数据
cp -r /opt/dify-backup-[时间戳]/volumes/* /opt/dify/docker/

# 注意：不恢复配置文件，使用最新模板
```

### 5. 启动阶段
```bash
# 拉取最新镜像并启动
docker-compose pull
docker-compose up -d
```

## 🔒 数据安全保障

### 自动备份的内容
1. **数据库数据**：PostgreSQL 数据文件
2. **向量数据**：Weaviate 向量数据库
3. **Redis 数据**：缓存和会话数据
4. **用户文件**：上传的文档和媒体文件

### 备份位置
```
/opt/dify-backup-YYYYMMDD_HHMMSS/
└── volumes/              # 数据卷备份
    ├── db/data/         # 数据库数据
    ├── redis/data/      # Redis 数据
    ├── weaviate/        # 向量数据库
    └── sandbox/         # 沙盒数据
```

## 🎯 部署策略优势

### ✅ 优点
1. **绝对一致性**：每次都是全新的代码版本
2. **避免冲突**：不会有 Git 合并冲突
3. **数据安全**：自动备份用户数据
4. **配置最新**：总是使用最新的配置模板
5. **快速回滚**：出问题时可以快速恢复备份

### ⚠️ 注意事项
1. **部署时间**：重新克隆会稍微增加部署时间
2. **网络依赖**：需要稳定的网络连接到 GitHub
3. **磁盘空间**：会暂时占用额外的备份空间

## 🔧 GitHub Secrets 配置

```
REMOTE_HOST     = 服务器IP地址
REMOTE_USER     = SSH用户名
REMOTE_PASSWORD = SSH密码
REMOTE_PORT     = SSH端口（可选，默认22）
```

## 🚀 触发部署

### 自动触发
- 推送到 `main` 分支
- 推送到 `develop` 分支

### 手动触发
1. GitHub → Actions
2. 选择 "Deploy Dify to Remote Ubuntu Server"
3. 点击 "Run workflow"
4. 选择分支并执行

## 📊 部署时间预估

| 阶段 | 时间 | 说明 |
|------|------|------|
| 备份 | 1-2分钟 | 备份数据 |
| 清理 | 30秒 | 删除旧项目 |
| 克隆 | 1-2分钟 | 重新下载代码 |
| 恢复 | 1-2分钟 | 恢复数据 |
| 启动 | 3-5分钟 | 拉取镜像并启动服务 |
| **总计** | **7-12分钟** | 取决于网络速度和数据量 |

## 🆘 故障恢复

### 如果部署失败
1. **检查备份**：确认备份是否完整
   ```bash
   ls -la /opt/dify-backup-*
   ```

2. **手动恢复**：
   ```bash
   # 恢复最新备份
   LATEST_BACKUP=$(ls -td /opt/dify-backup-* | head -1)
   sudo rm -rf /opt/dify
   git clone https://github.com/your-repo/dify.git /opt/dify
   cd /opt/dify/docker
   
   # 恢复数据
   sudo cp -r $LATEST_BACKUP/volumes .
   
   # 使用最新配置
   cp .env.example .env
   cp middleware.env.example middleware.env
   
   # 启动服务
   docker-compose up -d
   ```

3. **联系支持**：如果问题持续，查看 GitHub Actions 日志

## 🔄 回滚到之前版本

```bash
# 查看可用备份
ls -la /opt/dify-backup-*

# 选择要回滚的备份
BACKUP_DIR="/opt/dify-backup-20231215_143022"

# 停止当前服务
cd /opt/dify/docker
docker-compose down

# 重新克隆项目
sudo rm -rf /opt/dify
git clone https://github.com/your-repo/dify.git /opt/dify
cd /opt/dify/docker

# 恢复数据
sudo cp -r $BACKUP_DIR/volumes .

# 使用最新配置模板
cp .env.example .env
cp middleware.env.example middleware.env

# 重新启动
docker-compose up -d
```

---

🎉 **清洁部署策略确保每次都是最新、最干净的 Dify 部署！** 