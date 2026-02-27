

## 配置文件模板

```ini
[Unit]
Description=service描述文本
After=network.target  # 确保网络就绪后再启动，适合大多数需要网络的程序

[Service]
Type=simple  # (默认)主进程启动即完成，不会 fork。适合持续运行的前台程序，如直接运行的二进制、脚本
User=your_user  # 指定用户/组运行。强烈建议使用非 root 用户运行，提升安全性
Group=your_group
WorkingDirectory=/path/to/workdir  # 工作目录
Environment="KEY=value"  # 设置环境变量，可多次使用。如果变量多，可用 `EnvironmentFile=/etc/your-service/env`
ExecStart=/full/path/to/your/binary --arg1 --arg2   # 启动命令（建议使用绝对路径，避免环境问题）
ExecStop=/bin/kill -SIGTERM $MAINPID   # 停止命令（显式指定停止信号，保证优雅退出（可省略，systemd 默认发 SIGTERM））
Restart=on-failure  # 重启策略：`no`（永不）、`on-success`、`on-failure`、`on-abnormal`、`always`。on-failure仅在非正常退出时重启（如崩溃、信号终止），手动停止则不重启，符合预期
RestartSec=10  # 重启前等待的秒数
StandardOutput=journal  # 日志输出位置：`journal`（默认）、`syslog`、`file:/path`、`null`
StandardError=journal  # 将日志输出到 systemd 日志，用 `journalctl -u your-service` 查看

[Install]
WantedBy=multi-user.target  # 定义该服务被哪个 target 引用，通常填 `multi-user.target`，表示开机启动进入多用户模式时启动本服务
```

按需可添加的扩展项：
- **内存/CPU限制**：通过 `MemoryMax=`、`CPUQuota=` 实现资源控制。
- **依赖其他服务**：在 `[Unit]` 中添加 `Requires=postgresql.service` 和 `After=postgresql.service`。

注意事项
- **绝对路径**：`ExecStart` 中务必使用绝对路径，避免 PATH 找不到
- **Type 选择**：如果程序会 fork 并 daemonize，需用 `forking`；否则用 `simple`
- **后台运行**：服务程序不要自己后台运行（如添加 `&`），systemd 会接管前台
- **权限**：确保指定的 `User` 对工作目录和文件有读写权限
- **停止信号**：若程序需要特殊停止信号（如 `SIGQUIT`），可自定义 `ExecStop`

## 使用和调试技巧

1. **创建 service 文件**：保存到 `/etc/systemd/system/your-service.service`
2. **重新加载配置**：`sudo systemctl daemon-reload`
3. **启动/停止**：`sudo systemctl start your-service` / `sudo systemctl stop your-service`
4. **设置开机自启**：`sudo systemctl enable your-service`
5. **查看状态**：`sudo systemctl status your-service`
6. **实时查看日志**：`journalctl -u your-service -f`
7. **测试配置**：修改后先用 `systemd-analyze verify /etc/systemd/system/your-service.service` 检查语法