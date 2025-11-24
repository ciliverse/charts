# Ciliverse Charts Repository

这是Ciliverse的Helm Charts仓库，包含了用于Kubernetes部署的各种应用chart。

## 🚀 快速开始

### 添加仓库并安装Charts

```bash
# 添加Ciliverse Charts仓库
helm repo add ciliverse https://charts.cillian.website

# 更新Helm仓库
helm repo update

# 搜索可用的Charts
helm search repo ciliverse

# 安装cilikube
helm install cilikube ciliverse/cilikube --namespace cilikube --create-namespace

# 安装monitor-operator
helm install monitor-operator ciliverse/monitor-operator --namespace monitoring --create-namespace
```

## 📦 可用的Charts

- **cilikube** - Cilikube前后端应用部署
- **monitor-operator** - Kubernetes监控栈管理

## 🛠️ 开发者指南

### 自动化工具

本仓库提供了多种自动化工具来简化chart管理：

#### 1. 自动更新脚本

```bash
# 自动打包所有charts并更新仓库索引
./auto-update-charts.sh
```

#### 2. Makefile命令

```bash
# 查看所有可用命令
make help

# 自动更新所有内容
make update

# 只打包charts
make package

# 测试charts语法
make test

# 清理生成的文件
make clean

# 本地预览服务器
make serve
```

#### 3. Git Hooks自动化

安装Git hooks实现真正的自动化：

```bash
# 安装Git hooks
./install-git-hooks.sh
```

安装后，当你修改`Chart.yaml`文件并提交时，会自动：
- 打包所有charts
- 更新仓库索引
- 更新HTML页面
- 将更新的文件添加到提交中

### 添加新的Chart

1. **手动创建**：
   ```bash
   helm create ciliverse/your-chart-name
   ```

2. **使用Makefile**：
   ```bash
   make new-chart
   ```

3. **编辑Chart.yaml**：
   修改chart的基本信息，包括版本、描述等

4. **自动更新**：
   ```bash
   make update
   # 或者
   ./auto-update-charts.sh
   ```

### 目录结构

```
.
├── ciliverse/
│   ├── charts/                 # 生成的chart包和仓库文件
│   │   ├── *.tgz              # chart包文件
│   │   ├── index.yaml         # 仓库索引
│   │   └── index.html         # 仓库主页
│   ├── cilikube/              # cilikube chart源码
│   └── monitor-operator/      # monitor-operator chart源码
├── auto-update-charts.sh      # 自动更新脚本
├── update-charts.sh          # 完整更新脚本
├── install-git-hooks.sh      # Git hooks安装脚本
├── Makefile                  # Make命令
└── README.md                 # 本文件
```

## 🔧 工作流程

### 开发新Chart

1. 创建新的chart目录
2. 编辑Chart.yaml和templates
3. 运行`make test`测试语法
4. 运行`make update`更新仓库
5. 提交代码（如果安装了Git hooks会自动更新）

### 更新现有Chart

1. 修改chart源码
2. 更新Chart.yaml中的版本号
3. 运行`make update`或直接提交（如果有Git hooks）

### 部署到生产环境

```bash
# 方法1: 使用rsync
rsync -av ciliverse/charts/ user@server:/var/www/charts/

# 方法2: 使用git部署
git add ciliverse/charts/
git commit -m "Update charts"
git push

# 方法3: 自定义部署脚本
make deploy  # 需要先配置Makefile中的deploy命令
```

## 🌐 访问仓库

- **仓库地址**: https://charts.cillian.website
- **仓库主页**: https://charts.cillian.website/index.html

## 📝 注意事项

1. 每次修改chart后记得更新版本号
2. 使用`make test`确保chart语法正确
3. 提交前运行`make update`确保仓库是最新的
4. 生产环境部署后记得测试chart安装

## 🤝 贡献

欢迎提交PR来改进charts或添加新的charts！

## 📄 许可证

本项目采用MIT许可证。