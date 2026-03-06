---
stepsCompleted: [step-01-validate-prerequisites, step-02-design-epics, step-03-create-stories, step-04-final-validation]
inputDocuments:
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/architecture.md
---

# qingqiu_genealogy - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for qingqiu_genealogy, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

FR1: 用户可以使用手机号或邮箱注册并登录系统
FR2: 管理员可以创建族谱并成为该族谱的主管理员
FR3: 系统可以为每个家族成员分配唯一账号标识（UUID）
FR4: 已登录用户可以登出系统
FR5: 管理员可以添加家族成员，录入姓名、性别、出生日期、逝世日期、籍贯、生平、照片
FR6: 管理员可以修改任意家族成员的信息
FR7: 管理员可以删除无关联关系的家族成员
FR8: 家族成员可以查看自己的成员详情
FR9: 家族成员可以提交对自己生平信息的修改申请
FR10: 系统可以支持不精确的出生/逝世日期（如「约 1900 年」「清末」「不详」）
FR11: 管理员可以建立成员之间的父子、母子、配偶关系
FR12: 系统可以在建立或修改关系时检测并拒绝循环引用（如 A→B→C→A）
FR13: 系统可以在建立或修改关系时检测并拒绝关系冲突（如 A 既是 B 的父亲又是 B 的儿子）
FR14: 系统可以根据已录入的关系自动生成家族关系网
FR15: 管理员可以在树状图上通过点选父/母/配偶来建立关系
FR16: 已登录用户可以以树状图形式浏览家族关系
FR17: 已登录用户可以以世系表形式浏览家族关系
FR18: 已登录用户可以在树状图或世系表中搜索并定位成员
FR19: 已登录用户可以点击成员查看其详情（照片、生平、关系、时间线等）
FR20: 已登录用户可以分享成员或族谱的查看链接给其他族人
FR21: 系统支持树状图的缩放与拖拽操作
FR22: 家族成员提交生平修改后，系统可以自动发起审批流程
FR23: 审批人（管理员/族长）可以查看待审批的生平修改申请
FR24: 审批人可以通过或驳回生平修改申请
FR25: 审批人驳回时可以提供驳回意见
FR26: 申请人可以在申请被驳回后修改内容并重新提交
FR27: 审批通过后，系统可以将修改后的生平生效并通知申请人
FR28: 申请人可以通过站内信接收审批结果通知
FR29: 管理员可以下载 Excel 导入模板
FR30: 管理员可以上传 Excel 文件进行批量成员导入
FR31: 系统可以在导入前对数据进行校验并展示错误报告
FR32: 系统可以根据 Excel 中的「父亲姓名」「母亲姓名」自动建立父子、母子关系
FR33: 管理员可以在确认预览后执行导入
FR34: 导入完成后，系统可以触发关系网更新
FR35: 管理员可以查看操作日志（成员增删改、关系变更、审批等）
FR36: 管理员可以执行数据备份
FR37: 管理员可以导出族谱数据（支持迁移）
FR38: 系统提供健康检查接口，用于检测核心依赖组件的连通性
FR39: 系统支持容器化一键部署

### NonFunctional Requirements

NFR1: 首屏加载 < 3s（常规网络条件下）
NFR2: 树图初始渲染 < 2s（千级节点规模）
NFR3: 成员详情页 < 1s（含照片加载）
NFR4: 批量导入 50 人 < 30s（含校验与关系建立）
NFR5: 关系网生成千级 < 10s（小规模同步模式）
NFR6: 生产环境使用 HTTPS 传输加密
NFR7: 敏感字段按需加密存储
NFR8: 按角色（管理员/成员）控制数据访问
NFR9: 关键操作记录操作人、时间、内容
NFR10: 数据存储在家族自有环境，支持备份与导出
NFR11: 成员规模支持万级，百万级预留
NFR12: 关系网更新：同步（千级），预留异步队列与增量更新
NFR13: 单家族 < 50 人同时在线
NFR14: 无障碍目标 WCAG 2.1 AA；MVP 阶段：键盘导航、焦点管理、基础语义化

