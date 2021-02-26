# Lombok

## 辅助Pojo

可以辅助pojo快速开发get、set方法





## 辅助日志

`@Slf4j`注解：相当于

```java
private static final Logger log = LoggerFactory.getLogger(LogConfig.class);
```

可以使用 `log` 对象记录不同级别的日志，根据日志配置文件进行输出。

```java
log.debug("debug 级别输出");
log.info("info 级别输出");
log.warn("warn 级别输出");
log.error("error 级别输出");
```

