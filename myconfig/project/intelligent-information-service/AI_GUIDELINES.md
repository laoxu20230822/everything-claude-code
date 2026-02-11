# AI Guidelines - 智能信息服务平台

本文档为 AI 辅助开发提供关键版本约束和 API 验证指南，确保生成的代码与项目依赖版本一致，避免"幻觉"问题（如使用不兼容的 API）。

## 版本约束概览

| 依赖 | 版本 | 关键约束 |
|------|------|----------|
| Java | 17 | 支持现代语言特性（Records、Sealed Classes、Pattern Matching） |
| Spring Boot | 3.4.8 | **Jakarta EE namespace**（`javax.*` → `jakarta.*`） |
| Spring AI | 1.0.0 | 第一个稳定版本，API 已定型 |
| Spring AI Alibaba | 1.0.0.4 | 首个稳定版本，与 Spring AI 1.0.0 兼容 |
| MyBatis Plus | 3.5.6 | Spring Boot 3.x 专用版本 |
| SpringDoc | 2.7.0 | OpenAPI 3.0 规范 |

## 重大版本变更警示

### Jakarta EE Namespace 迁移（Spring Boot 2.x → 3.x）

**错误示例**（Spring Boot 2.x API）:
```java
import javax.servlet.http.HttpServletRequest;  // ❌ 错误！
import javax.persistence.Entity;             // ❌ 错误！
import javax.validation.constraints.NotNull;  // ❌ 错误！
```

**正确示例**（Spring Boot 3.x API）:
```java
import jakarta.servlet.http.HttpServletRequest;  // ✅ 正确
import jakarta.persistence.Entity;              // ✅ 正确
import jakarta.validation.constraints.NotNull;   // ✅ 正确
```

**常见需要替换的包**:
| 旧包（javax.*） | 新包（jakarta.*） |
|-----------------|-------------------|
| `javax.servlet.*` | `jakarta.servlet.*` |
| `javax.persistence.*` | `jakarta.persistence.*` |
| `javax.validation.*` | `jakarta.validation.*` |
| `javax.annotation.*` | `jakarta.annotation.*` |
| `javax.transaction.*` | `jakarta.transaction.*` |
| `javax.websocket.*` | `jakarta.websocket.*` |

### Spring AI 1.0.0 API 变更

Spring AI 1.0.0 是首个稳定版本，API 结构与早期 Milestone 版本有显著差异。

**关键 API 包路径**:
```java
// Spring AI 核心包
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatModel;
import org.springframework.ai.chat.prompt.ChatPrompt;
import org.springframework.ai.chat.messages.Message;

// Spring AI Alibaba 扩展
import com.alibaba.cloud.ai.dashscope.chat.DashScopeChatModel;
import com.alibaba.cloud.ai.dashscope.chat.DashScopeChatOptions;

// Graph 执行框架（项目核心）
import com.alibaba.cloud.ai.graph.StateGraph;
import com.alibaba.cloud.ai.graph.CompiledGraph;
import com.alibaba.cloud.ai.graph.NodeOutput;
import com.alibaba.cloud.ai.graph.OverAllState;
import com.alibaba.cloud.ai.graph.RunnableConfig;
```

## 文档验证协议

在生成涉及框架 API 的代码前，**必须**使用 Context7 MCP 工具获取版本特定的文档：

### 验证流程

1. **使用 Context7 查找版本特定文档**
   ```
   # 示例:验证 Spring Boot 3.4.x 的 API
   Create a REST controller with @RestController in Spring Boot 3.4.8. use context7
   ```

2. **Context7 自动获取最新文档**
   - Context7 会自动从源仓库获取匹配版本的文档
   - 无需手动搜索 URL

3. **验证 Spring AI 特定 API**
   ```
   # 搜索 Spring AI 1.0.0 文档
   Configure ChatClient with streaming in Spring AI 1.0.0. use context7
   ```

### 推荐的 Context7 库 ID

| 组件 | Context7 库 ID |
|------|----------------|
| Spring Boot | `/spring-projects/spring-boot` |
| Spring Framework | `/spring-projects/spring-framework` |
| Spring AI | `/spring-projects/spring-ai` |
| Jakarta EE | `/jakartaee/specifications` |
| MyBatis Plus | `/baomidou/mybatis-plus` |

**Context7 使用技巧**:
- 基本用法：在提示词末尾添加 `use context7`
- 指定库 ID：`use library /spring-projects/spring-boot for API and docs.`
- 指定版本：在提示词中提及版本号（如 "Spring Boot 3.4.8"）

## 代码验证工作流

### 生成代码前检查清单

在编写新代码或修改现有代码前，确认：

- [ ] **检查 Java 版本**: 确认使用 Java 17 特性（如 `record`、`sealed class`）
- [ ] **验证 Jakarta namespace**: 所有 `javax.*` 导入已替换为 `jakarta.*`
- [ ] **确认 Spring Boot 版本**: 使用 3.4.x API 而非 2.x API
- [ ] **验证依赖版本**: 检查 `pom.xml` 中的版本号
- [ ] **查阅官方文档**: 使用 Context7 MCP 工具获取版本特定文档

### 生成代码后检查清单

代码生成完成后：

