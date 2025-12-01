
---

### 1. 变更概要

- 新增了 JGit 依赖，为后续 Git 操作做准备
- 创建了 ReviewLogService 服务类，用于记录代码审核日志到文件系统
- 新增 EnvUtil 工具类，提供环境变量读取功能
- 在 FileUtil 中增加了递归删除文件和净化文件名的工具方法

### 2. 主要风险与问题

- [严重] ReviewLogService.addReviewLog 方法中存在复制粘贴错误：获取 `repoBranch` 和 `repoCommitHash` 时都错误地使用了 `reviewRepoName` 环境变量，应该分别使用 `reviewRepoBranch` 和 `reviewRepoCommit`
- [严重] addReviewLog 方法创建了临时目录但没有任何清理机制，可能导致磁盘空间泄露
- [中等] FileUtil.deleteRecursively 方法将 IOException 包装为 RuntimeException 抛出，破坏了异常处理的契约，应该直接抛出 IOException 或使用自定义异常
- [中等] ReviewLogService 作为工具类但没有 final 构造函数，容易被实例化
- [轻微] addReviewLog 方法参数 `reviewContent` 未被使用，且方法返回空字符串，缺乏实际意义

### 3. 可读性与风格建议

- ReviewLogService 中的常量 `baseDir` 应该改为 `BASE_DIR` 以符合 Java 命名规范
- addReviewLog 方法中的环境变量获取混用了常量和字符串（如 `reviewRepoToken` 和 `ReviewLogService.reviewRepo`），应该统一使用常量
- 建议将 ReviewLogService 的 static 方法改为实例方法，通过依赖注入管理，便于测试和扩展
- FileUtil.main 方法中的硬编码路径应该移除

### 4. 性能与资源使用

- Files.createTempDirectory 创建的临时目录需要在使用后清理，建议使用 try-with-resources 或添加清理逻辑
- FileUtil.deleteRecursively 使用 Files.walk 可能在大目录时消耗较多内存，但考虑到是临时目录清理，影响有限

### 5. 安全性

- FileUtil.purifyFileName 方法正确处理了文件名中的特殊字符，防止路径遍历风险
- 建议对 addReviewLog 的 reviewContent 参数进行长度限制和内容校验，防止恶意输入

### 6. 测试建议

- 缺少单元测试，建议为 EnvUtil 的两个方法添加测试用例
- 应该为 FileUtil.deleteRecursively 和 purifyFileName 添加测试
- ReviewLogService.addReviewLog 需要集成测试验证文件创建逻辑

### 7. 结论

本次改动引入了代码审核日志记录的基础功能，但存在严重的逻辑错误和资源泄露问题。建议修复环境变量获取错误、添加临时目录清理机制、完善异常处理后合入。

**结论：需要修改**