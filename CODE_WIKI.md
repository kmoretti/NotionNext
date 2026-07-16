# NotionNext Code Wiki

> 本文档为 NotionNext 项目的完整技术参考手册，涵盖项目架构、模块职责、关键函数、依赖关系与运行方式。

---

## 目录

- [1. 项目概览](#1-项目概览)
- [2. 技术栈](#2-技术栈)
- [3. 项目目录结构](#3-项目目录结构)
- [4. 核心架构](#4-核心架构)
- [5. 配置系统](#5-配置系统)
- [6. 数据层（Notion 集成）](#6-数据层notion-集成)
- [7. 缓存系统](#7-缓存系统)
- [8. 路由与页面](#8-路由与页面)
- [9. 主题系统](#9-主题系统)
- [10. 组件架构](#10-组件架构)
- [11. 插件系统](#11-插件系统)
- [12. API 路由](#12-api-路由)
- [13. 认证系统](#13-认证系统)
- [14. 中间件](#14-中间件)
- [15. 构建与部署](#15-构建与部署)
- [16. 国际化（i18n）](#16-国际化i18n)
- [17. 工具函数](#17-工具函数)
- [18. 测试](#18-测试)
- [19. 开发与调试](#19-开发与调试)

---

## 1. 项目概览

### 1.1 简介

**NotionNext** 是一个基于 [Next.js](https://nextjs.org/) 的开源博客/站点生成框架，以 [Notion](https://www.notion.so/) 作为唯一内容数据源（CMS）。它将 Notion 数据库中的内容通过非官方 Notion API 拉取后，经过多层转换和缓存，最终以 SSG（静态站点生成）+ ISR（增量静态再生）方式渲染为高性能的博客页面。

- **仓库地址**: `https://github.com/tangly1024/NotionNext`
- **版本**: 4.10.6
- **协议**: MIT
- **Node.js 要求**: >= 22 < 25
- **包管理器**: Yarn 1.22.22

### 1.2 核心特性

| 特性 | 说明 |
|------|------|
| Notion 作为 CMS | 无需传统数据库，Notion 数据库即内容源 |
| 25 个主题 | 内置 25 套风格各异的主题，支持运行时热切换 |
| 多语言支持 | 从 Notion 页面 ID 级别支持多语言，自动检测与路由 |
| 多评论系统 | 集成 10+ 种评论方案（Giscus/Twikoo/Waline 等） |
| 插件化架构 | 分析统计、广告、搜索、AI 功能等均可按需启用 |
| 企业级认证 | 可选 Clerk 身份认证 + Supabase 后端 |
| 多部署方案 | Vercel、Docker、Netlify、Cloudflare Pages 等 |

---

## 2. 技术栈

### 2.1 核心框架

| 技术 | 版本 | 用途 |
|------|------|------|
| Next.js | ^14.2.30 | Web 框架（SSG/ISR/SSR） |
| React | ^18.3.1 | UI 渲染 |
| TypeScript | 5.9.3 | 类型系统（严格模式） |
| Tailwind CSS | ^3.4.19 | 原子化 CSS |

### 2.2 Notion 集成

| 技术 | 版本 | 用途 |
|------|------|------|
| notion-client | 7.10.0 | 非官方 Notion API SDK |
| react-notion-x | 7.10.0 | Notion block 渲染组件 |
| notion-utils | 7.10.0 | Notion 数据工具函数 |
| @notionhq/client | ^2.2.15 | Notion 官方 API SDK |

### 2.3 认证与数据

| 技术 | 版本 | 用途 |
|------|------|------|
| @clerk/nextjs | ^5.7.6 | 身份认证 |
| @supabase/supabase-js | ^2.107.0 | 后端数据服务 |
| ioredis | - | Redis 缓存 |
| memory-cache | - | 内存缓存 |

### 2.4 开发工具

| 技术 | 用途 |
|------|------|
| Jest | 单元测试 |
| ESLint | 代码检查 |
| Prettier | 代码格式化 |
| patch-package | 依赖补丁 |
| webpack-bundle-analyzer | 打包分析 |

---

## 3. 项目目录结构

```
NotionNext/
├── .github/                    # GitHub 工作流与模板
│   ├── workflows/              # CI/CD、Stale、Labeler 等
│   ├── ISSUE_TEMPLATE/         # Issue 模板
│   └── DISCUSSION_TEMPLATE/    # Discussion 模板
│
├── .vitepress/                 # VitePress 文档站点配置
│   ├── config.mts              # VitePress 配置
│   └── theme/                  # 文档站点主题
│
├── __tests__/                  # 测试文件
│   ├── components/             # 组件测试
│   ├── lib/                    # 库函数测试
│   └── themes/                 # 主题测试
│
├── components/                 # 全局共享组件（100+）
│   ├── NotionPage.js           # 核心：Notion 页面渲染
│   ├── Comment.js              # 评论系统入口
│   ├── SEO.js                  # SEO 管理
│   ├── ExternalPlugins.js      # 外部插件调度器
│   ├── ThemeSwitch.js          # 主题切换
│   ├── AISummary.js            # AI 摘要
│   └── ...                     # 更多组件
│
├── conf/                       # 配置子模块（20 个）
│   ├── notion.config.js        # Notion 数据库配置
│   ├── plugin.config.js        # 第三方插件配置
│   ├── comment.config.js       # 评论系统配置
│   ├── analytics.config.js     # 统计分析配置
│   ├── ai.config.js            # AI 功能配置
│   ├── ad.config.js            # 广告配置
│   └── ...                     # 更多配置模块
│
├── docs/                       # VitePress 文档源文件
│   ├── developer/              # 开发者文档
│   ├── user-guide/             # 用户指南
│   └── public/                 # 文档静态资源
│
├── functions/                  # Serverless 函数
│   └── api/docs-chat.ts        # AI 聊天 API
│
├── hooks/                      # React 自定义 Hooks
│   ├── useWindowSize.ts        # 窗口尺寸监听
│   └── useAdjustStyle.js       # 样式自适应
│
├── lib/                        # 核心库
│   ├── build/                  # 构建阶段逻辑
│   │   ├── buildEnv.js         # 构建环境配置
│   │   ├── prefetch.js         # 预热系统
│   │   └── staticPaths.js      # 静态路径生成
│   ├── cache/                  # 多级缓存系统
│   │   ├── cache_manager.js    # 缓存管理器
│   │   ├── redis_cache.js      # Redis 缓存
│   │   ├── local_file_cache.js # 本地文件缓存
│   │   ├── memory_cache.js     # 内存缓存
│   │   ├── vercel_cache.js     # Vercel 缓存
│   │   ├── file_lock.js        # 文件锁
│   │   └── build_session.js    # 构建会话
│   ├── config/                 # 配置验证
│   │   └── env-validation.js   # 环境变量校验
│   ├── db/                     # 数据访问层
│   │   ├── SiteDataApi.js      # 核心数据编排
│   │   └── notion/             # Notion API 交互
│   │       ├── getNotionAPI.js         # Notion 客户端
│   │       ├── getNotionPost.js        # 单页获取
│   │       ├── getPageProperties.js    # 属性解析
│   │       ├── getPostBlocks.js        # Block 拉取
│   │       ├── getAllPageIds.js         # 页面 ID 提取
│   │       ├── getAllCategories.js      # 分类聚合
│   │       ├── getAllTags.js            # 标签聚合
│   │       ├── getNotionConfig.js       # Notion 配置读取
│   │       ├── getMetadata.js           # 元数据
│   │       ├── mapImage.js              # 图片处理
│   │       ├── CustomNotionApi.ts       # 自定义 API 扩展
│   │       ├── RateLimiter.ts           # 限流器
│   │       └── ...                      # 更多
│   ├── lang/                   # 多语言翻译文件
│   │   ├── zh-CN.js            # 中文简体
│   │   ├── en-US.js            # 英语
│   │   └── ...                 # 更多语言
│   ├── middleware/              # 中间件逻辑
│   │   └── security.js         # 安全策略
│   ├── plugins/                # 内置插件
│   │   ├── aiSummary.js        # AI 摘要
│   │   ├── algolia.js          # Algolia 搜索
│   │   ├── busuanzi.js         # 不蒜子统计
│   │   ├── gtag.js             # Google Analytics
│   │   └── ...                 # 更多插件
│   ├── site/                   # 站点服务抽象层
│   │   ├── site.api.ts         # API 接口定义
│   │   ├── site.service.ts     # 服务实现
│   │   └── site.types.ts       # 类型定义
│   ├── utils/                  # 通用工具函数
│   │   ├── index.js            # 工具入口
│   │   ├── sitemap.js          # Sitemap 生成
│   │   ├── rss.js              # RSS 生成
│   │   ├── post.js             # 文章处理
│   │   └── ...                 # 更多工具
│   ├── config.js               # 配置读取入口
│   └── global.js               # 全局 React Context
│
├── pages/                      # Next.js 页面路由
│   ├── _app.js                 # App 入口
│   ├── _document.js            # HTML 文档
│   ├── index.js                # 首页
│   ├── [prefix]/               # 一级路由
│   ├── [prefix]/[slug]/        # 二级路由
│   ├── archive/                # 归档
│   ├── category/               # 分类
│   ├── tag/                    # 标签
│   ├── search/                 # 搜索
│   ├── page/                   # 分页
│   ├── api/                    # API 路由
│   ├── auth/                   # 认证页面
│   ├── sign-in/                # Clerk 登录
│   ├── sign-up/                # Clerk 注册
│   ├── dashboard/              # 用户仪表盘
│   ├── 404.js                  # 404 页面
│   └── 500.js                  # 500 页面
│
├── public/                     # 静态资源
│   ├── css/                    # 全局样式
│   ├── images/                 # 图片资源
│   ├── js/                     # 客户端脚本
│   └── webfonts/               # 字体文件
│
├── scripts/                    # 工具脚本
│   ├── translate/              # 多语言翻译脚本
│   ├── health-check.js         # 健康检查
│   └── ...                     # 更多脚本
│
├── styles/                     # 全局样式表
│   ├── globals.css             # 全局基础样式
│   ├── notion.css              # Notion 渲染样式
│   ├── prism-theme.css         # 代码高亮样式
│   └── utility-patterns.css    # Tailwind 工具类
│
├── themes/                     # 主题系统（25 个主题）
│   ├── theme.js                # 主题系统核心入口
│   ├── claude/                 # Claude 主题
│   ├── heo/                    # Heo 主题
│   ├── next/                   # Next 主题
│   ├── starter/                # Starter 主题
│   └── ...                     # 更多主题
│
├── blog.config.js              # 博客主配置文件
├── next.config.js              # Next.js 配置
├── middleware.ts                # 请求中间件
├── tailwind.config.js          # Tailwind 配置
├── tsconfig.json               # TypeScript 配置
├── Dockerfile                  # Docker 构建文件
├── package.json                # 项目依赖
└── yarn.lock                   # 依赖锁定
```

---

## 4. 核心架构

### 4.1 整体架构图

```
                    ┌─────────────────────────────────────────────┐
                    │              Notion 数据库（CMS）              │
                    └──────────────────┬──────────────────────────┘
                                       │
                    ┌──────────────────▼──────────────────────────┐
                    │     Notion API 交互层 (lib/db/notion/)        │
                    │  notionClient / RateLimiter / normalizeUtil  │
                    └──────────────────┬──────────────────────────┘
                                       │
                    ┌──────────────────▼──────────────────────────┐
                    │        数据编排层 (lib/db/SiteDataApi.js)      │
                    │  convertNotionToSiteData / getPageProperties │
                    └──────────────────┬──────────────────────────┘
                                       │
                    ┌──────────────────▼──────────────────────────┐
                    │          多级缓存 (lib/cache/)                 │
                    │      Redis → 本地文件 → 内存缓存               │
                    └──────────────────┬──────────────────────────┘
                                       │
                    ┌──────────────────▼──────────────────────────┐
                    │          配置系统 (blog.config.js)             │
                    │  conf/ 子模块 + 环境变量 + Notion 配置页面      │
                    └──────────────────┬──────────────────────────┘
                                       │
          ┌────────────────────────────┼────────────────────────┐
          │                            │                        │
┌─────────▼─────────┐   ┌─────────────▼─────────┐   ┌─────────▼─────────┐
│   页面路由层       │   │    主题系统              │   │   全局组件层        │
│   pages/          │   │    themes/              │   │   components/      │
│   getStaticProps  │   │    theme.js             │   │   100+ 组件        │
│   getStaticPaths  │   │    25 个主题            │   │   插件调度器        │
└─────────┬─────────┘   └─────────────┬─────────┘   └─────────┬─────────┘
          │                            │                        │
          └────────────────────────────┼────────────────────────┘
                                       │
                    ┌──────────────────▼──────────────────────────┐
                    │             Next.js 渲染引擎                   │
                    │        SSG + ISR + SSR 混合模式                │
                    └─────────────────────────────────────────────┘
```

### 4.2 数据流

```
1. 构建时/请求时触发 getStaticProps / getServerSideProps
2. 调用 fetchGlobalAllData({ pageId, locale })
   ├── getOrSetDataWithCache(cacheKey, ...)      // 二级缓存（全局数据）
   │   └── getSiteDataByPageId({ pageId })
   │       └── getOrSetDataWithCache(cacheKey, ...)  // 一级缓存（站点数据）
   │           ├── fetchNotionPageBlocks(pageId)      // 拉取原始 block
   │           └── convertNotionToSiteData(...)        // 转换为站点数据
   │               ├── getAllPageIds()                 // 提取页面 ID
   │               ├── getPageProperties()            // 解析属性
   │               ├── getConfigMapFromConfigPage()    // 读取配置
   │               └── 筛选/排序/聚合
   └── handleDataBeforeReturn()                    // 脱敏 + 定时发布检查
3. 数据传入页面组件，通过 DynamicLayout 按主题渲染
```

### 4.3 请求处理流程

```
HTTP 请求
    │
    ▼
middleware.ts
    ├── Clerk 认证检查（如已配置）
    ├── UUID 重定向（如未配置 Clerk）
    └── 匹配路由规则
    │
    ▼
Next.js 路由
    ├── pages/_app.js（全局 Provider、样式、主题）
    ├── pages/_document.js（HTML 结构、深色模式预设）
    └── 匹配的页面组件
        └── getStaticProps / getServerSideProps
            └── fetchGlobalAllData → 缓存 → Notion API
    │
    ▼
DynamicLayout → 主题布局组件
    ├── 全局组件（SEO、Comment、ExternalPlugins 等）
    └── 主题私有组件（Header、Footer、Sidebar 等）
```

---

## 5. 配置系统

### 5.1 配置优先级链

```
Notion 数据库 CONFIG 页面（最高优先级）
    ↓
环境变量 (process.env.XXX)
    ↓
conf/ 子模块中的默认值
    ↓
blog.config.js 中的默认值
    ↓
代码硬编码默认值（最低优先级）
```

读取函数：`siteConfig(key, defaultVal, extendConfig)`（[lib/config.js](lib/config.js)）

### 5.2 博客主配置（blog.config.js）

核心配置项：

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `NOTION_PAGE_ID` | Notion 数据库页面 ID（多语言逗号分隔） | 必填 |
| `API_BASE_URL` | Notion API 地址 | `https://www.notion.so/api/v3` |
| `THEME` | 当前主题 | `simple` |
| `LANG` | 默认语言 | `zh-CN` |
| `AUTHOR` | 作者名 | - |
| `BLOG_FAVICON` | 网站图标 | - |
| `APPEARANCE` | 外观模式 | `light` |
| `APPEARANCE_DARK_TIME` | 夜间模式时间段 | `[18, 6]` |
| `ENABLE_RSS` | RSS 开关 | `false` |
| `CAN_COPY` | 是否允许复制 | `true` |
| `CUSTOM_MENU` | 自定义菜单 | `false` |
| `NEXT_REVALIDATE_SECOND` | ISR 重验证间隔 | `60` |
| `REVALIDATION_TOKEN` | 重验证令牌 | - |
| `PSEUDO_STATIC` | 伪静态路径模式 | `false` |
| `UUID_REDIRECT` | UUID 重定向 | `false` |

### 5.3 配置子模块（conf/ 目录）

| 文件 | 职责 |
|------|------|
| [notion.config.js](conf/notion.config.js) | Notion 数据库字段映射（20+ 字段） |
| [comment.config.js](conf/comment.config.js) | 评论系统配置（10 种方案） |
| [analytics.config.js](conf/analytics.config.js) | 访问统计（9 种平台） |
| [ai.config.js](conf/ai.config.js) | AI 功能（摘要/聊天机器人） |
| [ad.config.js](conf/ad.config.js) | 广告配置（AdSense/WWAds） |
| [plugin.config.js](conf/plugin.config.js) | 第三方插件 API 密钥 |
| [animation.config.js](conf/animation.config.js) | 动效美化（樱花/雪花/星空） |
| [widget.config.js](conf/widget.config.js) | 悬浮挂件（宠物/音乐） |
| [code.config.js](conf/code.config.js) | 代码块样式配置 |
| [font.config.js](conf/font.config.js) | 自定义字体 |
| [image.config.js](conf/image.config.js) | 图片压缩与处理 |
| [performance.config.js](conf/performance.config.js) | 性能优化开关 |
| [layout-map.config.js](conf/layout-map.config.js) | 路由→布局映射 |
| [post.config.js](conf/post.config.js) | 文章列表设置 |
| [top-tag.config.js](conf/top-tag.config.js) | 置顶文章配置 |
| [right-click-menu.js](conf/right-click-menu.js) | 右键菜单 |
| [themeSwitch.manifest.js](conf/themeSwitch.manifest.js) | 主题切换清单 |
| [contact.config.js](conf/contact.config.js) | 联系方式 |
| [dev.config.js](conf/dev.config.js) | 开发调试 |
| [techgrow.config.js](conf/techgrow.config.js) | TechGrow 导流 |

### 5.4 环境变量分类

| 分类 | 关键变量 | 说明 |
|------|----------|------|
| 核心 | `NOTION_PAGE_ID`, `API_BASE_URL` | Notion 连接 |
| 外观 | `NEXT_PUBLIC_THEME`, `NEXT_PUBLIC_APPEARANCE` | 主题/暗色模式 |
| 语言 | `NEXT_PUBLIC_LANG` | 默认语言 |
| 字体 | `NEXT_PUBLIC_FONT_STYLE/URL/SANS/SERIF` | 自定义字体 |
| 搜索 | `NEXT_PUBLIC_ALGOLIA_*` | Algolia 搜索 |
| 评论 | `NEXT_PUBLIC_COMMENT_GISCUS/TWIKOO/WALINE/...` | 各评论系统 |
| 统计 | `NEXT_PUBLIC_ANALYTICS_*` | 分析工具 |
| 广告 | `NEXT_PUBLIC_ADSENSE_*` | 广告平台 |
| AI | `NEXT_PUBLIC_AI_*` | AI 功能 |
| 缓存 | `ENABLE_CACHE`, `REDIS_URL` | 缓存配置 |
| 构建 | `BUILD_PREFETCH_ENABLED`, `NOTION_BUILD_RATE_*` | 构建控制 |
| 认证 | `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk 认证 |
| 数据库 | `SUPABASE_URL/SECRET_KEY` | Supabase |
| 翻译 | `NOTION_TOKEN`, `TRANSLATOR_PROVIDER` | 翻译脚本 |

**命名规范**：
- `NEXT_PUBLIC_` 前缀：客户端可访问
- 无前缀：仅服务端（含密钥/Token）

---

## 6. 数据层（Notion 集成）

### 6.1 Notion 客户端（lib/db/notion/getNotionAPI.js）

全局单例模式，封装 `notion-client` SDK：

```javascript
// 关键特性
- 全局单例 globalStore.notion，避免重复创建
- 请求去重：globalStore.inflight Map，相同请求共享 Promise
- 构建阶段启用 RateLimiter 限流
- 拦截 syncRecordValues 做 API 兼容处理
```

### 6.2 数据编排层（lib/db/SiteDataApi.js）

核心函数 `fetchGlobalAllData()` 是整个数据流的顶层入口：

**输入参数**：
- `pageId`: Notion 页面 ID（可多语言逗号分隔）
- `locale`: 当前语言
- `from`: 调用来源（调试用）

**返回数据**：
```javascript
{
  allPages,        // 所有页面数组
  categoryOptions, // 分类选项
  tagOptions,      // 标签选项
  latestPosts,     // 最新文章
  customNav,       // 自定义导航菜单
  notionConfig,    // Notion 配置表数据
  siteInfo,        // 站点信息
  // ... 更多
}
```

### 6.3 Block 拉取（lib/db/notion/getPostBlocks.js）

```javascript
fetchNotionPageBlocks(id, from, options)
  // 关键特性：
  - 基于 cacheVersion（lastEditedDate）的缓存版本控制
  - p-limit 全局并发 15，每请求间隔 50ms
  - 失败重试 3 次，降级读取缓存
  - formatNotionBlock() 格式化：
    - 兼容新版双层嵌套格式 { value: { value: {...} } }
    - 清理 crdt 字段
    - 处理 sync_block 类型
    - 修复代码语言标识（C++ → cpp）
    - 修复文件/视频/音频 URL
```

### 6.4 数据转换（convertNotionToSiteData）

核心转换流程：

```
1. adapterNotionBlockMap()          → 统一 block 格式
2. normalizeNotionMetadata()        → 提取数据库元信息
3. normalizeCollection()            → 提取 collection + schema
4. getAllPageIds()                   → 提取所有页面 ID
5. fetchInBatches()                 → 补拉缺失 block
6. getPageProperties()              → 逐页解析属性
7. getConfigMapFromConfigPage()     → 读取站点配置
8. adjustPageProperties()           → 处理 URL 前缀/伪静态/密码
9. fetchMembersFromOfficialAPI()    → 补充 Member 数据
10. 筛选、排序、生成衍生数据
```

### 6.5 页面属性解析（getPageProperties.js）

| 属性类型 | 处理方式 |
|----------|----------|
| 文本 | `getTextContent()` |
| 日期 | `getDateValue()` |
| 选择 | 逗号分割为数组 |
| 人物 | 调用 `notionAPI.getUsers()` 获取详情 |
| URL | 直接读取 |
| 布尔 | 直接读取 |

映射用户自定义字段名（通过 `BLOG.NOTION_PROPERTY_NAME`），支持 type/status 中文值→英文标准值。

### 6.6 图片处理（mapImage.js）

```javascript
mapImgUrl(img)
  - 相对路径拼接 NOTION_HOST
  - 旧图床转为 Notion 签名 URL
  - 支持随机图片替换
  - 追加 t={blockId} 防缓存

compressImage(img)
  - Notion 图床：拼接 width + cache=v2
  - Unsplash：拼接 q/width/fmt 参数
  - 默认宽度由 IMAGE_COMPRESS_WIDTH 配置
```

### 6.7 Notion 配置页面（getNotionConfig.js）

从 `type === 'CONFIG'` 的 Notion 页面中读取配置表：
- 子数据库格式：`启用(Enable) + 配置名(Name) + 配置值(Value)`
- 支持 `CONTACT_EMAIL` 加密、`INLINE_CONFIG` 展开合并
- 优先级最高，覆盖环境变量和 blog.config.js

### 6.8 数据兼容层（normalizeUtil.js）

```javascript
normalizeNotionMetadata()  → 兼容新老版本元数据包装
normalizeCollection()      → 最多剥 3 层 { value: {...}, role } 包装
normalizeSchema()          → 确保 schema 字段有 name 和 type
normalizePageBlock()       → 最多剥 5 层，返回 { id, type, properties }
```

---

## 7. 缓存系统

### 7.1 多级缓存架构

采用**链式缓存**设计，按优先级依次尝试：

```
Redis → 本地文件 → 内存
```

选择逻辑：
- 有 `REDIS_URL` → Redis 优先
- 构建阶段 / 非生产 → 文件缓存
- 内存缓存始终兜底

管理器：[lib/cache/cache_manager.js](lib/cache/cache_manager.js)

### 7.2 缓存层实现

| 缓存层 | 文件 | TTL | 说明 |
|--------|------|-----|------|
| Redis | `redis_cache.js` | revalidate × 1.5 | JSON 序列化，ioredis |
| 本地文件 | `local_file_cache.js` | 可配置 | 原子写入（rename），`.next/cache/notion/data/` |
| 内存 | `memory_cache.js` | 生产 10min / 开发 120min | memory-cache 库 |
| Vercel | `vercel_cache.js` | 可配置 | tag-based 失效 |

### 7.3 防击穿机制

- **inflightMap**: 相同 key 的并发请求共享同一 Promise
- **文件锁**（`file_lock.js`）: 构建阶段跨进程保护
  - 心跳机制（每 15 秒更新）
  - 陈旧锁检测（超过 120 秒自动清理）
  - 进程存活检测（`process.kill(pid, 0)`）
  - 策略：`bypass`（跳过）/ `wait`（等待）/ `throw`（抛异常）

### 7.4 构建会话（build_session.js）

- 缓存根目录：`.next/cache/notion/`（优先），只读回退到 `os.tmpdir()`
- 支持 `NOTION_NEXT_NOTION_CACHE_DIR` 环境变量
- 会话 ID 从 `build-session.json` 读取，支持多轮构建隔离

### 7.5 缓存清理 API

```http
POST /api/cache
Authorization: Bearer <CACHE_REVALIDATION_TOKEN>
```

### 7.6 按需重验证 API

```http
POST /api/revalidate
Authorization: Bearer <REVALIDATION_TOKEN>

# 单页刷新
{ "path": "/article/my-post" }

# 批量刷新
{ "paths": ["/", "/article/post-1"] }

# 全站刷新
{ "all": true }
```

---

## 8. 路由与页面

### 8.1 路由总览

| 路由 | 文件 | 渲染方式 | 说明 |
|------|------|----------|------|
| `/` | `pages/index.js` | SSG + ISR | 首页 |
| `/[prefix]` | `pages/[prefix]/index.js` | SSG + ISR | 一级页面（万能路由） |
| `/[prefix]/[slug]` | `pages/[prefix]/[slug]/index.js` | SSG + ISR | 二级页面 |
| `/[prefix]/[slug]/[...suffix]` | `pages/[prefix]/[slug]/[...suffix].js` | SSG + ISR | 三级+ 页面 |
| `/page/[page]` | `pages/page/[page].js` | SSG + ISR | 分页列表 |
| `/archive` | `pages/archive/index.js` | SSG + ISR | 归档页 |
| `/category` | `pages/category/index.js` | SSG + ISR | 分类索引 |
| `/category/[category]` | `pages/category/[category]/index.js` | SSG + ISR | 分类文章列表 |
| `/tag` | `pages/tag/index.js` | SSG + ISR | 标签索引 |
| `/tag/[tag]` | `pages/tag/[tag]/index.js` | SSG + ISR | 标签文章列表 |
| `/tag/[tag]/page/[page]` | `pages/tag/[tag]/page/[page].js` | SSG + ISR | 标签分页 |
| `/search` | `pages/search/index.js` | SSG + ISR | 前端搜索 |
| `/search/[keyword]` | `pages/search/[keyword]/index.js` | SSG + ISR (blocking) | 服务端全文搜索 |
| `/sitemap.xml` | `pages/sitemap.xml.js` | SSR | 动态 Sitemap |
| `/auth` | `pages/auth/index.js` | SSR | Notion OAuth |
| `/auth/result` | `pages/auth/result.js` | SSG | 授权结果 |
| `/sign-in/[[...index]]` | `pages/sign-in/[[...index]].js` | SSG + ISR | Clerk 登录 |
| `/sign-up/[[...index]]` | `pages/sign-up/[[...index]].js` | SSG + ISR | Clerk 注册 |
| `/dashboard/[[...index]]` | `pages/dashboard/[[...index]].js` | SSG + ISR | 用户仪表盘 |
| `/404` | `pages/404.js` | SSG + ISR | 404 页面 |
| `/500` | `pages/500.js` | SSR | 500 页面 |

### 8.2 全局布局文件

#### _app.js

```javascript
// 职责：全局入口组件
- 引入全局样式（globals.css、notion.css）
- 解析主题（URL 参数 > Notion 配置 > blog.config）
- GlobalContextProvider 全局状态
- ClerkProvider（可选）认证包裹
- SEO 组件
- ExternalPlugins 插件加载
- AppErrorBoundary 错误边界
```

#### _document.js

```javascript
// 职责：自定义 HTML 文档
- 深色模式预设脚本（避免闪烁）
  - 读取 localStorage.darkMode
  - 支持 auto 模式（APPEARANCE_DARK_TIME 定时切换）
- Font Awesome 预加载
- html lang 设置
- Unsplash DNS 预解析
```

### 8.3 万能路由机制

NotionNext 的核心路由设计是 `[prefix]` + `[slug]` 的万能路由：

- **一级页面**：`/about` → `[prefix]/index.js`
- **二级页面**：`/article/about` → `[prefix]/[slug]/index.js`
- **三级+页面**：`/article/2023/10/29/test` → `[prefix]/[slug]/[...suffix].js`

所有层级统一复用 `Slug` 组件和 `resolvePostProps` 数据获取逻辑。

### 8.4 密码保护

`[prefix]/index.js` 支持文章密码保护：

```
1. post.password 非空 → 显示密码输入界面
2. 支持 MD5（旧版兼容）和 SHA-256（新版）哈希
3. 验证成功 → localStorage 记录，下次自动提交
4. 解锁后 → 生成目录（getPageTableOfContents）并渲染内容
```

### 8.5 构建时副作用

首页 `getStaticProps` 在构建时（`yarn build`）执行：

```
- 生成 robots.txt
- 生成 RSS Feed
- 生成 sitemap.xml
- 检查 Algolia 索引清理
- 生成 UUID 重定向 JSON（如启用）
```

---

## 9. 主题系统

### 9.1 所有可用主题（25 个）

| 主题 | 风格 | 特色功能 |
|------|------|----------|
| **simple** | 三栏极简 | 动态懒加载组件，高度可配置 |
| **next** | 三栏博客 | 阅读进度条、侧边栏、Algolia 搜索 |
| **heo** | 上中下大图 | Hero 大图、通知栏、个人资料卡牌 |
| **hexo** | 左右双栏 | Hero 首页、随机跳转、分类导航 |
| **medium** | 左右双栏 | 杂志风格、目录、Live2D |
| **starter** | SaaS 落地页 | Hero/Features/Pricing/FAQ/Team、Clerk 集成 |
| **gitbook** | 文档知识库 | NavPostList 导航树、仪表盘 |
| **commerce** | 电商 | 产品中心/分类组件 |
| **magzine** | 杂志卡片 | Banner/Swiper、多种卡片样式 |
| **game** | 游戏 | GameEmbed、PWA、全屏 |
| **landing** | 落地页 | Hero/Pricing/Testimonials/Newsletter |
| **fukasawa** | 瀑布流 | 类 FukuSawa 风格、分组标签 |
| **fuwari** | 卡片网格 | 主题色切换、公告卡片 |
| **typography** | 排版 | 文字排版优先 |
| **endspace** | 多模式 | 侧边导航、背景动效 |
| **proxio** | 工作室 | 代理风格、Clerk 集成 |
| **thoughtlite** | 时间线 | HomeTimeline、LatestCard |
| **plog** | 摄影日志 | 图片优先 |
| **photo** | 照片 | 图片画廊 |
| **movie** | 影视 | 影视资源展示 |
| **nav** | 导航站 | 导航站风格 |
| **nobelium** | 极简 | 类 Nobelium 博客 |
| **matery** | 卡片 | 卡片式布局 |
| **example** | 示例 | 主题开发模板 |

### 9.2 主题标准结构

```
themes/<theme-name>/
├── index.js       # 布局组件导出（必须）
├── config.js      # 主题配置常量（必须）
├── style.js       # 主题 CSS 样式（必须）
└── components/    # 主题私有 UI 组件（必须）
```

### 9.3 必须导出的布局组件

```javascript
export {
  Layout404,            // 404 页面
  LayoutArchive,        // 归档页
  LayoutBase,           // 基础布局框架
  LayoutCategoryIndex,  // 分类索引
  LayoutIndex,          // 首页
  LayoutPostList,       // 文章列表
  LayoutSearch,         // 搜索页
  LayoutSlug,           // 文章详情页
  LayoutTagIndex,       // 标签索引
  CONFIG as THEME_CONFIG // 主题配置
}

// 部分主题额外导出：
export { LayoutDashboard, LayoutSignIn, LayoutSignUp }
```

### 9.4 主题加载机制（themes/theme.js）

```
1. 构建时扫描 themes/ 目录 → THEMES 数组
2. 运行时按需动态加载 → next/dynamic + import()
3. URL 参数 ?theme=xxx → 实时切换主题
4. 缓存已加载布局 → baseLayoutCache / layoutByThemeCache
5. 切换时 DOM 修复 → fixThemeDOM() 清理残留元素
6. 暗色模式管理 → initDarkMode()
```

### 9.5 暗色模式优先级

```
URL ?mode=dark > localStorage.darkMode > 站点强制设置 > 系统偏好
```

支持 `auto` 模式：根据 `APPEARANCE_DARK_TIME`（默认 18:00-06:00）自动切换。

---

## 10. 组件架构

### 10.1 组件分类

#### 核心渲染组件

| 组件 | 文件 | 职责 |
|------|------|------|
| NotionPage | [components/NotionPage.js](components/NotionPage.js) | 封装 `react-notion-x`，渲染 Notion block 内容 |
| NotionLink | `components/NotionLink.js` | Notion 内部链接处理 |

#### 评论系统

| 组件 | 文件 | 说明 |
|------|------|------|
| Comment | [components/Comment.js](components/Comment.js) | 统一入口，IntersectionObserver 懒加载 |
| Giscus | `components/Giscus.js` | GitHub Discussions 评论 |
| Twikoo | `components/Twikoo.js` | 腾讯云评论 |
| Waline | `components/WalineComponent.js` | Waline 评论 |
| Valine | `components/ValineComponent.js` | LeanCloud 评论 |
| Artalk | `components/Artalk.js` | Artalk 评论 |
| Cusdis | `components/CusdisComponent.js` | Cusdis 评论 |
| Utterances | `components/Utterances.js` | GitHub Issues 评论 |
| Gitalk | `components/Gitalk.js` | Gitalk 评论 |
| WebMention | `components/WebMention.js` | Web 协议评论 |
| NotionComments | `components/NotionComments.js` | Notion 原生评论 |

#### SEO 与分享

| 组件 | 文件 | 职责 |
|------|------|------|
| SEO | [components/SEO.js](components/SEO.js) | Open Graph、JSON-LD、WebFontLoader |
| ShareBar | `components/ShareBar.js` | 分享栏容器 |
| ShareButtons | `components/ShareButtons.js` | 20+ 分享平台支持 |

#### 状态与导航

| 组件 | 文件 | 职责 |
|------|------|------|
| ThemeSwitch | `components/ThemeSwitch.js` | 可拖拽主题切换按钮 + 侧边栏 |
| DarkModeButton | `components/DarkModeButton.js` | 深色模式切换 |
| Loading | `components/Loading.js` | 基础加载占位 |
| LoadingCover | `components/LoadingCover.js` | 页面加载遮罩 |
| LoadingProgress | `components/LoadingProgress.js` | NProgress 进度条 |

#### 功能组件

| 组件 | 文件 | 职责 |
|------|------|------|
| AISummary | [components/AISummary.js](components/AISummary.js) | AI 摘要 + 打字机动画 |
| LazyImage | `components/LazyImage.js` | 高级图片懒加载 + 自动压缩 |
| PrismMac | `components/PrismMac.js` | 代码高亮 + Mac 风格 + Mermaid |
| WordCount | `components/WordCount.js` | 字数统计 + 阅读时间 |
| Busuanzi | `components/Busuanzi.js` | 不蒜子访客计数 |
| ExternalPlugins | [components/ExternalPlugins.js](components/ExternalPlugins.js) | 外部插件中央调度器 |
| ExternalScript | `components/ExternalScript.js` | 动态加载外部脚本 |
| GlobalStyle | `components/GlobalStyle.js` | 全局 CSS 注入 |

### 10.2 外部插件调度器（ExternalPlugins.js）

```javascript
// 通过配置条件性加载插件，使用 requestIdleCallback 延迟加载非关键插件

分析统计：Google Analytics / 百度统计 / CNZZ / Busuanzi / Ackee / Vercel Analytics / Matomo / 51LA / Umami / Clarity
广告系统：Google AdSense / 万维广告 (WWAds)
AI 功能：TianliGPT / DifyChatbot / ChatBase / Coze
视觉特效：Fireworks / Sakura / StarrySky / Nest / FlutteringRibbon / Ribbon
交互功能：CustomContextMenu / DisableCopy / MouseFollow / AOSAnimation
其他：LoadingProgress / MusicPlayer / VConsole / Facebook Messenger
```

---

## 11. 插件系统

### 11.1 内置插件（lib/plugins/）

| 插件 | 文件 | 职责 |
|------|------|------|
| AI 摘要 | `aiSummary.js` | 调用 AI API 生成文章摘要 |
| Algolia | `algolia.js` | Algolia 搜索索引同步 |
| 不蒜子 | `busuanzi.js` | 站点访问统计 |
| Google Analytics | `gtag.js` | GA 跟踪代码 |
| 邮件加密 | `mailEncrypt.js` | 邮箱地址加密防爬 |
| Mailchimp | `mailchimp.js` | 邮件订阅 |
| 化学公式 | `mhchem.js` | LaTeX 化学方程式 |
| Notion 评论 | `notionComments.js` | Notion 评论系统 |
| 字数统计 | `wordCount.js` | 文章字数统计 |
| 动效 | `wow.js` | 滚动动画 |

### 11.2 插件加载方式

插件在两个层级加载：

1. **组件级**：通过 `ExternalPlugins.js` 在 `_app.js` 中全局加载
2. **数据级**：在 `getStaticProps` 中服务端执行（如 algolia 索引同步）

---

## 12. API 路由

| 路由 | 方法 | 文件 | 职责 |
|------|------|------|------|
| `/api/rss` | GET | [pages/api/rss.js](pages/api/rss.js) | RSS Feed（支持 RSS/Atom/JSON），10 分钟缓存 |
| `/api/cache` | POST | [pages/api/cache.js](pages/api/cache.js) | 清理本地文件缓存，Bearer Token 鉴权 |
| `/api/revalidate` | POST | [pages/api/revalidate.js](pages/api/revalidate.js) | 按需 ISR 重验证，Bearer Token 鉴权 |
| `/api/notion-comments` | GET/POST | [pages/api/notion-comments.js](pages/api/notion-comments.js) | Notion 评论 API，IP 限流 5 次/分钟 |
| `/api/auth/callback/notion` | GET | [pages/api/auth/callback/notion.ts](pages/api/auth/callback/notion.ts) | Notion OAuth 回调 |
| `/api/user` | GET | [pages/api/user.ts](pages/api/user.ts) | Clerk 认证测试端点 |
| `/api/subscribe` | POST | [pages/api/subscribe.js](pages/api/subscribe.js) | Mailchimp 邮件订阅 |

---

## 13. 认证系统

### 13.1 双认证系统

NotionNext 支持两套独立的认证系统，通过环境变量开关控制：

#### Notion OAuth

```
入口：/auth → 重定向到 Notion 授权页面
回调：/api/auth/callback/notion?code=xxx
结果：/auth/result?msg=...

环境变量：OAUTH_CLIENT_ID, OAUTH_CLIENT_SECRET, OAUTH_REDIRECT_URI
```

#### Clerk 认证

```
登录：/sign-in/[[...index]]
注册：/sign-up/[[...index]]
仪表盘：/dashboard/[[...index]]

环境变量：NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY

保护规则（middleware.ts）：
- /dashboard → 未登录重定向到 /sign-in
- /admin/*/memberships → 需要管理员权限
- /admin/*/domain → 需要管理员权限
```

---

## 14. 中间件

### 14.1 middleware.ts

```typescript
// 匹配范围：排除静态资源、_next、/sign-in、/auth
// 匹配 /、所有 API 路由

模式 1（已配置 Clerk）：
  clerkMiddleware() 认证
  - /dashboard → 未登录重定向
  - /admin/* → 管理员权限检查

模式 2（未配置 Clerk）：
  noAuthMiddleware()
  - 从 /redirect.json 加载 UUID 映射
  - 识别 Notion ID 并 308 重定向到 slug
```

### 14.2 安全中间件（lib/middleware/security.js）

服务端安全策略模块。

---

## 15. 构建与部署

### 15.1 核心命令

```bash
# 开发
yarn dev              # 启动开发服务器

# 构建
yarn build            # 构建生产版本（BUILD_MODE=true）
yarn export           # 静态导出模式

# 测试
yarn test             # 运行 Jest 测试
yarn test:ci          # CI 模式 + 覆盖率

# 代码质量
yarn lint             # ESLint 检查
yarn lint:fix         # 自动修复
yarn type-check       # TypeScript 类型检查
yarn format           # Prettier 格式化

# 其他
yarn translate        # 多语言翻译
yarn bundle-report    # 打包分析
```

### 15.2 Docker 部署

三阶段构建：

```dockerfile
# 阶段 1：依赖安装
FROM node:22-alpine AS deps
RUN yarn install --frozen-lockfile

# 阶段 2：构建
FROM node:22-alpine AS builder
ARG NOTION_PAGE_ID
ENV NEXT_BUILD_STANDALONE=true
RUN yarn build

# 阶段 3：运行
FROM node:22-alpine AS runner
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

### 15.3 Vercel 部署

- 支持一键部署
- 支持 ISR（增量静态再生）
- 支持按需重验证 API（`/api/revalidate`）
- 使用 Vercel 缓存（`vercel_cache.js`）

### 15.4 Netlify 部署

配置文件：`netlify.toml`

### 15.5 静态导出

```bash
yarn export
# 生成 out/ 目录，可部署到任何静态托管
```

### 15.6 next.config.js 关键配置

```javascript
// 三种输出模式
output: 'export' | 'standalone' | 默认

// 图片优化
images: { formats: ['avif', 'webp'], unoptimized: true }

// 路由重写
rewrites: [
  '/zh/xxx' → '/xxx',        // 多语言路径
  '/rss' → 静态文件 fallback  // RSS 路由
]

// 路由重定向
redirects: ['/feed' → '/rss/feed.xml']

// Webpack 定制
alias: { '@': rootDir }
// 禁用 Node.js 模块 polyfill
// 自动扫描主题目录
```

---

## 16. 国际化（i18n）

### 16.1 多语言机制

NotionNext 的多语言方案基于 Notion 页面 ID：

```
NOTION_PAGE_ID=pageId1,zh:pageId2,en:pageId3
```

- 从 `NOTION_PAGE_ID` 中自动检测多语言配置
- 格式：`pageId,zh:pageId,en:pageId`
- 每种语言对应一个独立的 Notion 数据库

### 16.2 支持的语言

| 语言代码 | 文件 | 语言 |
|----------|------|------|
| zh-CN | `lib/lang/zh-CN.js` | 中文简体 |
| zh-TW | `lib/lang/zh-TW.js` | 中文繁体 |
| zh-HK | `lib/lang/zh-HK.js` | 中文香港 |
| en-US | `lib/lang/en-US.js` | 英语 |
| ja-JP | `lib/lang/ja-JP.js` | 日语 |
| fr-FR | `lib/lang/fr-FR.js` | 法语 |
| tr-TR | `lib/lang/tr-TR.js` | 土耳其语 |

### 16.3 翻译脚本

`scripts/translate/` 目录提供 AI 驱动的内容翻译：

```bash
yarn translate       # 翻译当前语言
yarn translate:all   # 翻译所有语言
```

支持的翻译提供商：GLM、DeepSeek（通过 `TRANSLATOR_PROVIDER` 环境变量选择）

---

## 17. 工具函数

### 17.1 lib/utils/ 核心工具

| 文件 | 函数 | 职责 |
|------|------|------|
| `index.js` | - | 工具入口，导出所有工具函数 |
| `sitemap.js` | - | Sitemap XML 生成 |
| `sitemap.xml.js` | - | Sitemap 内容构建 |
| `rss.js` | - | RSS Feed 生成 |
| `robots.txt.js` | - | robots.txt 生成 |
| `post.js` | - | 文章数据处理 |
| `formatDate.js` | - | 日期格式化 |
| `font.js` | - | 字体配置 |
| `lang.js` | - | 语言工具 |
| `password.js` | - | 密码哈希（MD5/SHA-256） |
| `pageId.js` | - | Notion 页面 ID 处理 |
| `redirect.js` | - | UUID 重定向映射 |
| `pinnedPosts.js` | - | 置顶文章处理 |
| `articleCopyright.js` | - | 文章版权声明 |
| `copyPermission.js` | - | 复制权限控制 |
| `debounce.js` | - | 防抖函数 |
| `throttle.js` | - | 节流函数 |
| `errorHandler.js` | - | 错误处理 |
| `notion.util.js` | - | Notion 数据工具 |
| `validation.js` | - | 数据验证 |
| `buildMode.js` | - | 构建模式检测 |
| `clean.util.ts` | - | 数据清理 |
| `time.util.ts` | - | 时间工具 |

### 17.2 lib/db/notion/ 关键函数

| 函数 | 文件 | 职责 |
|------|------|------|
| `fetchGlobalAllData` | SiteDataApi.js | 全站数据获取入口 |
| `getSiteDataByPageId` | SiteDataApi.js | 单站点数据获取 |
| `resolvePostProps` | SiteDataApi.js | 文章页面数据解析 |
| `convertNotionToSiteData` | SiteDataApi.js | Notion→站点数据转换 |
| `handleDataBeforeReturn` | SiteDataApi.js | 返回数据脱敏 |
| `getNotionAPI` | getNotionAPI.js | Notion 客户端单例 |
| `getPageProperties` | getPageProperties.js | 页面属性解析 |
| `fetchNotionPageBlocks` | getPostBlocks.js | Block 数据拉取 |
| `formatNotionBlock` | getPostBlocks.js | Block 格式化 |
| `fetchInBatches` | getPostBlocks.js | 批量补拉 |
| `getAllPageIds` | getAllPageIds.js | 提取所有页面 ID |
| `getAllCategories` | getAllCategories.js | 分类聚合 |
| `getAllTags` | getAllTags.js | 标签聚合 |
| `getConfigMapFromConfigPage` | getNotionConfig.js | 读取 Notion 配置 |
| `mapImgUrl` | mapImage.js | 图片 URL 处理 |
| `compressImage` | mapImage.js | 图片压缩 |
| `getPageContentText` | getPageContentText.js | 提取纯文本内容 |
| `convertInnerUrl` | convertInnerUrl.js | Notion ID→slug 转换 |

---

## 18. 测试

### 18.1 测试框架

- **框架**: Jest
- **配置**: `jest.config.js`
- **环境**: `jest.env.js` + `jest.setup.js`

### 18.2 测试命令

```bash
yarn test              # 运行测试
yarn test:ci           # CI 模式（覆盖率）
```

### 18.3 测试文件

```
__tests__/
├── components/
│   ├── SEO.test.js
│   ├── NotionLink.test.js
│   └── LazyImage.test.js
├── lib/
│   ├── utils/
│   │   └── rss.test.js
│   ├── sitemap-utils.test.js
│   ├── sitemap.xml.test.js
│   └── staticPaths.test.js
└── themes/
    ├── proxio/
    │   ├── Blog.test.js
    │   └── MenuList.test.js
    ├── heo/
    │   └── InfoCard.test.js
    ├── starter/
    │   └── Features.test.js
    ├── endspace/
    │   └── menu.test.js
    └── magzine/
        └── PostListRecommend.test.js
```

---

## 19. 开发与调试

### 19.1 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/tangly1024/NotionNext.git
cd NotionNext

# 2. 安装依赖
yarn install

# 3. 复制环境变量模板
cp .env.example .env

# 4. 配置 Notion 页面 ID（必填）
# 编辑 .env，设置 NOTION_PAGE_ID

# 5. 启动开发服务器
yarn dev

# 6. 访问 http://localhost:3000
```

### 19.2 最小配置

```bash
# .env
NOTION_PAGE_ID=你的Notion数据库页面ID
NEXT_PUBLIC_THEME=starter
NEXT_PUBLIC_LANG=zh-CN
NEXT_PUBLIC_AUTHOR=你的名字
```

### 19.3 主题切换

```bash
# 运行时切换（无需重启）
http://localhost:3000?theme=heo
http://localhost:3000?theme=next
http://localhost:3000?theme=hexo

# 或修改 blog.config.js
THEME: process.env.NEXT_PUBLIC_THEME || 'starter'
```

### 19.4 调试工具

- **VConsole**: 启用 `NEXT_PUBLIC_VCONSOLE=true` 显示移动端调试面板
- **DebugPanel**: 开发环境可显示调试面板
- **webpack-bundle-analyzer**: `yarn bundle-report` 分析打包体积
- **Lighthouse**: `lighthouserc.js` 配置了 Lighthouse CI

### 19.5 工具脚本（scripts/）

| 脚本 | 职责 |
|------|------|
| `health-check.js` | 项目健康检查 |
| `quality-check.js` | 代码质量检查 |
| `final-validation.js` | 最终验证 |
| `dev-tools.js` | 开发工具集 |
| `audit-theme-performance.js` | 主题性能审计 |
| `check-page-data-budget.js` | 页面数据预算检查 |
| `perf-baseline.js` | 性能基准测试 |
| `compress-theme-previews.js` | 主题预览图压缩 |
| `setup-git-hooks.js` | Git hooks 设置 |
| `translate/` | 多语言翻译管线 |

### 19.6 文档站点

项目包含 VitePress 文档站点（`.vitepress/` + `docs/`），可通过以下命令启动：

```bash
# 安装文档依赖后
npx vitepress dev docs
```

---

## 附录 A：关键设计模式

| 模式 | 应用场景 | 说明 |
|------|----------|------|
| 单例模式 | Notion 客户端 | `globalStore.notion` 全局唯一 |
| 链式缓存 | Redis→文件→内存 | 按优先级逐级尝试 |
| 配置驱动 | 插件/组件加载 | `siteConfig()` 读取配置决定加载 |
| 动态导入 | 主题/组件 | `next/dynamic` 按需加载 |
| 观察者模式 | 图片懒加载 | `IntersectionObserver` |
| 限流器 | Notion API | 滑动窗口 + 文件锁 |
| 防抖/节流 | 搜索/滚动 | `debounce` / `throttle` |
| 事件委托 | 路由变化 | `routeChangeStart/Complete` |
| 状态提升 | 全局状态 | `GlobalContextProvider` |

## 附录 B：环境变量速查

详见 [`.env.example`](.env.example)（约 100+ 个环境变量），关键必填项：

```bash
NOTION_PAGE_ID=xxx           # Notion 页面 ID（必填）
API_BASE_URL=https://...     # Notion API 地址（可选）
NEXT_PUBLIC_THEME=simple     # 主题（可选）
NEXT_PUBLIC_LANG=zh-CN       # 语言（可选）
NEXT_PUBLIC_AUTHOR=xxx       # 作者（可选）
```

## 附录 C：TypeScript 路径别名

```json
{
  "@/*": ".",
  "@/blog.config": "./blog.config",
  "@/components/*": "./components/*",
  "@/hooks/*": "./hooks/*",
  "@/themes/*": "./themes/*",
  "@/pages/*": "./pages/*",
  "@/data/*": "./data/*",
  "@/lib/*": "./lib/*",
  "@/styles/*": "./styles/*"
}
```

---

> 本文档由代码分析自动生成，基于 NotionNext v4.10.6 版本。