- [ ] **编译检查**: 运行 `mvn clean compile -Dcheckstyle.skip=true -DskipTests -Dspring-javaformat.skip=true` 确保无编译错误
- [ ] **导入验证**: 检查所有导入语句是否使用正确的包名
- [ ] **API 版本兼容**: 确认使用的 API 在当前版本中存在
- [ ] **注解检查**: 确认注解来自正确的包（如 `@Entity` 来自 `jakarta.persistence`）

## MCP 工具使用指南

本项目集成了多个 MCP 工具，用于文档搜索和代码分析：

### Context7 - 版本特定文档查找工具

**用途**: 从源代码仓库获取最新的、版本特定的文档和代码示例

**使用方式**:

1. **基本用法** - 在提示词中添加 `use context7`:
   ```
   Create a Spring Boot 3.4 REST controller with streaming response. use context7
   ```

2. **指定库 ID** - 使用精确的库 ID 跳过匹配步骤:
   ```
   Create a Spring Boot 3.4 controller. use library /spring-projects/spring-boot for API and docs.
   ```

3. **指定版本** - 在提示词中提及版本号:
   ```
   How to configure Spring AI 1.0.0 ChatClient? use context7
   ```

**Context7 工具** (由 MCP 自动调用):
- `resolve-library-id`: 解析库名称为 Context7 库 ID
- `query-docs`: 使用库 ID 检索版本特定文档

### Serena - 代码分析工具

**用途**: 分析项目代码结构、查找符号引用、理解代码关系

**使用示例**:
```bash
# 查找特定类的使用
/mcp__serena__jet_brains_find_referencing_symbols "ChatController" "./src/main/java/com/alibaba/cloud/ai/example/deepresearch/controller/ChatController.java"

# 获取文件符号概览
/mcp__serena__jet_brains_get_symbols_overview "./src/main/java/com/alibaba/cloud/ai/example/deepresearch/controller/ChatController.java"

# 搜索代码模式
/mcp__serena__search_for_pattern "StateGraph" "src/main/java"
```

## 常见问题与解决方案

### 问题 1: 使用了 `javax.*` 包导致编译失败

**症状**:
```
package javax.servlet does not exist
package javax.persistence does not exist
```

**解决方案**:
1. 全局替换 `javax.servlet` → `jakarta.servlet`
2. 全局替换 `javax.persistence` → `jakarta.persistence`
3. 全局替换 `javax.validation` → `jakarta.validation`

### 问题 2: 使用了不存在的 Spring AI API

**症状**:
```
package org.springframework.ai.openai.api does not exist
```

**解决方案**:
1. 使用 Context7 获取 Spring AI 1.0.0 的正确 API 包路径：`Search Spring AI 1.0.0 ChatClient API. use context7`
2. 参考项目中现有的导入语句
3. 查看 `pom.xml` 确认依赖版本

### 问题 3: 代码示例来自旧版本文档

**症状**: 代码无法编译，但看起来与文档一致

**解决方案**:
1. 确认文档版本与项目版本匹配
2. 检查文档顶部的版本标识
3. 使用版本号限定搜索（如 "Spring Boot 3.4"）

## 项目特定约定

### 包结构规范

```
com.alibaba.cloud.ai.example.deepresearch/
├── controller/          # REST API 控制器
├── service/            # 业务逻辑服务
├── agents/             # AI 代理实现
├── node/               # Graph 执行节点
├── rag/                # RAG 相关组件
├── config/             # 配置类
├── model/              # 数据模型
│   ├── dto/            # 数据传输对象
│   ├── domain/         # 领域模型
│   ├── enums/          # 枚举类型
│   └── req/            # 请求对象
├── exception/          # 异常处理
├── mapper/             # MyBatis 映射器
└── util/               # 工具类
```

### 命名约定

- **Controller**: `{Name}Controller.java`
- **Service Interface**: `{Name}Service.java`
- **Service Implementation**: `{Name}ServiceImpl.java`
- **DTO**: `{Name}DTO.java` 或 `{Name}Request.java` / `{Name}Response.java`
- **Entity**: `{Name}.java`（数据库实体）
- **Mapper**: `{Name}Mapper.java`（MyBatis Plus）
- **Config**: `{Name}Config.java` 或 `{Name}Configuration.java`

### 注解使用规范

```java
// REST Controller
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/{resource}")
@CrossOrigin(origins = "*")
public class ExampleController {

    @PostMapping
    public ApiResponse<ResponseType> create(@RequestBody RequestType request) {
        // ...
    }

    @GetMapping("/{id}")
    public ApiResponse<ResponseType> getById(@PathVariable String id) {
        // ...
    }
}

// Service
import org.springframework.stereotype.Service;

@Service
public class ExampleServiceImpl implements ExampleService {
    // ...
}

// Configuration
import org.springframework.context.annotation.Configuration;

@Configuration
public class ExampleConfig {
    // ...
}

// Entity (MyBatis Plus)
import com.baomidou.mybatisplus.annotation.TableName;
import jakarta.persistence.Entity;

@TableName("table_name")
public class ExampleEntity {
    // ...
}
```

## 持续更新

本文档应随项目依赖升级而更新。当以下情况发生时，请更新本文档：

1. **Spring Boot 版本升级**: 记录新的 API 变更
2. **Spring AI 版本升级**: 记录新的包路径和 API
3. **Java 版本升级**: 记录新的语言特性
4. **新增重大依赖**: 添加版本约束和验证指南

---

**最后更新**: 2025-01-11
**维护者**: 开发团队
**反馈**: 如发现版本不一致问题，请及时更新本文档
