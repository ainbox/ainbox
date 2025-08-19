# AI Inbox - 产品需求文档 (PRD)

## 项目概述

### 项目名称
AI Inbox (ainbox)

### 项目愿景
打造一个智能化的个人信息收集整理工具，通过AI技术自动对用户随手记录的各种消息进行分类和标签化，形成便于查询和管理的个人信息空间。

### 核心价值
- **智能收集**：用户可以随时记录任何类型的信息和想法
- **自动分类**：AI自动将信息分类到合适的空间中
- **智能标签**：AI自动为信息打标签，用户可参与辅助
- **灵活查询**：支持跨空间搜索和标签视图查看
- **轻量化管理**：不是系统性知识库，专注于碎片化信息整理

## 技术架构

### 前端技术栈
- **Web版本**：React + TypeScript + Tailwind CSS + Vite
- **桌面版本**：Tauri 2.0 (基于Web技术栈)
- **状态管理**：Zustand
- **UI组件库**：Headless UI + 自定义组件
- **包管理器**：pnpm

### 后端技术栈
- **框架**：Rust + Axum
- **数据库**：PostgreSQL + SQLx
- **认证**：JWT + bcrypt

> **TODO**：完善JWT颁发、刷新、撤销和过期策略
- **AI集成**：DeepSeek API / Qwen API 等国内AI服务
- **部署**：Docker + Docker Compose

### 部署架构
- **柔性部署**：支持Web应用和桌面应用两种模式
- **容器化**：后端完全容器化部署
- **API优先**：前后端完全分离，通过HTTP API通信

## 功能模块

### 1. 用户认证与管理
- **用户注册/登录**：支持邮箱注册登录
- **用户配置**：个人偏好设置、AI参数调节
- **权限管理**：个人用户数据隔离

> **TODO**：后期支持第三方登录和账户绑定功能

### 2. 信息收集与录入
- **快速录入**：支持文本、语音、图片等多种格式
- **导入功能**：支持从文件、剪贴板等导入信息
- **批量录入**：支持一次性录入多条信息
- **移动端支持**：短期通过响应式网页支持，长期开发原生App

### 3. AI智能处理

> **TODO**：完善AI费用控制机制和服务fallback策略

#### 3.1 空间分类系统
- **默认空间**：系统为用户创建"待分类"默认空间
- **自动分类**：AI分析内容特征，自动分配到合适空间
- **智能提醒**：当同类卡片超过3张时，提醒用户创建新空间
- **用户预设**：用户可预设空间规则，AI按规则自动分类
- **空间管理**：用户可创建、编辑、删除、合并空间

#### 3.2 标签系统
- **AI自动打标**：基于内容语义分析自动生成标签
- **用户辅助**：用户可手动添加、修改、删除标签
- **标签建议**：AI基于历史数据提供标签建议
- **标签层次**：支持父子标签关系
- **标签统计**：显示标签使用频率和关联卡片量

### 4. 查询与视图系统

#### 4.1 空间视图
- **空间列表**：显示所有空间及其卡片统计
- **空间内容**：每张卡片唯一归属于一个空间
- **空间搜索**：在特定空间内快速查找卡片
- **空间排序**：按时间、重要性、卡片量等排序

#### 4.2 标签视图
- **标签云**：可视化展示所有标签及其权重
- **标签筛选**：按单个或多个标签筛选卡片
- **标签组合**：支持标签的AND/OR组合查询
- **交叉视图**：一张卡片可在多个标签下显示

#### 4.3 全局搜索
- **跨空间搜索**：在所有空间中搜索关键词
- **语义搜索**：基于AI的语义理解搜索

> **TODO**：完善语义搜索和向量搜索的技术方案
- **高级筛选**：按时间、空间、标签等多维度筛选
- **搜索历史**：仅在本地保存常用搜索条件，不跨设备同步

### 5. 卡片管理
- **卡片编辑**：支持修改卡片内容、空间归属、标签
- **卡片导出**：支持按空间或标签导出卡片数据
- **数据备份**：定期备份用户卡片数据
- **安全控制**：API限流机制，防止滥用
- **操作日志**：记录关键操作日志，便于审计和问题排查

## 接口设计规范