### Additional Requirements

- **Starter Template（Epic 1 Story 1）**：前端使用 create-vue 初始化（Vue 3 + Vite + TypeScript + Router + Pinia），后端集成 django-ninja 并挂载 /api/v1/
- **前端初始化命令**：`npm create vue@latest frontend -- --typescript --router --pinia`，安装 element-plus
- **后端集成**：`pip install django-ninja django-ninja-jwt`，在 qingqiu_genealogy/ 下创建 api.py 并挂载 /api/v1/
- **数据库**：PostgreSQL（生产）/ SQLite（开发），Django Migrations 版本化 schema
- **认证**：JWT（django-ninja-jwt），RBAC（管理员/成员）
- **文件存储**：django-storages，支持本地存储与对象存储（S3/OSS/MinIO）
- **图可视化**：AntV G6，支持千级节点、树状图布局
- **富文本**：TipTap，用于生平编辑
- **API 版本**：/api/v1/ 前缀，OpenAPI/Swagger 文档（/api/docs）
- **健康检查**：/api/health/ 接口
- **部署**：Docker + Docker Compose，Gunicorn + Nginx
- **项目结构**：frontend/、members/、relations/、genealogy/、core/ 等 Django apps
- **命名约定**：后端/API 使用 snake_case，前端使用 camelCase（组件 PascalCase）
- **API 响应格式**：成功直接返回数据；错误 {detail: string}；分页 {items: [], total: N}
- **响应式断点**：768px、1024px，树图在移动端可简化
- **日期模型**：支持精确、约略、朝代、不详，需灵活日期字段/JSON 建模

### FR Coverage Map

FR1: Epic 1 - 用户手机/邮箱注册登录
FR2: Epic 1 - 管理员创建族谱
FR3: Epic 1 - 成员 UUID 标识
FR4: Epic 1 - 用户登出
FR5: Epic 2 - 管理员添加成员
FR6: Epic 2 - 管理员修改成员
FR7: Epic 2 - 管理员删除成员
FR8: Epic 2 - 成员查看自己详情
FR9: Epic 2 - 成员提交生平修改申请
FR10: Epic 2 - 非精确日期支持
FR11: Epic 3 - 建立父子/母子/配偶关系
FR12: Epic 3 - 循环引用检测
FR13: Epic 3 - 关系冲突检测
FR14: Epic 3 - 自动生成关系网
FR15: Epic 3 - 树状图点选建立关系
FR16: Epic 4 - 树状图浏览
FR17: Epic 4 - 世系表浏览
FR18: Epic 4 - 搜索定位成员
FR19: Epic 4 - 查看成员详情
FR20: Epic 4 - 分享链接
FR21: Epic 4 - 树状图缩放拖拽
FR22: Epic 5 - 生平修改自动发起审批
FR23: Epic 5 - 审批人查看待审批
FR24: Epic 5 - 审批通过/驳回
FR25: Epic 5 - 驳回意见
FR26: Epic 5 - 驳回后重新提交
FR27: Epic 5 - 审批通过生效并通知
FR28: Epic 5 - 站内信通知
FR29: Epic 6 - 下载 Excel 模板
FR30: Epic 6 - 上传 Excel 导入
FR31: Epic 6 - 导入校验与错误报告
FR32: Epic 6 - 自动建立父子/母子关系
FR33: Epic 6 - 确认预览后执行导入
FR34: Epic 6 - 导入后触发关系网更新
FR35: Epic 7 - 操作日志
FR36: Epic 7 - 数据备份
FR37: Epic 7 - 导出族谱数据
FR38: Epic 7 - 健康检查接口
FR39: Epic 7 - 容器化部署

## Epic List

### Epic 1: 项目基础与用户认证

