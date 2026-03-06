---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
workflowType: 'architecture'
lastStep: 8
status: 'complete'
completedAt: '2026-03-06'
inputDocuments:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/product-brief-qingqiu_genealogy-2026-03-06.md
  - _bmad-output/planning-artifacts/prd-validation-report.md
  - _bmad-output/brainstorming/brainstorming-session-2026-03-06-0946.md
workflowType: 'architecture'
project_name: 'qingqiu_genealogy'
user_name: 'BobCheung'
date: '2026-03-06'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

---

## Project Context Analysis

### Requirements Overview

**Functional Requirements（39 项，7 类）：**

| 类别 | 数量 | 架构含义 |
|------|------|----------|
| 用户与认证 | 4 | 需认证/授权模块、族谱多租户隔离、UUID 标识体系 |
| 成员管理 | 6 | CRUD 服务、富文本生平、照片存储、非精确日期建模 |
| 关系管理 | 5 | 图数据/关系建模、环路检测、冲突检测、关系网生成算法 |
| 族谱展示 | 6 | 树状图/世系表双视图、图可视化引擎、搜索定位、分享链接 |
| 审批流程 | 7 | 工作流引擎、站内信/通知、状态机 |
| 批量导入 | 6 | Excel 解析、校验、关系自动建立、导入事务 |
| 系统管理 | 5 | 操作日志、备份、导出、健康检查、容器化 |

**Non-Functional Requirements：**

- **性能**：首屏 < 3s、树图千级 < 2s、详情 < 1s、批量导入 50 人 < 30s、关系网千级 < 10s
- **安全**：HTTPS、敏感字段加密、RBAC、操作审计、数据归属可控
- **可扩展**：MVP 万级成员、百万级预留异步；单家族 < 50 并发
- **无障碍**：WCAG 2.1 AA，MVP 键盘导航、焦点管理、语义化

**Scale & Complexity：**

- **Primary domain**：Full-stack Web 应用（SPA + RESTful API）
- **Complexity level**：Medium（关系建模、审批流程、大规模图可视化）
- **Estimated architectural components**：认证、成员、关系、族谱展示、审批、导入、存储、通知、日志、部署

### Technical Constraints & Dependencies

| 约束/依赖 | 来源 | 架构影响 |
|-----------|------|----------|
| 前后端分离 | PRD Web App | Vue 3 + Vite / Django-Ninja 已预设 |
| 家族树可视化 | PRD | 需图引擎（AntV G6 等），支持千级节点、懒加载 |
| 富文本编辑 | PRD | TipTap/Quill，生平字段需富文本存储 |
| 文件存储 | PRD | django-storages，本地 + 对象存储（S3/OSS/MinIO） |
| 关系完整性 | PRD Domain | 环路检测、冲突检测需在关系层实现 |
| 日期多样性 | PRD | 精确/约略/朝代/不详，需灵活日期模型 |
| Docker 部署 | PRD | 容器化、健康检查接口 |
| API 版本 | PRD | `/api/v1/` 前缀 |
| 响应式断点 | PRD | 768/1024px，树图在移动端可简化 |

### Cross-Cutting Concerns Identified

1. **认证与授权**：贯穿成员、关系、审批、导入、系统管理
2. **关系图数据**：关系建立、关系网生成、树状图、世系表、导入关系建立
3. **富媒体存储**：成员照片、生平富文本、后续视频
4. **审批工作流**：生平修改 → 审批 → 通知 → 生效
5. **操作审计**：成员/关系/审批变更需统一日志
6. **数据导出与迁移**：备份、导出、家族数据归属

---

## Starter Template Evaluation

### Primary Technology Domain

**Full-stack Web 应用**（前后端分离）：基于 PRD 与项目上下文分析，主技术域为 Vue 3 SPA + Django RESTful API。

**项目现状：** 后端已有 Django 6.0.3 脚手架；前端尚未创建。

