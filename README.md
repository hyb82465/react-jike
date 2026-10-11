# react-jike-2024 · 极客园文章管理后台

一个跟随课程完成的 React 后台管理练习项目。项目围绕内容创作者的文章管理流程，实现了登录、文章列表、筛选分页、富文本编辑、封面上传以及文章发布、编辑和删除。

**本仓库只包含前端。** 用户、频道和文章数据来自代码中配置的课程接口；仓库不包含后端服务、数据库或本地 Mock。以下说明依据现有代码整理。

## 功能

- **登录与退出**：手机号、验证码表单校验；登录后保存 Token，获取用户信息并显示用户名，支持退出确认。
- **路由鉴权**：未持有 Token 时跳转登录页；Axios 自动携带 Bearer Token，并对 HTTP 401 做清理与跳转处理。
- **首页图表**：使用 ECharts 和封装的 `BarChart` 展示三组柱状图，数据为代码内的静态示例。
- **文章列表**：展示封面、标题、审核状态、发布时间及阅读、评论、点赞数；支持频道、状态、日期筛选和分页。
- **文章发布**：填写标题、选择频道，使用 React Quill 编辑富文本；支持无图、单图、三图封面和图片上传。
- **文章编辑与删除**：按文章 ID 加载并回填表单，提交更新；删除前弹出确认框。

> 文章相关功能依赖远程接口可用及有效登录凭据。当前发布和更新请求均使用 `draft=false`，没有单独的草稿保存入口；审核状态由接口返回，项目未实现审核操作。

## 技术栈

| 用途 | 技术 |
| --- | --- |
| UI 与路由 | React 18、React Router 6 |
| 状态管理 | Redux Toolkit 2、React Redux 9 |
| 组件与样式 | Ant Design 5、Sass、normalize.css |
| 网络请求 | Axios |
| 内容编辑 | React Quill 2 |
| 图表 | ECharts 5 |
| 构建配置 | Create React App / react-scripts 5、CRACO 7 |

`craco.config.js` 将 `@` 配置为 `src` 路径别名。具体依赖版本以 `package.json` 和 `package-lock.json` 为准。

## 页面路由

| 路径 | 页面 |
| --- | --- |
| `/login` | 手机号、验证码登录 |
| `/` | 后台布局与图表示例首页 |
| `/article` | 文章列表和筛选 |
| `/publish` | 新建文章 |
| `/publish?id=<文章ID>` | 编辑已有文章，复用发布页面 |

除登录页外，页面均位于 `AuthRoute` 包裹的后台布局下。

## 目录结构

```text
react-jike-2024/
├── public/
├── src/
│   ├── apis/                 # 用户、文章和频道 API 封装
│   ├── assets/               # 登录背景、Logo、占位图片等
│   ├── components/AuthRoute.js
│   ├── hooks/useChannel.js   # 获取频道列表的自定义 Hook
│   ├── pages/
│   │   ├── Home/             # 首页与 BarChart
│   │   ├── Layout/           # 导航、用户名、退出与子路由出口
│   │   ├── Login/            # 登录表单
│   │   ├── Article/          # 文章列表
│   │   └── Publish/          # 发布与编辑文章
│   ├── router/               # 浏览器路由配置
│   ├── store/                # Redux store 与用户状态
│   ├── utils/                # Axios 实例、Token 存取
│   └── index.js              # 应用入口
├── craco.config.js
├── jsconfig.json
└── package.json
```

## 本地运行

需要 Node.js 和 npm。仓库未指定 Node.js 版本；旧依赖在不同 Node.js 版本下的兼容性需要自行验证。

```bash
git clone https://github.com/hyb82465/react-jike-2024.git
cd react-jike-2024
npm ci
npm start
```

开发服务默认地址为 <http://localhost:3000>。未登录时会跳转到 `/login`。

登录需要课程接口接受的手机号和验证码。仓库没有提供可确认有效的测试账户或固定验证码，也没有实现发送验证码按钮；请使用课程服务提供的有效凭据。

## 接口配置与外部依赖

当前代码直接写入了两个地址，并未从 `.env` 读取：

| 配置位置 | 当前地址 |
| --- | --- |
| `src/utils/request.js` 中的 `baseURL` | `http://geek.itheima.net/v1_0` |
| `src/pages/Publish/index.js` 中的 Upload `action` | `http://geek.itheima.net/v1_0/upload` |

Axios 请求超时为 5 秒，Token 存在 localStorage 的 `token_key` 中。更换 API 服务时需要检查上述**两处**配置，并保证接口路径、认证方式和返回数据结构与现有代码兼容。

主要接口如下（相对于 `baseURL`）：

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| POST | `/authorizations` | 登录 |
| GET | `/user/profile` | 获取用户信息 |
| GET | `/channels` | 获取频道列表 |
| GET | `/mp/articles` | 查询文章，传入筛选和分页参数 |
| GET | `/mp/articles/:id` | 获取文章详情 |
| POST | `/mp/articles?draft=false` | 发布文章 |
| PUT | `/mp/articles/:id?draft=false` | 更新文章 |
| DELETE | `/mp/articles/:id` | 删除文章 |
| POST | `/upload` | 上传封面，表单字段为 `image` |

图片上传由 Ant Design Upload 直接发送，不经过封装的 Axios 实例。如果新服务要求上传认证，需另行配置上传请求头。

## 常用命令与构建

| 命令 | 作用 |
| --- | --- |
| `npm start` | 通过 CRACO 启动开发服务 |
| `npm run build` | 构建静态文件到 `build/` |
| `npm test` | 启动测试运行器；当前仓库没有业务测试用例 |

部署静态构建产物时，服务器需要将前端路由回退到 `index.html`，以支持直接访问或刷新 `/article`、`/publish` 等路径。当前 API 使用 HTTP，若前端部署为 HTTPS，需要先解决接口协议与混合内容问题。

## 已知限制与复习提示

- 远程课程服务的当前可用性和登录凭据未验证；前端能启动并不代表登录、上传和文章操作可用。
- 筛选代码直接读取日期区间的两个值；当前使用时需先选择日期范围，否则可能报错。
- 发布和更新操作没有等待接口完成就跳转到列表，尚缺少提交完成确认和完整错误反馈。
- 网络错误处理直接访问 `error.response.status`；无响应的网络错误可能触发额外异常。
- 首页图表是组件练习用示例，不是文章数据的实时统计。

本项目保留课程练习时的实现。README 补写时核对了源码、依赖和脚本，未执行依赖安装、构建或远程接口联调。具体课程名称和原始链接未在仓库中明确记录，因此不推断课程版本。