用户能够注册、登录、登出；管理员能够创建族谱并成为主管理员；项目具备前后端基础架构。
**FRs covered:** FR1, FR2, FR3, FR4 + Starter Template

### Epic 2: 成员管理

管理员能够添加、编辑、删除家族成员；成员能够查看自己的详情；成员能够提交生平修改申请；系统支持不精确日期。
**FRs covered:** FR5, FR6, FR7, FR8, FR9, FR10

### Epic 3: 关系管理

管理员能够建立父子、母子、配偶关系；系统检测并拒绝循环引用与关系冲突；自动生成关系网；支持在树状图上点选建立关系。
**FRs covered:** FR11, FR12, FR13, FR14, FR15

### Epic 4: 族谱展示

用户能够以树状图、世系表浏览家族；搜索并定位成员；查看成员详情（照片、生平、关系、时间线）；分享链接；树状图支持缩放与拖拽。
**FRs covered:** FR16, FR17, FR18, FR19, FR20, FR21

### Epic 5: 生平修改审批

成员提交生平修改后自动发起审批；审批人可查看、通过或驳回；驳回时提供意见；申请人可修改后重新提交；通过后生效并站内信通知。
**FRs covered:** FR22, FR23, FR24, FR25, FR26, FR27, FR28

### Epic 6: 批量导入

管理员能够下载 Excel 模板、上传文件、校验并展示错误报告；系统根据父亲/母亲姓名自动建立关系；确认预览后执行导入并触发关系网更新。
**FRs covered:** FR29, FR30, FR31, FR32, FR33, FR34

### Epic 7: 系统管理与部署

管理员能够查看操作日志、执行备份、导出族谱数据；系统提供健康检查接口；支持容器化一键部署。
**FRs covered:** FR35, FR36, FR37, FR38, FR39

---

## Epic 1: 项目基础与用户认证

用户能够注册、登录、登出；管理员能够创建族谱并成为主管理员；项目具备前后端基础架构。

### Story 1.1: 项目骨架初始化

As a 开发者,
I want 使用 create-vue 和 django-ninja 搭建前后端基础架构,
So that 项目具备开发族谱应用所需的技术栈与目录结构。

**Acceptance Criteria:**

**Given** 项目根目录已存在
**When** 执行 `npm create vue@latest frontend -- --typescript --router --pinia` 并在 frontend 中安装 element-plus
**Then** 前端具备 Vue 3 + Vite + TypeScript + Router + Pinia，且 Element Plus 可用
**And** 执行 `pip install django-ninja django-ninja-jwt` 并在 qingqiu_genealogy/ 下创建 api.py 挂载至 /api/v1/
**And** 项目结构符合架构文档（frontend/、qingqiu_genealogy/、api.py）
**And** 访问 /api/docs 可查看 OpenAPI 文档

### Story 1.2: 用户注册

As a 新用户,
I want 使用手机号或邮箱注册账号,
So that 能够登录系统并参与族谱。

**Acceptance Criteria:**

**Given** 用户访问注册页面
**When** 用户输入有效的手机号或邮箱、密码并提交
**Then** 系统创建用户账号并返回成功
**And** 手机号/邮箱格式校验正确，重复注册时返回明确错误
**And** 密码满足安全要求（长度、复杂度）
**And** API 路径为 POST /api/v1/auth/register（或等效）

### Story 1.3: 用户登录与登出

As a 已注册用户,
I want 使用手机号或邮箱登录并能在需要时登出,
So that 能够访问族谱内容并安全结束会话。

**Acceptance Criteria:**

**Given** 用户已注册
**When** 用户输入正确手机号/邮箱和密码并提交登录
**Then** 系统返回 JWT access token 与 refresh token
**And** 前端可将 token 存储并用于后续 API 请求
**When** 已登录用户点击登出
**Then** 前端清除 token，用户会话结束
**And** 登出后访问受保护接口返回 401

### Story 1.4: 管理员创建族谱

As a 已登录用户,
I want 创建族谱并成为该族谱的主管理员,
So that 能够开始录入家族成员与关系。