### Starter Options Considered

| 层级 | 选项 | 说明 |
|------|------|------|
| **前端** | create-vue | Vue 官方推荐，Vite 驱动，替代 Vue CLI |
| **前端** | Vue CLI | 已进入维护模式，不推荐新项目 |
| **后端** | 现有 Django + django-ninja | 项目已有 Django，采用集成方式 |
| **后端** | django-admin startproject | 仅适用于全新项目，本项目已存在 |

### Selected Starter: create-vue（前端）+ Django + django-ninja（后端）

**选择理由：**

- **前端**：create-vue 为 Vue 官方推荐，基于 Vite，与 PRD 中「Vue 3 + Vite」一致；支持 TypeScript、Vue Router、Pinia 等可选能力。
- **后端**：项目已有 Django 6.0.3，通过 `pip install django-ninja` 集成即可，无需重新脚手架。

### 前端初始化命令

```bash
# 在项目根目录下创建 frontend 子目录
npm create vue@latest frontend -- --typescript --router --pinia
```

**说明：** 使用 `--` 传递参数可跳过交互式提示。PowerShell 用户可使用：`npm create vue@latest frontend '--' --typescript --router --pinia`

### 后端集成命令

```bash
# 在项目虚拟环境中执行
pip install django-ninja
```

随后在 `qingqiu_genealogy/` 下创建 `api.py`，并在 `urls.py` 中挂载 `/api/v1/`（符合 PRD 的 API 版本约定）。

### Starter 提供的架构决策

**前端（create-vue）：**

| 维度 | 决策 |
|------|------|
| **语言与运行时** | TypeScript、Node 18.3+ |
| **构建工具** | Vite（ESM、HMR、快速冷启动） |
| **路由** | Vue Router |
| **状态管理** | Pinia |
| **UI 框架** | Element Plus（表单、表格、弹窗、树形等） |
| **代码组织** | `src/` 下 components、views、router、stores |
| **开发体验** | 热更新、Vite 插件生态 |

**后端（Django + django-ninja）：**

| 维度 | 决策 |
|------|------|
| **API 框架** | django-ninja（Pydantic 校验、OpenAPI 文档） |
| **API 版本** | `/api/v1/` 前缀 |
| **文档** | 内置 Swagger UI（`/api/docs`） |
| **校验** | Pydantic schemas |

**注意：** 使用上述命令完成项目初始化应作为首个实现故事。

---

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions（阻塞实现）：**

- 数据库：PostgreSQL（生产）/ SQLite（开发）
- 认证：JWT（django-ninja-jwt）
- 文件存储：django-storages（本地 + S3 兼容）
- 图可视化：AntV G6
- 富文本：TipTap
- UI 框架：Element Plus

**Important Decisions（塑造架构）：**

- 关系网生成：同步（千级），预留异步
- 审批：状态机 + 站内信
- 部署：Docker Compose

**Deferred Decisions（MVP 后）：**

- 社交登录、多通道通知、懒加载树图、异步关系网

### Data Architecture

| 决策 | 选择 | 版本/说明 | 理由 |
|------|------|-----------|------|
| 数据库 | PostgreSQL | 16+ | 生产环境；支持并发、JSON、全文；符合「数据家族自有」 |
| 开发数据库 | SQLite | 内置 | 本地开发零配置 |
| ORM | Django ORM | 内置 | 与 Django 深度集成 |
| 迁移 | Django Migrations | 内置 | 版本化 schema |
| 缓存 | 暂不引入 | — | MVP 单家族 < 50 并发，可后加 Redis |
| 日期模型 | 自定义字段/JSON | — | 支持精确、约略、朝代、不详（PRD FR10） |

### Authentication & Security

