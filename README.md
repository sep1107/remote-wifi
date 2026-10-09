# remote Wi-Fi

Kindle 原系统 KUAL Wi-Fi 接收端，配套安卓 [远程控制器](https://github.com/sep1107/remote-controller)。

需要已越狱、KUAL 和已安装的 KOReader。接收端只使用 KOReader 的 LuaJIT／LuaSocket，不启动其界面，不修改 KindleLazy，不加载 USB HID 模块。

1. 将 本仓库的 `config.xml`、`menu.json`、三个 `.sh`、`server.lua` 和 `control.lua`复制到 Kindle 的 `extensions/kindle-remote/`，包括 `config.xml`。
2. 安全弹出 USB，退出并重新打开 KUAL。
3. 在 **remote Wi-Fi → Start native remote (8080)** 启动，退出 KUAL并打开原系统书籍。停止时点 `Stop native remote`。

### 与 KOReader 共用端口

为了自动交接，先备份 Kindle 的 `extensions/koreader/menu.json`，再将其中启动动作：

```text
/mnt/us/koreader/koreader.sh
```

替换为：

```text
/mnt/us/extensions/kindle-remote/koreader-launch.sh
```

保持 `params` 不变。启动脚本会在进入 KOReader 前停止此前运行的原系统接收端，正常退出后恢复它。KOReader HTTP 服务需配置自动启动。该机制仅覆盖经 KUAL 启动的入口；其他入口需要手动停止原系统接收端。KOReader 更新可能覆盖 KUAL 菜单，更新后检查。

接收端不自动开机启动。原系统翻页通过 `/dev/input/event1` 注入触摸，优先读取驱动坐标，失败时使用 KPW3 的 1072×1448 坐标。亮度限制为 0–24；息屏仅在设备处于 active 时触发，屏保状态下不会误唤醒。横屏、倒置翻页未实测。日志在扩展目录 `server.log`、`receiver.log`。

设备睡眠后 Wi-Fi 可能断开，需实体电源键或保护套唤醒。两种 HTTP 接口均无认证，仅在可信局域网启用，勿将端口暴露到公网。


## 实现与限制

需要越狱、KUAL 和 KOReader（只复用其 LuaJIT／LuaSocket，不运行界面）。使用 8080；原系统翻页、亮度、息屏在 KPW3 已验证。通过 `/dev/input/event1` 注入触摸，坐标查询失败时使用 PW3 1072×1448。其他机型、横屏与倒置未验证。亮度限制 0–24；睡眠请求只在 active 状态触发，已睡眠时不会误唤醒。设备睡眠后需电源键或保护套唤醒。

日志：`server.log`、`receiver.log`。固定 HTTP 命令与 KOReader 一致；仅实现五种操作，不提供完整 Inspector 或任意 shell 执行。无认证，请仅在可信局域网启动，不要把 8080 暴露公网。

## 测试

```sh
luajit tests/control-test.lua .
sh -n start.sh stop.sh koreader-launch.sh
```

## 许可与参考

[MIT License](LICENSE)。不附带 KOReader／LuaJIT／LuaSocket 运行文件，上游许可证由各项目维护。触摸协议和控制思路参考 [KindleLazy](https://github.com/llakssz/KindleLazy)，HTTP 与电源接口参考 [KOReader](https://github.com/koreader/koreader)。