**Acceptance Criteria:**

**Given** 用户已登录
**When** 用户提交创建族谱请求（含族谱名称等必要信息）
**Then** 系统创建族谱并将该用户设为主管理员
**And** 族谱与用户关联正确，支持多族谱场景
**And** 非管理员无法创建族谱或需通过邀请流程（MVP 可简化为首建即为主管理员）

### Story 1.5: 成员 UUID 标识体系

As a 系统,
I want 为每个家族成员分配唯一 UUID,
So that 成员在系统中可唯一标识且便于关系关联。

**Acceptance Criteria:**

**Given** 需要创建成员数据模型
**When** 定义 FamilyMember 模型（或等效）
**Then** 成员主键或唯一标识使用 UUID
**And** 新建成员时系统自动分配 UUID
**And** UUID 在族谱内唯一，支持后续关系建立

---

## Epic 2: 成员管理

管理员能够添加、编辑、删除家族成员；成员能够查看自己的详情；成员能够提交生平修改申请；系统支持不精确日期。

### Story 2.1: 管理员添加家族成员

As a 管理员,
I want 添加家族成员并录入姓名、性别、出生日期、逝世日期、籍贯、生平、照片,
So that 族谱中有完整的成员基础信息。

**Acceptance Criteria:**

**Given** 管理员已登录且有所属族谱
**When** 管理员提交新成员信息（姓名、性别、出生日期、逝世日期、籍贯、生平、照片）
**Then** 系统创建成员并返回成功
**And** 必填字段校验正确，照片支持上传并存储
**And** 成员与族谱正确关联
**And** API 为 POST /api/v1/members

### Story 2.2: 支持不精确日期

As a 管理员,
I want 录入约略日期（如「约 1900 年」「清末」「不详」）,
So that 历史成员信息可如实记录。

**Acceptance Criteria:**

**Given** 成员有出生/逝世日期字段
**When** 管理员输入精确日期（如 1900-01-15）、约略日期（年/月/日精度）或文本（如「清末」「不详」）
**Then** 系统正确存储并在展示时统一规范显示
**And** 日期模型支持 JSON 或等效灵活结构
**And** 校验规则允许上述多种格式

### Story 2.3: 管理员修改与删除成员

As a 管理员,
I want 修改任意成员信息或删除无关联关系的成员,
So that 族谱数据保持准确。

**Acceptance Criteria:**

**Given** 管理员已登录
**When** 管理员提交成员信息修改
**Then** 系统更新成员并返回成功
**And** 有父子/配偶等关系的成员不可直接删除，或需先解除关系
**And** 删除无关联成员成功，并记录操作日志
**And** API 为 PUT /api/v1/members/{id} 和 DELETE /api/v1/members/{id}

### Story 2.4: 成员查看自己详情

As a 家族成员,
I want 查看自己的成员详情（照片、生平、关系等）,
So that 了解自己在族谱中的信息。

**Acceptance Criteria:**

**Given** 成员已登录且已关联到族谱中的某成员
**When** 成员请求自己的成员详情
**Then** 系统返回该成员的完整信息（照片、生平、关系、时间线等）
**And** 成员只能查看自己（或族谱内可见）的详情
**And** API 为 GET /api/v1/members/{id} 且需权限校验

### Story 2.5: 成员提交生平修改申请

As a 家族成员,
I want 提交对自己生平信息的修改申请,
So that 生平信息可由本人补充并经过审批后生效。

**Acceptance Criteria:**

**Given** 成员已登录且已关联到族谱成员
**When** 成员编辑自己的生平（富文本）并提交
**Then** 系统创建审批申请，不直接修改成员生平
**And** 申请状态为待审批
**And** 成员可查看自己提交的申请状态
**And** 富文本使用 TipTap 或等效组件

---

## Epic 3: 关系管理

管理员能够建立父子、母子、配偶关系；系统检测并拒绝循环引用与关系冲突；自动生成关系网；支持在树状图上点选建立关系。

