# TestHub 本地部署实践笔记

> 本项目为个人学习笔记，记录基于开源 [TestHub](https://github.com/chenjigang4167/testhub_platform) 的 Windows 本地部署过程。  
> **未修改原项目源码**，仅完成环境搭建与功能验证。

## 原项目信息

| 项目 | 链接 |
|------|------|
| 开源仓库 | https://github.com/chenjigang4167/testhub_platform |
| 协议 | GPL v3 |
| 技术栈 | Django 4.2 + Vue 3 + MySQL 8.0 |

## 环境要求

| 组件 | 版本 | 是否必须 |
|------|------|----------|
| Python | 3.12（推荐） | 是 |
| Node.js | 18+ | 开发环境必须 |
| MySQL | 8.0+ | 是 |
| Java | 17+ | 可选（Allure 报告） |
| Redis | 6.0+ | 可选（APP 自动化） |

## 部署步骤（Windows）

### 1. 获取源码

```powershell
git clone https://github.com/chenjigang4167/testhub_platform.git
cd testhub_platform
```

或从 GitHub 下载 ZIP 并解压。

### 2. 创建 Python 虚拟环境

```powershell
python -m venv venv
venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

### 3. 配置环境变量

```powershell
copy .env.example .env
```

编辑 `.env`，至少配置：

- `SECRET_KEY`：随机字符串
- `DB_NAME` / `DB_USER` / `DB_PASSWORD` / `DB_PORT`：MySQL 连接信息

> 注意：`.env` 含敏感信息，**不要提交到 Git**。

### 4. 初始化 MySQL 数据库

```sql
CREATE DATABASE testhub CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

```powershell
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

### 5. 初始化扩展模块（按需）

```powershell
python manage.py init_locator_strategies
python manage.py load_component_pack
python manage.py makemigrations data_factory
python manage.py migrate data_factory
```

### 6. 启动后端

```powershell
python manage.py runserver
```

可选服务：

```powershell
python manage.py run_all_scheduled_tasks
celery -A backend worker -l info
```

### 7. 启动前端

```powershell
cd frontend
npm install
npm run dev
```

### 8. 访问地址

| 服务 | 地址 |
|------|------|
| 前端 | http://localhost:3000 |
| 后端 API | http://localhost:8000 |
| API 文档 | http://localhost:8000/api/docs/ |
| Admin 后台 | http://localhost:8000/admin/ |

## 部署过程中遇到的问题（请补充）

> 以下为模板，请根据你实际遇到的情况填写，面试时非常有用。

### 问题 1：（示例）pip 安装依赖失败

- **现象**：
- **原因**：
- **解决**：

### 问题 2：（示例）数据库连接失败

- **现象**：
- **原因**：
- **解决**：

### 问题 3：（示例）前端 npm install 报错

- **现象**：
- **原因**：
- **解决**：

## 功能验证清单

- [ ] 用户注册 / 登录
- [ ] 创建项目与测试用例
- [ ] 浏览 API 文档（Swagger）
- [ ] API 测试模块基本操作
- [ ] UI 自动化模块页面可访问
- [ ] （可选）AI 需求分析 / 用例生成

## 运行截图

> 建议在此目录下新建 `screenshots/` 文件夹，放入 2～3 张截图（登录页、用例管理、API 测试等）。

## 个人收获

- 熟悉了 Django + Vue3 前后端分离项目的本地联调流程
- 了解了测试管理平台的核心模块：用例管理、API 测试、UI 自动化
- 掌握了 MySQL 数据库迁移、`.env` 环境配置等部署基础操作

## 免责声明

本仓库仅为个人部署学习笔记，TestHub 平台版权归原开源作者所有。