### HTTP接口规范
- **请求方法**：统一使用POST方法
- **参数格式**：参数放在JSON body中
- **响应格式**：返回字段使用小写+下划线命名格式

## 数据库设计

### 数据库规范
- **主键**：统一使用自增ID
- **外键约束**：不使用数据库外键约束，通过应用层维护关联关系
- **字段命名**：使用小写+下划线格式
- **时间字段**：统一使用BIGINT存储64位Unix时间戳

### 业务约束规则
- **默认空间约束**：`user_settings.default_space_id` 必须属于同一 `user_id`
- **空间删除规则**：每个用户至少保留一个空间，删除空间时需检查；若默认空间被删除，自动设置为用户的第一个空间
- **资源删除策略**：删除卡片时立即删除所有关联资源（文件、链接等），禁止多卡片共享资源

### 核心表结构

> **卡片数据模型说明**：每张卡片可包含多种内容类型的组合：
> - 文本内容：1个（可选）
> - 图片：多个，不超过100张
> - 音频：多个，不超过100条
> - 文件：多个，不超过100个
> - 链接：多个，不超过100条
> 
> 卡片内容通过 `card_contents` 表关联到具体资源：
> - text类型：直接在 `text_data` 字段存储文本内容
> - image/audio/file类型：通过 `file_id` 关联到 `files` 表
> - link类型：通过 `link_id` 关联到 `links` 表

#### users (用户表)
- `user_id` (BIGSERIAL, PRIMARY KEY)
- `email` (VARCHAR, UNIQUE)
- `password_hash` (VARCHAR)
- `created_at` (BIGINT) // 创建时间戳
- `updated_at` (BIGINT) // 更新时间戳

#### spaces (空间表)
- `space_id` (BIGSERIAL, PRIMARY KEY)  
- `user_id` (BIGINT) // 关联用户，支持快速检索
- `name` (VARCHAR)
- `description` (TEXT)
- `auto_rules` (JSON) // 自动分类规则
- `created_at` (BIGINT) // 创建时间戳
- `updated_at` (BIGINT) // 更新时间戳
- INDEX: `(user_id, created_at)`

#### cards (卡片表)
- `card_id` (BIGSERIAL, PRIMARY KEY)
- `user_id` (BIGINT) // 用户ID，支持快速检索  
- `space_id` (BIGINT) // 空间ID，支持快速检索
- `title` (VARCHAR) // 卡片标题（可选）
- `ai_summary` (TEXT) // AI生成的摘要
- `created_at` (BIGINT) // 创建时间戳
- `updated_at` (BIGINT) // 更新时间戳
- INDEX: `(user_id, space_id, created_at)`
- INDEX: `(user_id, created_at)`

#### card_contents (卡片内容表)
- `content_id` (BIGSERIAL, PRIMARY KEY)
- `card_id` (BIGINT) // 关联卡片ID
- `content_type` (VARCHAR) // 内容类型：text, image, audio, file, link
- `text_data` (TEXT) // 文本内容（仅text类型使用）
- `file_id` (BIGINT) // 关联文件ID（image/audio/file类型使用）
- `link_id` (BIGINT) // 关联链接ID（link类型使用）
- `display_order` (INT) // 显示顺序
- `created_at` (BIGINT) // 创建时间戳
- INDEX: `(card_id, display_order)`
- UNIQUE: `(card_id, display_order)`

#### tags (标签表)
- `tag_id` (BIGSERIAL, PRIMARY KEY)
- `user_id` (BIGINT) // 用户ID
- `name` (VARCHAR)
- `parent_tag_id` (BIGINT) // 父标签ID，支持层次结构，可为空（根标签）。约束：必须与子标签同user_id，禁止跨用户移动归属。防环策略：子标签ID必须大于父标签ID，在更新和合并操作时严格执行此规则
- `color` (VARCHAR) // 标签颜色
- `created_at` (BIGINT) // 创建时间戳
- UNIQUE: `(user_id, name)`

#### card_tags (卡片-标签关联表)
- `card_id` (BIGINT)
- `tag_id` (BIGINT)
- `confidence` (FLOAT) // AI打标的置信度
- `source` (VARCHAR) // 标签来源：ai, user
- `created_at` (BIGINT) // 创建时间戳
- PRIMARY KEY: `(card_id, tag_id)`
- INDEX: `(tag_id)`