### Story 3.1: 建立父子、母子、配偶关系

As a 管理员,
I want 建立成员之间的父子、母子、配偶关系,
So that 族谱能正确表达家族结构。

**Acceptance Criteria:**

**Given** 管理员已登录且族谱中有成员
**When** 管理员指定两个成员及关系类型（父子、母子、配偶）并提交
**Then** 系统创建关系并返回成功
**And** 关系类型与成员角色校验正确（如父子需区分父与子）
**And** API 为 POST /api/v1/relations
**And** 支持一对多（如多子女）场景

### Story 3.2: 循环引用与关系冲突检测

As a 系统,
I want 在建立或修改关系时检测循环引用与关系冲突,
So that 族谱数据逻辑正确。

**Acceptance Criteria:**

**Given** 管理员尝试建立或修改关系
**When** 操作会导致循环引用（如 A→B→C→A）
**Then** 系统拒绝并返回明确错误提示
**When** 操作会导致关系冲突（如 A 既是 B 的父亲又是 B 的儿子）
**Then** 系统拒绝并返回明确错误提示
**And** 检测逻辑在关系建立/修改时同步执行

### Story 3.3: 自动生成关系网

As a 系统,
I want 根据已录入的关系自动生成家族关系网,
So that 树状图与世系表可正确展示。

**Acceptance Criteria:**

**Given** 族谱中已有成员与关系
**When** 请求关系网数据
**Then** 系统返回图结构数据（节点、边）
**And** 千级规模下生成时间 < 10s（NFR5）
**And** 数据格式便于前端树状图/世系表渲染
**And** API 为 GET /api/v1/relations/graph 或等效

### Story 3.4: 树状图上点选建立关系

As a 管理员,
I want 在树状图上通过点选父/母/配偶来建立关系,
So that 建关系更直观高效。

**Acceptance Criteria:**

**Given** 树状图已展示且管理员已登录
**When** 管理员点选两个节点并选择关系类型（父、母、配偶）
**Then** 系统建立对应关系并刷新树状图
**And** 点选流程有明确引导与反馈
**And** 建立前仍执行循环与冲突检测

---

## Epic 4: 族谱展示

用户能够以树状图、世系表浏览家族；搜索并定位成员；查看成员详情（照片、生平、关系、时间线）；分享链接；树状图支持缩放与拖拽。

### Story 4.1: 树状图浏览

As a 已登录用户,
I want 以树状图形式浏览家族关系,
So that 直观了解家族结构。

**Acceptance Criteria:**

**Given** 用户已登录且族谱有成员与关系
**When** 用户进入树状图页面
**Then** 使用 AntV G6 渲染树状图
**And** 千级节点下初始渲染 < 2s（NFR2）
**And** 节点展示成员关键信息（如姓名）
**And** 父子、配偶等关系正确连线

### Story 4.2: 树状图缩放与拖拽

As a 已登录用户,
I want 对树状图进行缩放与拖拽,
So that 方便浏览大型家族结构。

**Acceptance Criteria:**

**Given** 树状图已展示
**When** 用户进行缩放（滚轮或控件）
**Then** 视图正确缩放
**When** 用户拖拽画布
**Then** 视图正确平移
**And** 交互流畅，无卡顿

### Story 4.3: 世系表浏览

As a 已登录用户,
I want 以世系表形式浏览家族关系,
So that 以表格形式查看世系信息。

**Acceptance Criteria:**

**Given** 用户已登录且族谱有成员与关系
**When** 用户切换到世系表视图
**Then** 以表格形式展示成员（含世系、姓名、生卒等）
**And** 世系层级正确
**And** 支持与树状图切换

### Story 4.4: 搜索并定位成员

As a 已登录用户,
I want 在树状图或世系表中搜索并定位成员,
So that 快速找到目标成员。

**Acceptance Criteria:**