| 决策 | 选择 | 版本/说明 | 理由 |
|------|------|-----------|------|
| 认证方式 | JWT | django-ninja-jwt | SPA 前后端分离；无状态；与 django-ninja 兼容 |
| Token 存储 | 前端内存 + 可选 localStorage | — | 避免 XSS 时优先内存；需配合 httponly cookie 策略 |
| 授权 | RBAC | 自定义 | 管理员 / 成员；Phase 2 子管理员 |
| 传输 | HTTPS | 生产强制 | PRD 安全要求 |
| 敏感字段 | 按需加密 | — | 手机号等可加密存储 |
| 审计 | 操作日志表 | — | 成员/关系/审批变更记录 |

### API & Communication Patterns

| 决策 | 选择 | 说明 | 理由 |
|------|------|------|------|
| API 风格 | REST | django-ninja | PRD 已定 |
| 版本前缀 | /api/v1/ | 路由 | PRD 要求 |
| 文档 | OpenAPI/Swagger | django-ninja 内置 | /api/docs |
| 错误格式 | JSON + HTTP 状态码 | 统一 | 便于前端处理 |
| 限流 | MVP 暂不 | 后期可加 | 单家族规模小 |
| CORS | 配置允许前端域名 | Django 中间件 | 前后端分离 |

### Frontend Architecture

| 决策 | 选择 | 版本/说明 | 理由 |
|------|------|-----------|------|
| 框架 | Vue 3 + Vite | create-vue | PRD；Composition API |
| 状态 | Pinia | 来自 starter | 审批、用户、族谱状态 |
| 路由 | Vue Router | 来自 starter | 页面与权限 |
| 树图 | AntV G6 | @antv/g6 | PRD；树/图布局、千级节点 |
| 富文本 | TipTap | 推荐 | PRD；生平编辑 |
| UI 框架 | Element Plus | element-plus | 表单、表格、弹窗、树形、响应式；中文文档完善 |
| 样式 | Tailwind CSS（可选） | — | 与 Element Plus 配合，或仅用 Element 主题变量 |
| 请求 | axios / fetch | — | 与 JWT 配合 |

### Infrastructure & Deployment

| 决策 | 选择 | 说明 | 理由 |
|------|------|------|------|
| 容器化 | Docker + Docker Compose | 多服务 | PRD FR39 |
| 应用服务器 | Gunicorn | Django 生产 | 替代 runserver |
| 反向代理 | Nginx | 静态/媒体/代理 | 生产标配 |
| 健康检查 | /api/health/ | 自定义接口 | PRD FR38 |
| 环境配置 | .env + django-environ | 12-factor | 密钥与配置分离 |
| 文件存储 | 本地 + django-storages | S3/OSS/MinIO 兼容 | PRD；自部署可选 MinIO |

### Decision Impact Analysis

**Implementation Sequence：**

1. 前端：`npm create vue@latest frontend` 初始化，安装 `element-plus`
2. 后端：`pip install django-ninja django-ninja-jwt`，挂载 `/api/v1/`
3. 数据库：PostgreSQL 配置，Django 模型（成员、关系、用户）
4. 认证：JWT 登录/刷新，前端 axios 拦截器
5. 存储：django-storages 配置（先本地）
6. 图可视化：AntV G6 集成，树状图 POC
7. 富文本：TipTap 集成
8. Docker Compose：前后端 + DB + Nginx

**Cross-Component Dependencies：**

- 认证 → 成员 CRUD、关系、审批、导入
- 关系模型 → 关系网生成、树图、世系表、导入
- 存储 → 成员照片、富文本
- 审批状态机 → 站内信、成员生平更新

---

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

**已识别的关键冲突点：** 命名、结构、格式、通信、流程 5 类，避免多 AI Agent 实现时产生不一致。

### Naming Patterns

**数据库命名（Django 约定）：**

- 表名：`snake_case` 复数，如 `family_members`、`member_relations`
- 列名：`snake_case`，如 `birth_date`、`father_id`
- 外键：`<关联表>_id`，如 `father_id`、`mother_id`
- 索引：`idx_<表>_<列>`，如 `idx_member_relations_father_id`

