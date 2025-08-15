# KaiyuanTV

<div align="center">
  <img src="public/logo.png" alt="LibreTV Logo" width="120">
</div>

> 🎬 **KaiyuanTV** 是一个开箱即用的、跨平台的影视聚合播放器。它基于 **Next.js 14** + **Tailwind&nbsp;CSS** + **TypeScript** 构建，支持多资源搜索、在线播放、收藏同步、播放记录、本地/云端存储，让你可以随时随地畅享海量免费影视内容。

<div align="center">

![Next.js](https://img.shields.io/badge/Next.js-14-000?logo=nextdotjs)
![TailwindCSS](https://img.shields.io/badge/TailwindCSS-3-38bdf8?logo=tailwindcss)
![TypeScript](https://img.shields.io/badge/TypeScript-4.x-3178c6?logo=typescript)
![License](https://img.shields.io/badge/License-MIT-green)
![Docker Ready](https://img.shields.io/badge/Docker-ready-blue?logo=docker)

</div>



<details>
  <summary>点击查看项目截图</summary>
  <img src="public/screenshot1.png" alt="项目截图" style="max-width:600px">
  <img src="public/screenshot2.png" alt="项目截图" style="max-width:600px">
  <img src="public/screenshot3.png" alt="项目截图" style="max-width:600px">
</details>




### Cloudflare 部署

**Cloudflare Pages 的环境变量尽量设置为密钥而非文本**

1. **Fork** 本仓库到你的 GitHub 账户。
2. 登陆 [Cloudflare](https://cloudflare.com)，点击 **计算（Workers）-> Workers 和 Pages**，点击创建
3. 选择 Pages，导入现有的 Git 存储库，选择 Fork 后的仓库
4. 构建命令填写 **pnpm install --frozen-lockfile && pnpm run pages:build**，预设框架为无，**构建输出目录**为 `.vercel/output/static`
5. 保持默认设置完成首次部署。进入设置，将兼容性标志设置为 `nodejs_compat`，无需选择，直接粘贴
6. 首次部署完成后进入设置，新增 PASSWORD 密钥（变量和机密下），而后重试部署。
7. 如需自定义 `config.json`，请直接修改 Fork 后仓库中该文件。
8. 每次 Push 到 `main` 分支将自动触发重新构建。

#### D1 支持

0. 完成普通部署并成功访问
1. 点击 **存储和数据库 -> D1 SQL 数据库**，创建一个新的数据库，名称随意
2. 进入刚创建的数据库，点击左上角的 Explore Data，将[D1 初始化](D1初始化.md) 中的内容粘贴到 Query 窗口后点击 **Run All**，等待运行完成
3. 返回你的 pages 项目，进入 **设置 -> 绑定**，添加绑定 D1 数据库，选择你刚创建的数据库，变量名称填 **DB**
4. 设置环境变量 NEXT_PUBLIC_STORAGE_TYPE，值为 **d1**；设置 USERNAME 和 PASSWORD 作为站长账号
5. 重试部署

### Docker 部署

#### 1. 直接运行（最简单，localstorage）

```bash
# 拉取预构建镜像
docker pull ghcr.io/senshinya/KaiyuanTV:latest

# 运行容器
# -d: 后台运行  -p: 映射端口 3000 -> 3000
docker run -d --name KaiyuanTV -p 3000:3000 --env PASSWORD=your_password ghcr.io/senshinya/KaiyuanTV:latest
```

访问 `http://服务器 IP:3000` 即可。（需自行到服务器控制台放通 `3000` 端口）

## Docker Compose 最佳实践

若你使用 docker compose 部署，以下是一些 compose 示例

### local storage 版本

```yaml
services:
  KaiyuanTV:
    image: ghcr.io/senshinya/KaiyuanTV:latest
    container_name: KaiyuanTV
    restart: unless-stopped
    ports:
      - '3000:3000'
    environment:
      - PASSWORD=your_password
    # 如需自定义配置，可挂载文件
    # volumes:
    #   - ./config.json:/app/config.json:ro
```

### Redis 版本（推荐，多账户数据隔离，跨设备同步）

```yaml
services:
  KaiyuanTV-core:
    image: ghcr.io/senshinya/KaiyuanTV:latest
    container_name: KaiyuanTV
    restart: unless-stopped
    ports:
      - '3000:3000'
    environment:
      - USERNAME=admin
      - PASSWORD=admin_password
      - NEXT_PUBLIC_STORAGE_TYPE=redis
      - REDIS_URL=redis://KaiyuanTV-redis:6379
      - NEXT_PUBLIC_ENABLE_REGISTER=true
    networks:
      - KaiyuanTV-network
    depends_on:
      - KaiyuanTV-redis
    # 如需自定义配置，可挂载文件
    # volumes:
    #   - ./config.json:/app/config.json:ro
  KaiyuanTV-redis:
    image: redis
    container_name: KaiyuanTV-redis
    restart: unless-stopped
    networks:
      - KaiyuanTV-network
    # 如需持久化
    # volumes:
    #   - ./data:/data
networks:
  KaiyuanTV-network:
    driver: bridge
```

## 环境变量

| 变量                              | 说明                                         | 可选值                           | 默认值                                                                                                                     |
| --------------------------------- | -------------------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| USERNAME                          | 非 localstorage 部署时的管理员账号           | 任意字符串                       | （空）                                                                                                                     |
| PASSWORD                          | 非 localstorage 部署时为管理员密码           | 任意字符串                       | （空）                                                                                                                     |
| SITE_NAME                         | 站点名称                                     | 任意字符串                       | KaiyuanTV                                                                                                                     |
| ANNOUNCEMENT                      | 站点公告                                     | 任意字符串                       | 本网站仅提供影视信息搜索服务，所有内容均来自第三方网站。本站不存储任何视频资源，不对任何内容的准确性、合法性、完整性负责。 |
| NEXT_PUBLIC_STORAGE_TYPE          | 播放记录/收藏的存储方式                      | localstorage、redis、d1、upstash | localstorage                                                                                                               |
| REDIS_URL                         | redis 连接 url                               | 连接 url                         | 空                                                                                                                         |
| UPSTASH_URL                       | upstash redis 连接 url                       | 连接 url                         | 空                                                                                                                         |
| UPSTASH_TOKEN                     | upstash redis 连接 token                     | 连接 token                       | 空                                                                                                                         |
| NEXT_PUBLIC_ENABLE_REGISTER       | 是否开放注册，仅在非 localstorage 部署时生效 | true / false                     | false                                                                                                                      |
| NEXT_PUBLIC_SEARCH_MAX_PAGE       | 搜索接口可拉取的最大页数                     | 1-50                             | 5                                                                                                                          |
| NEXT_PUBLIC_IMAGE_PROXY           | 默认的浏览器端图片代理                       | url prefix                       | (空)                                                                                                                       |
| NEXT_PUBLIC_DOUBAN_PROXY          | 默认的浏览器端豆瓣数据代理                   | url prefix                       | (空)                                                                                                                       |
| NEXT_PUBLIC_DISABLE_YELLOW_FILTER | 关闭色情内容过滤                             | true/false                       | false                                                                                                                      |

## 配置说明

所有可自定义项集中在根目录的 `config.json` 中：

```json
{
  "cache_time": 7200,
  "api_site": {
    "dyttzy": {
      "api": "http://caiji.dyttzyapi.com/api.php/provide/vod",
      "name": "电影天堂资源",
      "detail": "http://caiji.dyttzyapi.com"
    }
    // ...更多站点
  },
  "custom_category": [
    {
      "name": "华语",
      "type": "movie",
      "query": "华语"
    }
  ]
}
```

- `cache_time`：接口缓存时间（秒）。
- `api_site`：你可以增删或替换任何资源站，字段说明：
  - `key`：唯一标识，保持小写字母/数字。
  - `api`：资源站提供的 `vod` JSON API 根地址。
  - `name`：在人机界面中展示的名称。
  - `detail`：（可选）部分无法通过 API 获取剧集详情的站点，需要提供网页详情根 URL，用于爬取。
- `custom_category`：自定义分类配置，用于在导航中添加个性化的影视分类。以 type + query 作为唯一标识。支持以下字段：
  - `name`：分类显示名称（可选，如不提供则使用 query 作为显示名）
  - `type`：分类类型，支持 `movie`（电影）或 `tv`（电视剧）
  - `query`：搜索关键词，用于在豆瓣 API 中搜索相关内容

custom_category 支持的自定义分类已知如下：

- movie：热门、最新、经典、豆瓣高分、冷门佳片、华语、欧美、韩国、日本、动作、喜剧、爱情、科幻、悬疑、恐怖、治愈
- tv：热门、美剧、英剧、韩剧、日剧、国产剧、港剧、日本动画、综艺、纪录片

也可输入如 "哈利波特" 效果等同于豆瓣搜索

KaiyuanTV 支持标准的苹果 CMS V10 API 格式。

修改后 **无需重新构建**，服务会在启动时读取一次。

## 管理员配置

**该特性目前仅支持通过非 localstorage 存储的部署方式使用**

支持在运行时动态变更服务配置

设置环境变量 USERNAME 和 PASSWORD 即为站长用户，站长可设置用户为管理员

站长或管理员访问 `/admin` 即可进行管理员配置

## AndroidTV 使用

目前该项目可以配合 [OrionTV](https://github.com/zimplexing/OrionTV) 在 Android TV 上使用，可以直接作为 OrionTV 后端

暂时收藏夹与播放记录和网页端隔离，后续会支持同步用户数据

## Roadmap

- [x] 深色模式
- [x] 持久化存储
- [x] 多账户

## 安全与隐私提醒
