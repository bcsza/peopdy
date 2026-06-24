# HyperLink Aggregator

HyperLink Aggregator 是一个面向技术调研者、数据采集工程师与 SEO 策略分析师的轻量级外链资源归集与健康度监测工具。该项目不生产内容，不存储媒体文件，仅提供结构化的外部视频与影视类域名索引，并辅以可配置的可用性探测与访问状态记录。

该项目主要解决多源外链分散、人工记录易错、批量检测效率低的问题。用户可通过本项目提供的统一清单与配套脚本，快速获取指定批次的域名集合，并进行连通性验证、响应时间统计与备案信息查询，从而显著提升链路管理效率。

## 功能概览

**批量域名导入** 支持通过配置文件或命令行参数一次性载入当前批次的全部外链地址，自动解析协议与主机名。

**可用性探测引擎** 内置轻量级 HTTP/HTTPS 探针，支持超时设置、重试策略与状态码判定，可区分有效响应、连接拒绝、超时及 SSL 证书错误。

**响应数据记录** 每次探测结果以结构化格式（JSON Lines）写入本地日志，包含时间戳、目标 URL、响应码、响应耗时与异常类型。

**差异比对工具** 提供脚本对比当前批次与前一批次（43/48）的域名集合，输出新增、删除与变更条目，便于追踪资源变动。

**定时任务集成** 提供示例 Crontab 配置，支持每日凌晨自动执行探测任务，并将汇总报告输出至指定目录。

**纯静态报告生成** 基于日志数据生成 HTML 格式的可用性概览看板，包含饼图与条形图，无需额外后端服务。

**配置热加载** 修改域名清单后无需重启主进程，通过信号触发重新载入配置，适用于高频更新场景。

**多协议回退** 当 HTTPS 请求失败时自动尝试 HTTP 连接，并记录协议降级情况，提高探测成功率。

## 应用场景

媒体资源调研项目中的外链存活审计。调研团队每月需对数百个影视类域名进行可用性抽检，人工点击效率极低。使用 HyperLink Aggregator 可将域名清单导入，自动生成存活率报告，将原本 3 人/天的工作压缩至 10 分钟内完成。

数据采集管道的前置健康检查。采集调度系统在启动任务前，需要确认目标域名可访问。本项目可作为独立 Sidecar 容器运行，对外暴露健康检查 API，供上游调度器调用，避免采集任务因网络不可达而失败。

SEO 外链质量评估流程。SEO 专员定期导出友链与外部引用域名，使用本工具批量检测响应状态与重定向链，识别失效或异常跳转链接，及时清理低质量外链，维护站点权重。

## 快速开始

```bash
# 克隆仓库
git clone https://github.com/example/hyperlink-aggregator.git
cd hyperlink-aggregator

# 安装依赖（Python 3.9+）
pip install -r requirements.txt

# 运行批量探测（使用默认配置 /config/batch_44.json）
python src/main.py --config config/batch_44.json --output reports/
```

## 安装要求

| 依赖 | 必需 | 说明 |
|------|------|------|
| Python 3.9+ | 是 | 核心运行环境，低于此版本将导致类型注解解析失败 |
| requests 2.28+ | 是 | HTTP 请求库，用于执行连通性探测与重定向跟踪 |
| pydantic 2.0+ | 是 | 配置模型校验与序列化，保证输入数据格式正确 |
| pytest 7.0+ | 否 | 仅开发测试需要，生产环境可不安装 |
| loguru 0.7+ | 否 | 可选结构化日志输出，不安装则回退至标准 logging |
| jinja2 3.0+ | 否 | 生成 HTML 报告时需要，若不安装则仅输出 JSON 格式 |

## 文档导航

| 层面 | 目录 | 回答的问题 |
|------|------|------------|
| 用户手册 | docs/user_guide.md | 如何安装、配置清单、运行探测、解读报告 |
| 配置参考 | docs/config_reference.md | 配置文件内每个字段的含义与合法取值 |
| 开发指引 | docs/development.md | 如何扩展探测器、添加新输出格式、提交代码 |
| 运维部署 | docs/operations.md | 如何设置定时任务、日志轮转、监控告警阈值 |

## 资源列表

当前批次域名清单（第 44/48 批）

影视与流媒体类

https://www.hyfilm.skin
https://www.lanhutv.site
https://www.liminyy.shop
https://www.liyavideo.autos
https://www.meitianfilm.lol

