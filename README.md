# Sales Skills

一组可复用、行业中立的 Codex 销售 Skills，用于完成销售项目初始化、潜在客户研究、可选邮件触达、Contact List 更新、跟进和结果汇报。

## 包含内容

- `sales-onboarding`：通过对话完成销售项目初始化，确认 Contact List、邮件服务及必要业务信息。
- `sales-workflow`：执行带检查点的通用销售流程，包括研究、筛选、触达、记录、跟进和汇报。
- `example-sales`：可复制并按公司需求修改的示例 implementation。

## 安装

1. 下载或克隆本仓库。
2. 将 `skills` 目录中的内容复制到 Codex Skills 目录：

   ```text
   ~/.codex/skills/
   ```

3. 安装后目录结构应类似：

   ```text
   ~/.codex/skills/
   ├── sales-onboarding/
   ├── sales-workflow/
   └── implementations/example-sales/
   ```

更完整的步骤请参阅 [DEPLOYMENT.md](DEPLOYMENT.md)。

## 开始使用

1. 首次运行 `sales-onboarding`，根据聊天引导完成配置。
2. 复制 `implementations/example-sales`，创建自己的销售 implementation。
3. 在配置状态为 `ready` 后，通过该 implementation 调用 `sales-workflow`。

邮件发送和 Contact List 写入默认都需要明确配置及授权。系统不会猜测联系人邮箱，也不会将未确认的发送标记为成功。

## 配置与安全

- 仓库中的 YAML 文件仅为示例，不包含真实凭证或客户资料。
- 不要提交密码、API Key、Token、真实客户数据或包含个人信息的运行产物。
- 建议只提交脱敏后的示例配置；真实配置请保存在本地。

## License

本项目采用 [MIT License](LICENSE)。