**Given** 树状图或世系表已展示
**When** 用户输入成员姓名或关键词搜索
**Then** 系统筛选并高亮/定位匹配成员
**And** 树状图中可滚动至该成员并高亮
**And** 世系表中可筛选并定位

### Story 4.5: 查看成员详情

As a 已登录用户,
I want 点击成员查看其详情（照片、生平、关系、时间线）,
So that 像看电影一样了解先祖。

**Acceptance Criteria:**

**Given** 用户已登录
**When** 用户在树状图或世系表中点击某成员
**Then** 展示该成员详情页（照片、生平、关系、时间线等）
**And** 详情页加载 < 1s（NFR3）
**And** 照片正确加载，生平支持富文本展示
**And** 时间线展示重大事件（如有）

### Story 4.6: 分享成员或族谱链接

As a 已登录用户,
I want 分享成员或族谱的查看链接给其他族人,
So that 便于族人访问与传播。

**Acceptance Criteria:**

**Given** 用户已登录
**When** 用户点击分享并复制链接
**Then** 生成可访问的 URL（含成员 ID 或族谱 ID）
**And** 其他族人通过链接登录后可查看对应内容
**And** 链接格式清晰易读

---

## Epic 5: 生平修改审批

成员提交生平修改后自动发起审批；审批人可查看、通过或驳回；驳回时提供意见；申请人可修改后重新提交；通过后生效并站内信通知。

### Story 5.1: 生平修改自动发起审批

As a 系统,
I want 在成员提交生平修改时自动发起审批流程,
So that 修改需经审批才能生效。

**Acceptance Criteria:**

**Given** 成员已提交生平修改申请（Story 2.5）
**When** 申请创建成功
**Then** 系统自动创建审批任务，状态为待审批
**And** 审批人（管理员/族长）可看到该待办
**And** 申请与成员、修改内容正确关联

### Story 5.2: 审批人查看与处理申请

As a 审批人,
I want 查看待审批的生平修改申请并通过或驳回,
So that 确保族谱内容质量。

**Acceptance Criteria:**

**Given** 审批人已登录
**When** 审批人进入待办列表
**Then** 展示所有待审批的生平修改申请
**And** 审批人可查看申请详情（原内容、修改内容）
**When** 审批人选择通过
**Then** 成员生平更新为修改后内容，申请状态为已通过
**When** 审批人选择驳回并填写意见
**Then** 申请状态为已驳回，驳回意见保存

### Story 5.3: 驳回后重新提交

As a 申请人,
I want 在申请被驳回后修改内容并重新提交,
So that 修正后再次申请审批。

**Acceptance Criteria:**

**Given** 申请人的生平修改申请已被驳回
**When** 申请人查看驳回意见并修改内容后重新提交
**Then** 系统创建新的审批申请
**And** 新申请与旧申请可区分
**And** 审批人可看到新的待审批申请

### Story 5.4: 审批通过后生效与站内信通知

As a 申请人,
I want 审批通过后收到站内信通知,
So that 及时知晓修改已生效。

**Acceptance Criteria:**

**Given** 审批人通过了生平修改申请
**When** 审批操作完成
**Then** 成员生平立即更新
**And** 系统向申请人发送站内信：「您的生平修改已通过，现已生效」
**And** 申请人可在站内信列表中查看
**And** 支持未读/已读状态（可选）

---

## Epic 6: 批量导入

管理员能够下载 Excel 模板、上传文件、校验并展示错误报告；系统根据父亲/母亲姓名自动建立关系；确认预览后执行导入并触发关系网更新。

### Story 6.1: 下载 Excel 导入模板

As a 管理员,
I want 下载系统提供的 Excel 导入模板,
So that 按规范填写批量成员数据。

**Acceptance Criteria:**

**Given** 管理员已登录
**When** 管理员请求下载导入模板
**Then** 系统返回 Excel 文件
**And** 模板包含必要列：姓名、性别、出生日期、逝世日期、籍贯、父亲姓名、母亲姓名等
**And** 模板含示例行或说明
**And** API 为 GET /api/v1/import/template