## 项目结构

```
hyperlink-aggregator/
├── config/                              # 配置文件目录
│   ├── batch_44.json                   # 第44批域名清单（当前默认）
│   ├── batch_43.json                   # 第43批历史清单（用于对比）
│   └── schema.json                     # 配置 JSON Schema 校验文件
├── src/                                 # 核心源码目录
│   ├── main.py                         # 程序入口，解析命令行参数
│   ├── probe/                          # 探测引擎子模块
│   │   ├── http_probe.py               # HTTP/HTTPS 请求实现，含重试与超时
│   │   ├── ssl_checker.py              # SSL 证书有效期与颁发者提取
│   │   └── result_parser.py            # 解析响应头与正文特征
│   ├── storage/                        # 存储与记录子模块
│   │   ├── logger.py                   # 结构化日志写入，支持 JSON Lines 格式
│   │   └── report_builder.py           # 聚合日志生成 HTML/JSON 报告
│   ├── comparator/                     # 差异比对子模块
│   │   └── diff_tool.py                # 对比两个批次域名集合，输出变更集
│   └── utils/                          # 通用工具函数
│       ├── config_loader.py            # 加载 JSON 配置，支持环境变量替换
│       └── validator.py                # 域名格式校验与归一化辅助
├── tests/                               # 单元测试与集成测试目录
│   ├── test_probe.py                   # 探测引擎各类场景的 mock 测试
│   ├── test_storage.py                 # 日志写入与报告生成测试
│   └── fixtures/                       # 测试用的样例配置与预期结果
├── scripts/                             # 运维辅助脚本
│   ├── cron_daily.sh                   # 每日定时任务入口，调用 main.py
│   └── rotate_logs.sh                  # 日志归档与清理（保留最近30天）
├── reports/                             # 默认输出目录（运行时生成）
│   ├── summary.json                    # 最新一次探测汇总数据
│   └── dashboard.html                  # 可视化看板（若 jinja2 已安装）
├── requirements.txt                     # 生产环境依赖列表
├── requirements-dev.txt                 # 开发环境额外依赖（测试、代码检查）
├── Dockerfile                           # 容器化构建文件，基于 alpine 镜像
├── .github/                             # GitHub 工作流配置
│   └── workflows/                      # CI 流水线（运行测试与构建镜像）
│       └── ci.yml
├── LICENSE                              # MIT 许可证文件
└── README.md                            # 本文档
```

## 贡献指南

1. 阅读开发指引文档（docs/development.md）了解代码风格、测试规范与提交信息格式要求。所有提交必须通过 pre-commit 钩子中的 black 与 flake8 检查。

2. 在 issues 列表中认领未分配的任务，或创建新 issue 描述你发现的问题或希望新增的功能。重大变更需先通过 issue 讨论，避免无效工作。

3. 从 main 分支创建功能分支（命名格式为 feature/描述 或 fix/描述），开发完成后提交 pull request，并确保所有 CI 检查（单元测试、构建镜像）通过。至少需要一位维护者批准后方可合并。

4. 若新增探测协议或输出格式，需同步更新配置参考文档与用户手册，并在 pull request 描述中注明文档变更位置。

## 常见问题

Q: 探测时遇到 SSL 证书验证失败，但浏览器可以正常访问，如何处理？

A: 项目默认进行证书验证。若确认为自签名或过期证书，可在配置文件中将 verify_ssl 设为 false 以跳过验证。但不建议在生产环境关闭，建议将证书文件路径填入 ca_bundle 字段。

Q: 报告中的响应时间为负值或显示超时，是什么原因？

A: 负值通常表示请求被本地网络栈拒绝（如连接数耗尽），超时则代表服务器在设定时间内无响应。可调整配置中的 connect_timeout 和 read_timeout 参数，分别控制连接建立与数据读取的最大等待秒数。若大量超时，建议检查网络出口或目标服务器状态。

Q: 如何扩展探测目标到非 HTTP 协议（如 ping 或 TCP 端口检测）？

A: 项目设计采用策略模式，探测器基类位于 src/probe/base_probe.py。您可继承该基类实现 ping_probe 或 tcp_probe，并重写 check() 方法。随后在 main.py 中注册新探测器类型即可。详细示例参考开发指引中的扩展章节。

## 许可证

MIT

> 外链数量: 5 | 生成时间: 2026-06-24 16:38:18
