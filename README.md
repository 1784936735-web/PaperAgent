# PaperAgent
PaperAgent is an AI-powered academic literature review tool built on Java 17, Javalin, and LangChain4j. It solves LLM token limits and hallucinations by using Apache PDFBox for smart PDF dimensionality reduction and Crossref API for real-time citation verification. Features include async J.U.C concurrency and zero-disk in-memory Word report exports
## 核心特性

* 异步并发与无阻塞调度
基于 java.util.concurrent (J.U.C) 线程池构建任务分发中心。将耗时的 PDF 文档解析与大模型网络请求剥离至后台物理线程执行，前端通过状态机轮询获取进度，彻底解决 Web 容器在处理大文件上传时易发生的阻塞假死问题。

* 突破大模型上下文限制的长文本降维
集成 Apache PDFBox 引擎，在服务端对长篇学术 PDF 进行物理切割。系统动态识别并提取包含摘要与引言的首页，以及包含参考文献的尾页，精准剔除中间过程冗余文本。在大幅降低 API Token 消耗的同时，保证推理速度与核心语料的完整性。

* 基于底层网络爬虫的防幻觉溯源机制
为彻底解决大语言模型伪造文献的“幻觉”现象，系统引入 LangChain4j 的工具回调（Tool Calling）机制。当模型涉及引文审查时，Java 后端通过原生 HttpClient 强制接管控制权，向 Crossref 全球 DOI 数据库发起验证请求，并配置 10 秒超时熔断策略以保障极端网络环境下的系统可用性。

* 面向对象的内存级免落地报表导出
基于 Apache POI 开发了零磁盘占用的导出引擎。系统在 JVM 堆内存中（ByteArrayOutputStream）直接进行数据流转与组装，自动按行遍历大模型输出结果，通过条件语句动态将验证失败的文献渲染为红色预警，验证成功的文献渲染为绿色，最终以纯字节流形式将 Word 审查报表下发至前端。

* 多租户数据沙箱与存储防冲突
系统采用 JWT (HMAC256) 实现无状态鉴权，结合 SQLite 的 PreparedStatement 预编译技术实现数据隔离与防 SQL 注入防御。物理文件存储底层调用 Java NIO 的 Files.copy 零拷贝技术，配合基于日期结构与 UUID 动态生成的文件名前缀，确保高并发环境下的资产安全。

## 技术栈选型

* 运行环境：OpenJDK 17
* 后端框架：Javalin 5.6.3
* 智能代理：LangChain4j 0.27.1, Moonshot (Kimi) API
* 前端架构：HTML5, 原生 JavaScript, TailwindCSS
* 数据存储与处理：SQLite, Apache POI 5.2.5, Apache PDFBox 2.0.30

## 快速启动指南

1. 环境依赖：请确保本地计算机已正确配置 JDK 17 及 Maven 构建工具。
2. 获取代码：执行命令 `git clone https://github.com/你的用户名/你的项目名.git` 克隆本仓库。
3. 构建项目：在项目根目录执行 `mvn clean install` 下载全部相关依赖。
4. 密钥配置：在源码配置处填入你申请的 Kimi API-Key。
5. 启动服务：运行 `PaperAssistant.java` 中的 `main` 方法，控制台提示启动成功后，通过浏览器访问 `http://localhost:8888` 即可进入系统主页。

## 开源协议

本项目基于 MIT License 协议开源。