**API 命名（REST + django-ninja）：**

- 路径：复数资源，如 `/api/v1/members`、`/api/v1/relations`
- 路径参数：`{id}` 或 `:id`，如 `/members/{id}`
- 查询参数：`snake_case`，如 `?page_size=20&page=1`
- 请求/响应 JSON：`snake_case`（与 Django/Pydantic 一致）

**前端命名（Vue/JS 约定）：**

- 组件：PascalCase，如 `MemberCard`、`FamilyTree`
- 文件：kebab-case 或 PascalCase，如 `member-card.vue`、`FamilyTree.vue`
- 函数/变量：camelCase，如 `getMemberById`、`memberList`
- Pinia store：camelCase，如 `useMemberStore`

### Structure Patterns

**项目组织：**

```
qingqiu_genealogy/
├── frontend/          # Vue 3 + Vite（create-vue 产出）
│   ├── src/
│   │   ├── components/
│   │   ├── views/
│   │   ├── stores/
│   │   ├── router/
│   │   └── api/
│   └── ...
├── qingqiu_genealogy/ # Django 项目配置
├── members/           # Django app：成员
├── relations/         # Django app：关系
├── genealogy/        # Django app：族谱展示、审批等
├── api/               # django-ninja 路由聚合（或各 app 内 router）
└── manage.py
```

**测试位置：** 后端 `*/tests/` 或 `test_*.py`；前端 `*.spec.ts` 或 `__tests__/`。

**共享工具：** 后端 `core/utils.py` 或各 app 内；前端 `src/utils/`。

### Format Patterns

**API 响应：**

- 成功：直接返回数据，无 `{data: ...}` 包装；列表可用 `{items: [], total: N}`
- 错误：`{detail: string}` 或 `{detail: string, code?: string}`，HTTP 状态码 4xx/5xx
- 分页：`?page=1&page_size=20`，响应 `{items: [], total: N, page: 1, page_size: 20}`

**数据格式：**

- JSON 字段：`snake_case`（前后端统一，前端可转换展示）
- 日期：ISO 8601 字符串，如 `2024-01-15`、`2024-01-15T10:00:00Z`
- 布尔：`true`/`false`
- 空值：`null`

### Communication Patterns

**状态管理（Pinia）：**

- 更新：不可变更新，避免直接修改 state
- Action：`fetchMembers`、`submitApproval` 等动词开头
- 模块化：按领域分 store，如 `useMemberStore`、`useAuthStore`

**事件命名：** 若使用自定义事件，采用 `kebab-case`，如 `member-selected`。

### Process Patterns

**错误处理：**

- 后端：django-ninja 异常处理，返回统一 JSON 错误格式
- 前端：axios 拦截器统一处理 401/403/5xx，展示用户友好提示
- 区分：用户可见错误 vs 开发日志（不向用户暴露堆栈）

**加载状态：**

- 命名：`isLoading`、`isSubmitting`
- 作用域：列表/详情用局部，全局导航用全局
- UI：骨架屏或 loading 组件，避免整页空白

### Enforcement Guidelines

**所有 AI Agent 必须：**

- 遵循上述命名约定，不得混用 snake_case 与 camelCase 于同一层级
- API 请求/响应使用 snake_case
- 错误响应使用统一格式
- 新增 Django app 时保持 `members`、`relations`、`genealogy` 等模块边界

**模式校验：** 通过 lint（ESLint、Ruff）、PR 审查、API 契约测试验证。

### Pattern Examples

**正确示例：**

- API：`GET /api/v1/members?page=1&page_size=20` → `{items: [...], total: 100}`
- 错误：`{"detail": "成员不存在"}` + 404
- 组件：`<MemberCard :member="member" />`

**反模式：**

