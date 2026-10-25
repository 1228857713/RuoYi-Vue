# 本地运行

本仓库使用 JDK 8、Spring Boot 2.5.15 和 Maven，产物为 `ruoyi-admin/target/ruoyi-admin.jar`。命令均在后端仓库根目录执行。

## 依赖

- 使用 JDK 8，并通过 `mvn -version` 确认 Maven 使用同一 JDK。[Boot 2.5.15 官方系统要求](https://docs.spring.io/spring-boot/docs/2.5.15/reference/html/getting-started.html#getting-started.system-requirements)列出 Maven 3.5+。
- [若依官方环境说明](https://doc.ruoyi.vip/ruoyi-vue/document/hjbs.html#准备工作)列出 MySQL 5.7+、Redis 3.0+。准备独立的本地开发库，实际版本与当前 SQL 的兼容性需完成初始化和运行验收。
- 初始化前核对数据库目标和现有表，审阅仓库 `sql/` 下的业务与调度脚本；仅向已授权的开发库导入。登录凭据在本地初始化后设置和保管。

## 外部本机配置

将本机 YAML 配置放在仓库外，并用 `RUOYI_LOCAL_CONFIG` 指向该文件。以下是配置结构示例，所有连接信息和秘密由运行环境提供：

```yaml
server:
  address: 127.0.0.1
  port: ${RUOYI_SERVER_PORT}
ruoyi:
  profile: ${RUOYI_UPLOAD_DIR}
spring:
  datasource:
    druid:
      master:
        url: ${RUOYI_DB_URL}
        username: ${RUOYI_DB_USERNAME}
        password: ${RUOYI_DB_PASSWORD}
  redis:
    host: ${RUOYI_REDIS_HOST}
    port: ${RUOYI_REDIS_PORT}
    database: ${RUOYI_REDIS_DATABASE}
    password: ${RUOYI_REDIS_PASSWORD}
token:
  secret: ${RUOYI_TOKEN_SECRET}
```

通过终端、IDE 运行配置或本地服务管理器向应用进程注入上述环境变量；`RUOYI_DB_URL` 是完整 JDBC URL，`RUOYI_UPLOAD_DIR` 是可写的上传目录。建议为独立开发实例启用认证；密码由本机环境提供。使用独立的令牌签名秘密，保持配置文件仅由必要的本机用户读取。

上游 `logback.xml` 在 Spring 配置加载前可能尝试写入固定 Linux 日志目录。macOS 或无该目录权限的环境，需准备仓库外的 Logback XML，将日志目录改为本机可写目录，并让 `RUOYI_LOGBACK_CONFIG` 指向它。启动时同时设置 JVM 的 `logback.configurationFile`（早期初始化）与 `logging.config`（Spring Boot 初始化）。真实配置、运行日志和数据文件均不提交。

## 构建与启动

```sh
java -version
mvn -version
mvn -pl ruoyi-admin -am clean package
java -Dlogback.configurationFile="${RUOYI_LOGBACK_CONFIG}" \
  -Dlogging.config="file:${RUOYI_LOGBACK_CONFIG}" \
  -jar ruoyi-admin/target/ruoyi-admin.jar \
  --spring.config.additional-location="file:${RUOYI_LOCAL_CONFIG}"
```

`RUOYI_LOCAL_CONFIG` 必须是实际存在的配置文件路径。`spring.config.additional-location` 保留打包配置并加载外部覆盖项，详见 [Boot 2.5 外部配置文档](https://docs.spring.io/spring-boot/docs/2.5.15/reference/html/features.html#features.external-config.files)。命令中的变量是运行配置约定，不需要修改已提交的 YAML。

以前台方式启动后，使用 Ctrl-C 停止该进程。记录实际端口，联调时保持前端代理目标一致。

## 验收

分别记录构建结果与运行结果：确认数据库和 Redis 连接，验证当前配置下的验证码、登录、菜单、权限拦截及一个列表查询。需要写入时仅用可清理的开发数据。测试未执行、被跳过或数量为零时如实注明。

Spring Boot 2.x 已结束上游开源支持，本模板的 JDK 8 约束不代表已解决依赖长期维护风险。面向生产发布前须单独评估补丁、支持和升级方案。[Spring 官方说明](https://spring.io/blog/2023/11/23/spring-boot-2-7-18-available-now)
