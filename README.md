# WLOC 开源自部署资源包

基于 xepes0/wloc（Yu9191 Rewrite）整理的完整自部署包。  
核心功能：**拦截并修改 Apple 网络定位（gs-loc）返回的坐标**，实现 iOS 网络定位虚拟定位。

> 仅供学习与自有设备测试使用。不修改 GPS 硬件，也不保证所有 App 都会采用修改后的位置。

---

## 仓库信息

本资源包已配置为使用仓库：**https://github.com/ActionRobotAI/WLOC**

所有模块订阅链接、脚本地址和 Worker 部署链接均已指向该仓库。

---

## 目录结构

```
wloc-self-host/
├── README.md                  # 本说明文件
├── modules/                   # 各客户端模块
│   ├── wloc.stoverride        # Stash（推荐）
│   ├── wloc.sgmodule          # Surge / Egern
│   ├── wloc.conf              # Quantumult X
│   ├── wloc.lpx               # Loon
│   └── wloc.module            # Shadowrocket
├── scripts/                   # 核心脚本
│   ├── wloc.js                # 修改定位响应
│   └── wloc-settings.js       # 保存/读取坐标
├── shortcuts/                 # iOS 快捷指令说明与安装链接
│   └── README.md
```

---

## 一、最简部署（不需要 Worker）

只想固定在一个位置，或者手动填经纬度：

1. 在 Stash 中添加模块订阅：

   ```text
   https://raw.githubusercontent.com/ActionRobotAI/WLOC/main/modules/wloc.stoverride
   ```

   也可以直接导入本地的 `modules/wloc.stoverride` 文件。

2. 开启 **HTTPS 解密 / MITM**。
3. 安装并**完全信任** CA 证书：
   - 设置 → 通用 → VPN 与设备管理 → 安装描述文件
   - 设置 → 通用 → 关于本机 → 证书信任设置 → 打开信任开关
4. 在模块参数里修改默认经纬度，或通过选点方式写入。

坐标优先级：**本地保存的值 > 模块参数默认值**。

---

## 二、部署 Cloudflare Worker（选点页面 + 地图链接解析）

Worker 提供：
- 在线地图选点页面
- `/api/parse` 接口（解析高德/Apple 地图链接，自动做 GCJ-02 → WGS84）

### 方法 A：一键部署（推荐）

打开下面链接，登录 Cloudflare 授权即可：

```text
https://deploy.workers.cloudflare.com/?url=https://github.com/ActionRobotAI/WLOC/tree/main/worker
```

部署完成后会得到类似地址：

```text
https://你的项目名.你的子域名.workers.dev
```

### 方法 B：手动部署

```bash
cd worker
npm install
npx wrangler login          # 首次需要，会打开浏览器授权
npm run deploy
```

部署成功后终端会输出你的 Worker 地址。

> 免费套餐每天约 10 万次请求，个人使用完全足够。  
> 自己部署后，地图链接不会经过他人账号。

---

## 三、使用选点页面

1. 确保代理模块已启用，且 MITM 证书已信任。
2. 在 Safari 中打开你的 Worker 地址（建议添加到主屏幕）。
3. 在地图上选点，或粘贴地图链接。
4. 点击「储存到设备」。
5. 请求会被代理脚本拦截，坐标写入本地 `$persistentStore`。
6. 打开地图 App 验证位置是否已改变。

---

## 四、快捷指令（可选）

详细安装链接和使用说明见 `shortcuts/README.md`。

常用链接：

- 设置地理位置：https://www.icloud.com/shortcuts/a82717d8fdad4e6280866fcf911173f7
- 清理恢复位置：https://www.icloud.com/shortcuts/f42632d406504f24a2cd163af4fe012f

自部署 Worker 后，需要编辑快捷指令，把内部的解析地址改成你自己的 Worker 域名。

---

## 五、恢复真实定位

任选一种：

1. **推荐**：关闭或删除模块 → 脚本不再拦截 → 系统自动恢复。
2. 清除本地存储（透传模式）：
   - 在选点页面点「恢复」
   - 或在代理工具脚本编辑器中执行：`$persistentStore.write(null, "wloc_settings")`
3. iOS 26+ 有时需要重启设备清除定位缓存。

---

## 六、注意事项

- 必须开启 MITM，并**完全信任**证书，否则脚本无法解密响应。
- 只影响**网络定位**（Wi-Fi / 基站），GPS 信号强时可能被覆盖。
- iOS 27 beta 6 及之后版本存在证书限制，可能无法使用。
- 仅用于你拥有或获得授权的设备与 App 测试。
- 模块中的脚本地址已指向本仓库的 `scripts/` 目录。

---