- API 返回 camelCase 而 Django 默认 snake_case
- 组件名 `membercard`（应 PascalCase）
- 表名 `FamilyMember`（应 snake_case 复数）

---

## Project Structure & Boundaries

### Complete Project Directory Structure

```
qingqiu_genealogy/
├── README.md
├── .gitignore
├── .env.example
├── docker-compose.yml
├── Dockerfile.backend
├── Dockerfile.frontend
│
├── frontend/                          # Vue 3 + Vite（create-vue）
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   ├── index.html
│   └── src/
│       ├── main.ts
│       ├── App.vue
│       ├── api/                       # API 客户端
│       │   ├── client.ts              # axios 实例 + JWT 拦截
│       │   ├── auth.ts
│       │   ├── members.ts
│       │   ├── relations.ts
│       │   └── approvals.ts
│       ├── components/
│       │   ├── common/                # 通用 UI
│       │   │   ├── LoadingSpinner.vue
│       │   │   └── ErrorMessage.vue
│       │   ├── member/                # 成员相关
│       │   │   ├── MemberCard.vue
│       │   │   ├── MemberDetail.vue
│       │   │   └── MemberForm.vue
│       │   └── genealogy/             # 族谱展示
│       │       ├── FamilyTree.vue      # AntV G6 树状图
│       │       └── PedigreeTable.vue   # 世系表
│       ├── views/
│       │   ├── LoginView.vue
│       │   ├── TreeView.vue
│       │   ├── MemberDetailView.vue
│       │   └── ApprovalView.vue
│       ├── stores/
│       │   ├── auth.ts
│       │   ├── member.ts
│       │   └── approval.ts
│       ├── router/
│       │   └── index.ts
│       ├── utils/
│       └── types/
│
├── manage.py
├── requirements.txt
├── qingqiu_genealogy/                 # Django 项目配置
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── api.py                         # NinjaAPI 实例，挂载 /api/v1/
│   ├── wsgi.py
│   └── asgi.py
│
├── members/                           # Django app：成员管理
│   ├── __init__.py
│   ├── models.py                      # FamilyMember
│   ├── admin.py
│   ├── schemas.py                     # Pydantic
│   ├── api.py                         # router
│   ├── services.py
│   ├── tests/
│   └── migrations/
│
├── relations/                         # Django app：关系管理
│   ├── __init__.py
│   ├── models.py                      # MemberRelation
│   ├── admin.py
│   ├── schemas.py
│   ├── api.py
│   ├── services.py                    # 环路/冲突检测、关系网生成
│   ├── tests/
│   └── migrations/
│
├── genealogy/                         # Django app：族谱展示、审批、导入
│   ├── __init__.py
│   ├── models.py                      # ApprovalRequest, OperationLog
│   ├── admin.py
│   ├── schemas.py
│   ├── api.py                         # 树图数据、世系表、审批、导入
│   ├── services.py
│   ├── tests/
│   └── migrations/
│
├── core/                              # 共享：用户、认证、存储
│   ├── __init__.py
│   ├── models.py                      # User 扩展
│   ├── auth.py                        # JWT 认证
│   ├── storage.py                    # django-storages 配置
│   └── utils.py
│
├── static/
├── media/                             # 本地媒体（开发）
└── _bmad-output/
```

### Architectural Boundaries

**API 边界：**

- 外部：`/api/v1/*` 为唯一 API 入口
- 认证：`/api/v1/auth/token`、`/api/v1/auth/refresh`（django-ninja-jwt）
- 成员：`/api/v1/members`、`/api/v1/members/{id}`
- 关系：`/api/v1/relations`、`/api/v1/relations/graph`
- 族谱：`/api/v1/tree`、`/api/v1/pedigree`、`/api/v1/approvals`、`/api/v1/import`
- 健康检查：`/api/health/`

**组件边界：**

- 前端：views → stores → api；组件通过 props/emits 通信
- 后端：api（router）→ services → models；无跨 app 直接调用 model