### Story 6.2: 上传校验与错误报告

As a 管理员,
I want 上传 Excel 文件并在导入前看到校验结果与错误报告,
So that 修正错误后再导入。

**Acceptance Criteria:**

**Given** 管理员已准备好 Excel 文件
**When** 管理员上传文件
**Then** 系统解析并校验数据
**And** 返回预览数据与错误报告（行号、错误类型、说明）
**And** 格式错误、必填缺失、关系引用不存在等均有明确提示
**And** 校验通过时展示预览，不通过时阻止导入
**And** 50 人校验在合理时间内完成（NFR4 含校验）

### Story 6.3: 确认导入并自动建立关系

As a 管理员,
I want 确认预览后执行导入，系统根据父亲/母亲姓名自动建立关系,
So that 批量录入高效且关系正确。

**Acceptance Criteria:**

**Given** 校验通过且管理员已确认预览
**When** 管理员执行导入
**Then** 系统创建所有成员
**And** 根据「父亲姓名」「母亲姓名」自动建立父子、母子关系
**And** 姓名匹配逻辑正确（同族谱内）
**And** 导入完成后触发关系网更新
**And** 50 人导入 < 30s（NFR4）

### Story 6.4: 导入后关系网更新

As a 系统,
I want 在批量导入完成后触发关系网更新,
So that 树状图与世系表展示最新数据。

**Acceptance Criteria:**

**Given** 批量导入刚完成
**When** 关系网数据被请求
**Then** 返回包含新导入成员与关系的完整图数据
**And** 无需手动刷新或重建
**And** 与 Story 3.3 的关系网生成逻辑一致

---

## Epic 7: 系统管理与部署

管理员能够查看操作日志、执行备份、导出族谱数据；系统提供健康检查接口；支持容器化一键部署。

### Story 7.1: 操作日志

As a 管理员,
I want 查看操作日志（成员增删改、关系变更、审批等）,
So that 追溯与审计关键操作。

**Acceptance Criteria:**

**Given** 管理员已登录
**When** 管理员访问操作日志
**Then** 展示关键操作记录（操作人、时间、操作类型、对象、内容摘要）
**And** 支持按时间、操作类型筛选
**And** 成员增删改、关系变更、审批通过/驳回等均记录
**And** API 为 GET /api/v1/logs 或等效

### Story 7.2: 数据备份与导出

As a 管理员,
I want 执行数据备份并导出族谱数据,
So that 数据可迁移与恢复。

**Acceptance Criteria:**

**Given** 管理员已登录
**When** 管理员执行备份
**Then** 系统生成备份文件（含成员、关系、族谱等核心数据）
**And** 备份可下载或存储至指定位置
**When** 管理员执行导出
**Then** 系统导出族谱数据（如 JSON 或 Excel）
**And** 导出格式支持迁移至其他环境
**And** 满足数据归属与可控要求（NFR10）

### Story 7.3: 健康检查接口

As a 运维人员,
I want 系统提供健康检查接口,
So that 检测核心依赖组件连通性。

**Acceptance Criteria:**

**Given** 系统已部署
**When** 请求 GET /api/health/
**Then** 返回健康状态（如 200 OK）
**And** 可检测数据库、存储等核心依赖
**And** 不健康时返回非 2xx 及原因
**And** 适用于 Docker/K8s 健康检查

### Story 7.4: 容器化一键部署

As a 部署人员,
I want 使用 Docker 一键部署系统,
So that 快速在自有环境运行族谱应用。

**Acceptance Criteria:**

**Given** 具备 Docker 与 Docker Compose 环境
**When** 执行 docker-compose up
**Then** 前后端、数据库、Nginx 等服务启动
**And** 应用可访问且功能正常
**And** 配置通过 .env 管理，符合 12-factor
**And** 提供 docker-compose.yml 与 Dockerfile
