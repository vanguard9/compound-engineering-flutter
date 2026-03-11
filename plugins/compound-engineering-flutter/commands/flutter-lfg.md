---
name: flutter-lfg
description: Flutter 全自动工程工作流：计划、开发、审查一体化
argument-hint: "[功能描述]"
disable-model-invocation: true
---

重要：你必须严格按照以下步骤**顺序**执行。不要跳过任何步骤，不要提前开始编码。计划阶段必须在开发之前完成。

1. `/flutter:plan $ARGUMENTS`

   **检查点**：停下来。验证 `/flutter:plan` 是否在 `docs/plans/` 中生成了计划文件。如果没有，重新运行。直到计划文件存在才能继续。

2. `/flutter:work`

   **检查点**：停下来。验证所有任务是否都已标记完成。运行 `flutter test` 确保所有测试通过。

3. `/flutter:review latest`

   **检查点**：审查通过后继续。如果有 🔴 必须修复项，修复后再次审查。

4. 最终验证：

   ```bash
   flutter analyze
   flutter test
   dart format --set-exit-if-changed lib/ test/
   ```

5. 输出完成总结：
   - 实现了什么功能
   - 新增/修改的文件列表
   - 测试覆盖情况
   - 还有哪些需要手动验证的场景