**数据边界：**

- 成员、关系：各自 app 的 models
- 审批、操作日志：genealogy app
- 用户：Django auth + core

### Requirements to Structure Mapping

| FR 类别 | 后端位置 | 前端位置 |
|---------|----------|----------|
| 用户与认证 | core/auth.py, django-ninja-jwt | stores/auth.ts, api/auth.ts |
| 成员管理 | members/ | components/member/, views/MemberDetailView |
| 关系管理 | relations/ | api/relations.ts, FamilyTree.vue |
| 族谱展示 | genealogy/api.py (tree, pedigree) | FamilyTree.vue, PedigreeTable.vue |
| 审批流程 | genealogy/models, api | stores/approval.ts, ApprovalView.vue |
| 批量导入 | genealogy/api.py, services | （管理员功能） |
| 系统管理 | genealogy (logs), core | （管理员功能） |

### Integration Points

**内部通信：** 前端 axios → Nginx → Gunicorn → Django → django-ninja；JWT 在 Header 传递。

**外部集成：** 文件存储（S3/MinIO）、后续邮件/短信（Phase 2）。

**数据流：** 用户登录 → JWT → 请求带 Token → 成员/关系/树图 API → 响应 JSON → Pinia/store → 组件渲染。

---

## Architecture Validation Results

### Coherence Validation ✅

**决策兼容性：** Vue 3 + Vite、Element Plus、Django、django-ninja、PostgreSQL、JWT、AntV G6、TipTap、django-storages 相互兼容，无版本冲突。

**模式一致性：** 命名（snake_case 后端/API、camelCase 前端）、结构（frontend/ + Django apps）、格式（JSON snake_case）与所选技术栈一致。

**结构对齐：** 项目结构支持前后端分离、模块化 Django apps、API 版本前缀，边界清晰。

### Requirements Coverage Validation ✅

**功能需求覆盖：** 39 项 FR 均已映射到架构组件（members、relations、genealogy、core）。

**非功能需求覆盖：** 性能（树图千级、首屏）、安全（JWT、HTTPS、审计）、可扩展（PostgreSQL、异步预留）、无障碍（WCAG 目标）均有对应决策。

### Implementation Readiness Validation ✅

**决策完整性：** 关键决策均含版本/说明与理由。

**结构完整性：** 目录树具体到文件级，集成点与边界已定义。

**模式完整性：** 命名、结构、格式、通信、流程均有规则与示例。

### Gap Analysis Results

**已补充：** 前端 UI 框架推荐 Element Plus（表单、表格、弹窗、树形等组件齐全，适合族谱管理场景）；富文本 TipTap 具体版本可在实现时锁定。

**可选增强：** Phase 2 社交登录、多通道通知、懒加载树图等已在 PRD 中预留，架构可扩展。

### Architecture Completeness Checklist

- [x] 项目上下文分析完成
- [x] 规模与复杂度评估完成
- [x] 技术约束与依赖识别完成
- [x] 关键决策已记录并验证版本
- [x] 技术栈完整指定
- [x] 命名、结构、格式、通信、流程模式已定义
- [x] 完整目录结构已定义
- [x] 组件边界与集成点已映射
- [x] FR 与结构映射完成

### Architecture Readiness Assessment

**整体状态：** READY FOR IMPLEMENTATION

**置信度：** 高

**主要优势：** PRD 与架构对齐、技术栈成熟、模式明确、结构清晰。

**后续可增强：** E2E 测试策略、CI/CD 流水线。

### Implementation Handoff

**AI Agent 指南：**

- 严格遵循本文档中的架构决策
- 按实现模式在各组件中保持一致
- 遵守项目结构与边界
- 所有架构相关问题以本文档为准

**首个实现优先级：**

```bash
npm create vue@latest frontend -- --typescript --router --pinia
cd frontend && npm install element-plus
pip install django-ninja django-ninja-jwt
```
