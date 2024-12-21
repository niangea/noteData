### **ELK 整合步骤**

#### **1. 准备 ELK 环境**

##### **1.1 安装 Elasticsearch**

- 下载 Elasticsearch：https://www.elastic.co/downloads/elasticsearch

- 解压后配置 `elasticsearch.yml` 文件（如需要修改默认配置）。

- 启动 Elasticsearch：

  bash

  复制

  ```
  ./bin/elasticsearch
  ```

##### **1.2 安装 Logstash**

- 下载 Logstash：https://www.elastic.co/downloads/logstash
- 解压后准备 Logstash 配置文件（详见下文）。

##### **1.3 安装 Kibana**

- 下载 Kibana：https://www.elastic.co/downloads/kibana

- 解压后配置 `kibana.yml` 文件（如需要修改默认配置）。

- 启动 Kibana：

  bash

  复制

  ```
  ./bin/kibana
  ```

##### **1.4 验证 ELK 是否启动成功**

- Elasticsearch 默认运行在 `http://localhost:9200`。
- Kibana 默认运行在 `http://localhost:5601`。

------

#### **2. Spring Boot 项目日志输出到 Logstash**

##### **2.1 添加依赖**

在 `pom.xml` 中添加以下依赖：

xml

复制

```
<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Logback-Logstash 依赖，用于将日志发送到 Logstash -->
    <dependency>
        <groupId>net.logstash.logback</groupId>
        <artifactId>logstash-logback-encoder</artifactId>
        <version>7.4</version>
    </dependency>

    <!-- Spring Boot Actuator (可选，用于监控) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

------

##### **2.2 配置 Logback（日志框架）**

在 `src/main/resources` 目录下创建 `logback-spring.xml` 文件，配置 Logback 将日志发送到 Logstash。

xml

复制

```
<configuration>

    <!-- 控制台日志输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Logstash 日志输出（通过 Logstash TCP） -->
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:5000</destination> <!-- Logstash 的地址和端口 -->
        <encoder class="net.logstash.logback.encoder.LogstashEncoder" />
    </appender>

    <!-- 日志级别设置 -->
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="LOGSTASH" />
    </root>

</configuration>
```

------

#### **3. 配置 Logstash**

##### **3.1 创建 Logstash 配置文件**

在 Logstash 安装目录下的 `config` 文件夹中，创建一个配置文件 `logstash.conf`，配置 Logstash 以接收日志并发送到 Elasticsearch。

plaintext

复制

```
input {
    tcp {
        port => 5000 # 监听的端口，与 logback-spring.xml 中的端口对应
        codec => json # 接收 JSON 格式数据
    }
}

filter {
    # 这里可以添加过滤规则，例如解析日志内容
}

output {
    elasticsearch {
        hosts => ["http://localhost:9200"] # Elasticsearch 的地址
        index => "springboot-logs-%{+YYYY.MM.dd}" # 日志索引名称
    }

    stdout {
        codec => rubydebug # 在 Logstash 控制台打印日志，调试用
    }
}
```

##### **3.2 启动 Logstash**

运行以下命令启动 Logstash：

bash

复制

```
./bin/logstash -f config/logstash.conf
```

------

#### **4. 配置 Kibana**

##### **4.1 设置索引模式**

- 打开 Kibana：`http://localhost:5601`

- 进入 

  Management > Index Patterns

  ，创建索引模式：

  - 输入索引模式：`springboot-logs-*`
  - 选择时间字段（如 `@timestamp`）。

##### **4.2 可视化日志数据**

- 在 **Discover** 页面中，可以查看实时日志数据。
- 在 **Dashboard** 页面中，可以创建可视化面板。

------

#### **5. 验证日志收集流程**

##### **5.1 创建简单的 Spring Boot 控制器**

创建一个简单的控制器，在访问时输出日志：

java

复制

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class LogController {

    private static final Logger logger = LoggerFactory.getLogger(LogController.class);

    @GetMapping("/log")
    public String log() {
        logger.info("This is an INFO log message.");
        logger.warn("This is a WARN log message.");
        logger.error("This is an ERROR log message.");
        return "Logs have been sent!";
    }
}
```

##### **5.2 启动应用并访问接口**

- 启动 Spring Boot 应用。
- 访问接口：`http://localhost:8080/log`。
- 查看以下日志传递情况：
  1. 控制台输出日志。
  2. Logstash 控制台输出日志。
  3. 在 Kibana 的 **Discover** 页面中查看日志。

------

### **进阶功能**

#### **1. 添加更多日志字段**

可以在 Logback 的配置中添加自定义字段，例如应用名、环境等：

xml

复制

```
<encoder class="net.logstash.logback.encoder.LogstashEncoder">
    <customFields>{"app_name":"springboot-app","environment":"production"}</customFields>
</encoder>
```

#### **2. 过滤日志**

在 Logstash 的 `filter` 部分，可以添加日志过滤规则，例如仅处理特定级别的日志：

plaintext

复制

```
filter {
    if [level] == "INFO" {
        drop {} # 丢弃 INFO 级别的日志
    }
}
```

#### **3. 使用 Filebeat 替代直接传输**

如果不希望 Spring Boot 直接将日志发送到 Logstash，可以将日志存储到文件中，然后通过 Filebeat 将日志文件传输到 Logstash。

Filebeat 配置示例（`filebeat.yml`）：

yaml

复制

```
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /path/to/logfile.log

output.logstash:
  hosts: ["localhost:5000"]
```

------

### **总结**

通过以上步骤，Spring Boot 日志将被整合到 ELK 中，完整流程如下：

1. Spring Boot 使用 Logback 生成日志。
2. 日志通过 LogstashAppender 发送到 Logstash。
3. Logstash 处理日志并存储到 Elasticsearch。
4. Kibana 提供日志的可视化和检索。

该方案可以轻松实现日志的集中化管理和分析，适合生产环境中应用的日志监控需求。如果有更复杂的需求（如分布式日志收集），可以进一步扩展方案，结合 Filebeat、Kafka 等工具。