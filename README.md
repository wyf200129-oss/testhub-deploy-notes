# TestHub 本地部署实践笔记

> 个人学习笔记：记录在 **Windows 10** 环境下，基于开源项目 [TestHub](https://github.com/chenjigang4167/testhub_platform) 完成本地部署与功能验证的全过程。  
> **说明**：未修改原项目源码，仅按官方部署文档完成环境搭建。

---

## 一、项目背景

| 项目 | 说明 |
|------|------|
| 原开源仓库 | https://github.com/chenjigang4167/testhub_platform |
| 开源协议 | GPL v3 |
| 技术栈 | Django 4.2 + DRF + Vue 3 + Element Plus + MySQL 8.0 |
| 本地路径 | `D:\PythonProject1\testhub_platform-main` |
| 实践目标 | 熟悉全栈测试管理平台的部署流程与核心模块 |

TestHub 是一个 AI 驱动的测试管理平台，主要模块包括：

- 测试用例管理、评审、执行与报告
- API 自动化测试（HTTP / WebSocket）
- UI 自动化测试（Selenium / Playwright）
- AI 需求分析与测试用例生成
- 数据工厂、项目管理等

---

## 二、环境准备

### 2.1 软件清单

| 软件 | 版本要求 | 是否必须 | 用途 |
|------|----------|----------|------|
| Python | 3.12（推荐） | 是 | 后端运行 |
| Node.js | 18+ | 是 | 前端开发服务 |
| MySQL | 8.0+ | 是 | 数据存储 |
| Git | 最新版 | 建议 | 拉取源码 |
| Java | 17+ | 可选 | Allure 报告生成 |
| Redis | 6.0+ | 可选 | APP 自动化相关 |

### 2.2 安装验证

```powershell
python --version
node --version
npm --version
mysql --version
```

### 2.3 获取源码

**方式一：Git 克隆**

```powershell
git clone https://github.com/chenjigang4167/testhub_platform.git
cd testhub_platform
```

**方式二：下载 ZIP**

从 GitHub 下载 ZIP 并解压（本项目采用此方式）。

---

## 三、后端部署

### 3.1 创建虚拟环境并安装依赖

```powershell
cd D:\PythonProject1\testhub_platform-main

python -m venv venv
venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

> 若 PowerShell 禁止运行脚本，先执行：  
> `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`

### 3.2 配置环境变量

```powershell
copy .env.example .env
```

编辑 `.env`，至少修改以下项：

```ini
SECRET_KEY=随机生成的长字符串
DEBUG=True
ALLOWED_HOSTS=*
CORS_ALLOWED_ORIGINS=http://localhost,http://127.0.0.1

DB_NAME=testhub
DB_USER=root
DB_PASSWORD=你的MySQL密码
DB_PORT=3306

LANGUAGE_CODE=zh-hans
TIME_ZONE=Asia/Shanghai
```

> **安全提醒**：`.env` 含数据库密码等敏感信息，切勿上传到 GitHub。

### 3.3 创建数据库

登录 MySQL 后执行：

```sql
CREATE DATABASE testhub CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 3.4 数据库迁移

```powershell
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

按提示设置管理员账号（用于登录 Admin 后台）。

### 3.5 初始化扩展数据

```powershell
python manage.py init_locator_strategies
python manage.py load_component_pack
python manage.py makemigrations data_factory
python manage.py migrate data_factory
```

### 3.6 启动后端服务

**终端 1 — Django 主服务：**

```powershell
python manage.py runserver
```

**终端 2 — 定时任务（可选）：**

```powershell
python manage.py run_all_scheduled_tasks
```

**终端 3 — Celery（可选，APP 自动化）：**

```powershell
celery -A backend worker -l info
```

---

## 四、前端部署

新开一个终端：

```powershell
cd D:\PythonProject1\testhub_platform-main\frontend
npm install
npm run dev
```

前端默认端口 **3000**，并通过 Vite 代理将 `/api/` 请求转发到后端 `8000` 端口。

---

## 五、访问地址

| 服务 | 地址 |
|------|------|
| 前端页面 | http://localhost:3000 |
| 后端 API | http://localhost:8000 |
| Swagger 文档 | http://localhost:8000/api/docs/ |
| ReDoc 文档 | http://localhost:8000/api/redoc/ |
| Admin 后台 | http://localhost:8000/admin/ |

---

## 六、功能验证清单

部署完成后，逐项验证：

- [x] 后端 `runserver` 正常启动，无数据库连接报错
- [x] 前端 `npm run dev` 正常启动
- [ ] 浏览器打开 http://localhost:3000 可访问登录页
- [ ] 用户注册 / 登录成功
- [ ] 创建项目与测试用例
- [ ] Swagger 文档可正常浏览
- [ ] API 测试模块页面可访问
- [ ] UI 自动化模块页面可访问
- [ ] （可选）AI 需求分析 / 用例生成

---

## 七、常见问题与排查

### 7.1 虚拟环境激活失败

**现象**：`venv\Scripts\Activate.ps1` 报权限错误  

**解决**：

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

### 7.2 pip 安装依赖慢或失败

**现象**：下载超时、编译报错  

**解决**：

```powershell
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 7.3 数据库连接失败

**现象**：`django.db.utils.OperationalError: Access denied`  

**排查**：

1. 确认 MySQL 服务已启动
2. 确认 `.env` 中 `DB_USER`、`DB_PASSWORD`、`DB_PORT` 正确
3. 确认已执行 `CREATE DATABASE testhub`

### 7.4 migrate 报错

**现象**：迁移文件缺失或表已存在  

**解决**：

```powershell
python manage.py showmigrations
python manage.py makemigrations
python manage.py migrate
```

### 7.5 前端 npm install 慢

**解决**：

```powershell
npm install --registry=https://registry.npmmirror.com
```

### 7.6 前端能开但接口 404 / 跨域

**排查**：

1. 确认后端已在 8000 端口运行
2. 确认 `.env` 中 `CORS_ALLOWED_ORIGINS` 包含前端地址
3. 前端开发模式走 Vite 代理，无需额外改 CORS（生产环境需单独配置）

---

## 八、目录结构速览

```
testhub_platform-main/
├── apps/                 # Django 业务模块
├── backend/              # Django 配置（settings.py）
├── frontend/             # Vue3 前端
├── docs/                 # 官方文档
├── manage.py             # Django 入口
├── requirements.txt      # Python 依赖
├── .env.example          # 环境变量模板
└── .env                  # 本地配置（勿上传 Git）
```

---

## 九、个人收获

通过本次部署实践，我熟悉了：

1. **前后端分离项目**的本地联调方式（Vue dev server + Django API）
2. **Django 项目**的环境变量、数据库迁移、超级用户创建流程
3. **测试管理平台**的核心模块划分：用例管理、API 测试、UI 自动化、AI 分析
4. **`.env` 配置管理**与敏感信息隔离的基本规范

---

## 十、运行截图

截图放在 [`screenshots/`](./screenshots/) 目录（部署成功后补充）。

| 截图 | 说明 |
|------|------|
| `screenshots/login.png` | 登录页 |
| `screenshots/dashboard.png` | 首页 / 项目列表 |
| `screenshots/api-testing.png` | API 测试模块 |

---

## 十一、免责声明

本仓库仅为个人部署学习笔记，不包含 TestHub 原项目源码。  
TestHub 平台版权归原开源作者 [chenjigang4167](https://github.com/chenjigang4167) 所有。

---

## 十二、相关链接

- 原项目：https://github.com/chenjigang4167/testhub_platform
- 本笔记仓库：https://github.com/wyf200129-oss/testhub-deploy-notes-