#### links (链接表)
- `link_id` (BIGSERIAL, PRIMARY KEY)
- `user_id` (BIGINT) // 用户ID
- `url` (TEXT) // 链接地址
- `title` (VARCHAR) // 链接标题
- `description` (TEXT) // 链接描述
- `favicon_url` (VARCHAR) // 网站图标
- `created_at` (BIGINT) // 创建时间戳
- INDEX: `(user_id, created_at)`

#### files (文件表)
- `file_id` (BIGSERIAL, PRIMARY KEY)
- `user_id` (BIGINT) // 用户ID
- `file_path` (VARCHAR) // 文件存储路径
- `mime_type` (VARCHAR) // MIME类型
- `file_size` (BIGINT) // 文件大小（字节）
- `file_hash` (VARCHAR) // 文件哈希值，用于去重
- `uploaded_at` (BIGINT) // 上传时间戳（Unix时间戳）
- INDEX: `(user_id, uploaded_at)`
- INDEX: `(file_hash)`

#### user_settings (用户设置表)
- `setting_id` (BIGSERIAL, PRIMARY KEY)
- `user_id` (BIGINT) // 用户ID
- `ai_auto_tag` (BOOLEAN) // 是否开启AI自动打标
- `ai_auto_space` (BOOLEAN) // 是否开启AI自动分空间
- `default_space_id` (BIGINT) // 默认空间ID，必须属于同一user_id
- `settings_json` (JSON) // 其他设置
- `updated_at` (BIGINT) // 更新时间戳
- UNIQUE: `(user_id)`

## 部署方案

### Web应用部署
- **前端**：打包为静态文件，部署到CDN或静态网站托管
- **后端**：Docker容器部署，支持横向扩展
- **数据库**：PostgreSQL，支持主从复制

### 桌面应用部署
- **打包方式**：使用Tauri 2.0打包为原生应用
- **更新机制**：支持自动更新
- **本地缓存**：只读缓存，避免数据一致性问题

### 环境配置
- **开发环境**：Docker Compose一键启动
- **测试环境**：自动化CI/CD流水线
- **生产环境**：Kubernetes或Docker Swarm部署

## 开发计划

### Phase 1：基础框架搭建（2-3周）
- 前后端项目初始化
- 基础认证系统
- 数据库设计和迁移
- API框架搭建

### Phase 2：核心数据管理（3-4周）
- 数据录入功能
- 基础空间管理
- 默认空间创建
- 基础UI界面

### Phase 3：AI分类系统（4-5周）
- AI自动分类算法
- 空间智能分配
- 同类数据检测
- 新空间提醒机制

### Phase 4：标签系统（3-4周）
- AI自动打标功能
- 标签管理界面
- 标签视图开发
- 用户标签辅助

### Phase 5：搜索与查询（2-3周）
- 跨空间搜索功能
- 高级筛选系统
- 语义搜索集成
- 搜索性能优化

### Phase 6：桌面应用与部署（2-3周）
- Tauri桌面应用开发
- 全面测试
- 部署配置
- 文档完善

## 技术风险与解决方案

### 风险识别
1. **AI分类准确性**：AI自动分类可能不符合用户预期
2. **数据量增长**：用户数据快速增长带来的性能压力
3. **搜索性能**：跨空间全文搜索的性能瓶颈
4. **用户体验**：空间和标签管理的复杂性可能影响易用性

### 解决方案
1. **AI优化**：
   - 提供用户反馈机制，持续优化分类算法
   - 支持用户手动调整分类结果
   - 提供分类置信度显示
2. **性能优化**：
   - 数据库索引优化，支持user_id和space_id快速检索
   - 分页加载，避免一次性加载大量数据
   - 缓存常用搜索结果
3. **搜索优化**：
   - 使用全文搜索引擎（如Elasticsearch）
   - 异步搜索，提供搜索进度反馈
   - 智能搜索建议和历史记录
4. **用户体验**：
   - 提供简化的默认视图
   - 智能化的空间和标签建议
   - 详细的使用指南和帮助文档

---

*本文档将根据开发进展和用户反馈持续更新*
