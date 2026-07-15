# Sales Skills 部署说明

1. 解压 `skills.zip`。
2. 将解压出的 `skills` 文件夹内容复制到 Codex 的 Skill 目录：

   ```text
   ~/.codex/skills/
   ```

   完成后应包含：

   ```text
   ~/.codex/skills/
   ├── sales-onboarding/
   ├── sales-workflow/
   └── implementations/example-sales/
   ```

3. 首次使用时，运行 `sales-onboarding`。它会在聊天中引导你选择 Contact List、连接或关闭 Agent Mail，并收集必要的业务信息。
4. 配置完成后，使用 `example-sales` implementation 执行销售自动化。

注意：不要将密码、密钥或真实客户数据提交到公开仓库。
