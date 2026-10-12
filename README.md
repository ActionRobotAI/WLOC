# WLOC

拦截并修改 Apple 网络定位（Wi-Fi / 基站）返回的坐标，用于在自有设备上进行定位相关测试。

> 仅供学习与自有设备测试使用。不修改 GPS 硬件，也不保证所有 App 都会采用修改后的位置。

---

## 它是怎么工作的

完整链路需要这几项同时就绪：

```text
代理客户端 → 启用模块 → 开启 HTTPS 解密 → 完全信任证书 → 设置目标位置
```

设置位置后的实际过程：

```text
你在网页或快捷指令中选择位置
    ↓
向 gs-loc.apple.com/wloc-settings/save 发起保存请求
    ↓
代理模块在本机拦截这个请求，把坐标写入本地
    ↓
之后系统请求网络定位时，模块修改返回的坐标
    ↓
地图等应用读到修改后的位置
```

缺少任意一步，都可能出现「保存成功但位置不变」或「无法获取位置」。

---

## 重要提示：iOS 版本

**iOS 27 正式版 / RC 当前不支持。**

从 iOS 27 beta 6 起，系统对相关证书限制加强，传统方式无法正常工作。  
如果你的设备已升级到 iOS 27 正式版，请不要反复重装模块或证书。

请先确认系统版本：

```text
设置 → 通用 → 关于本机 → iOS 版本
```

---

## 快速开始

第一次使用建议严格按这个顺序：

```text
1. 安装对应客户端模块
2. 开启模块
3. 开启 HTTPS 解密（MITM）
4. 安装 CA 描述文件
5. 在「证书信任设置」中开启完全信任
6. 确认代理正常连接
7. 设置目标位置
8. 按推荐流程刷新定位
9. 打开地图验证
```

---

## 订阅地址

| 客户端 | 订阅地址 |
|--------|----------|
| Stash | `https://raw.githubusercontent.com/ActionRobotAI/WLOC/main/modules/wloc.stoverride` |
| Surge / Egern | `https://raw.githubusercontent.com/ActionRobotAI/WLOC/main/modules/wloc.sgmodule` |
| Quantumult X | `https://raw.githubusercontent.com/ActionRobotAI/WLOC/main/modules/wloc.conf` |
| Loon | `https://raw.githubusercontent.com/ActionRobotAI/WLOC/main/modules/wloc.lpx` |
| Shadowrocket | `https://raw.githubusercontent.com/ActionRobotAI/WLOC/main/modules/wloc.module` |

> 模块显示「已安装」并不代表已经能用，还必须完成下面的证书信任。

### 各客户端大致路径

**Stash**  
模块 / Override → 添加 URL → 粘贴 `.stoverride` 地址 → 启用

**Surge**  
模块 → 安装新模块 → 从 URL 安装 → 粘贴 `.sgmodule` 地址 → 启用

**Loon**  
配置 → 插件 → + → 添加 URL → 粘贴 `.lpx` 地址 → 打开开关

**Shadowrocket**  
配置 / 模块 → 添加模块 → URL → 粘贴 `.module` 地址 → 保存并启用

**Quantumult X**  
在重写 / 模块页面导入对应 URL

---

## 开启 HTTPS 解密并信任证书

需要处理的域名：

```text
gs-loc.apple.com
gs-loc-cn.apple.com
gsp-ssl.ls.apple.com
bluedot.is.autonavi.com
bluedot.is.autonavi.com.gds.alibabadns.com
```

### 1. 开启解密并安装证书

在代理客户端中找到类似「MITM / HTTPS 解密 / 证书」的设置，生成 CA 证书并选择安装。

### 2. 安装描述文件

```text
设置 → 通用 → VPN 与设备管理
```

找到刚才的描述文件，按提示安装。

### 3. 开启完全信任（最容易漏）

```text
设置 → 通用 → 关于本机 → 证书信任设置
```

打开对应证书的「针对根证书启用完全信任」。

> 只安装描述文件不够。没完成这一步时，常见现象是：快捷指令提示网络连接已中断、请求失败、设置后没有效果。

完成后确认：**模块已启用、HTTPS 解密已开、代理正常连接**。

---

## 设置位置

### 方式一：网页选点

在 Safari 中打开选点页面，选点后点击「储存到设备」。

使用时 Safari 必须走代理，否则保存请求无法被拦截。

### 方式二：直接保存坐标

在浏览器中打开（替换为你的经纬度）：

```text
https://gs-loc.apple.com/wloc-settings/save?lat=35.6586&lon=139.7454&acc=25
```

该请求会被本地拦截并保存。

### 方式三：快捷指令

安装后，在地图中选点 → 分享 → 选择对应快捷指令。

- [设置地理位置xepes0](https://www.icloud.com/shortcuts/0a6465168d554135b78008a8b4bd7c01)
- [清理恢复位置](https://www.icloud.com/shortcuts/f42632d406504f24a2cd163af4fe012f)

**Apple 地图**：选点 → 共享 → 选择快捷指令  
**高德地图**：选点 → 分享 → 更多 → 选择快捷指令

分享菜单里没有时，先在「快捷指令」App 中手动运行一次，再回地图尝试。

---

## 部署选点页面（可选）

选点页面可以在地图上点选位置，或解析高德 / Apple 地图分享链接。  
不部署也可以，直接用保存链接或快捷指令。

推荐自行部署。进入本仓库的 `worker` 目录，需要 Node.js 22 或更新版本。

```bash
cd worker
npm ci
npm run build:check
npx wrangler login
npm run deploy
```

- `npm ci`：按锁文件安装依赖  
- `npm run build:check`：部署前检查  
- `npx wrangler login`：首次需要登录 Cloudflare  
- `npm run deploy`：发布  

部署不需要 KV 或数据库。完成后终端会输出你的页面地址，用 Safari 打开即可选点。

### 使用注意

- Safari 需要走代理，保存才能成功。
- 免费额度足够个人使用。
- 自己部署后，如使用快捷指令，需把指令内的解析地址改成你的域名。

---

## 保存后如何让位置生效

保存坐标后，建议按以下顺序刷新：

1. 关闭定位服务
2. 打开飞行模式，并确认 **Wi-Fi 和蓝牙都已关闭**
3. 等待约 10 秒
4. 关闭飞行模式
5. 等待网络和代理恢复（确认代理已重新连接）
6. 最后重新开启定位服务
7. 完全退出地图 App，再重新打开验证

> iOS 可能保留之前手动开启的 Wi-Fi / 蓝牙，不要只看飞行模式图标。  
> 如果在代理尚未恢复时就开启定位，系统可能先拿到未经过修改的结果。

---

## 恢复真实位置

任选一种：

1. 关闭或删除模块（推荐）
2. 运行「清理恢复位置」快捷指令
3. 访问：
   ```text
   https://gs-loc.apple.com/wloc-settings/save?action=clear
   ```

清除后如仍显示旧位置：

- 完全退出地图 App
- 必要时重启设备
- 检查模块参数里是否还写了固定的经纬度

---

## 常见问题

### 无法获取位置 / 网络连接已中断

请依次检查：

1. 模块是否已启用
2. 代理是否正常连接
3. HTTPS 解密是否开启
4. 证书是否已在「证书信任设置」中完全信任
5. MitM 域名是否完整

### 保存成功但地图位置不变

可能原因：

- 当前 iOS 版本不支持（尤其是 iOS 27 正式版）
- 证书未完全信任
- Safari 没有走代理
- GPS 信号较强，系统优先使用真实 GPS
- 定位缓存未刷新
- 同时启用了多个旧模块

建议在室内或 GPS 信号较弱的环境下首次测试。

### 位置过一会又变回去

只影响网络定位（Wi-Fi / 基站），不修改 GPS。室外 GPS 信号强时，系统可能重新使用真实位置。

### 高德分享菜单找不到快捷指令

高德需要：分享 → 更多 → 在系统分享菜单中查找。  
也可先在「快捷指令」App 中手动运行一次。

### 快捷指令提示「获取词典值失败」

通常是解析接口没有返回预期的 JSON。请确认快捷指令使用的是最新版本，且解析地址正确。

### 其他 App 不生效

- 大多数普通 App 会读取系统定位结果，通常有效。
- 自己请求定位服务器的 App 可能需要额外处理。
- 有反作弊机制的 App 可能忽略或限制修改后的位置。

---

## 注意事项

- 只影响网络定位，不修改 GPS 硬件。
- 必须完全信任证书，否则无法正常工作。
- 仅用于你拥有或获得授权的设备与 App 测试。
- 本地保存的坐标优先于模块中的默认参数。

---

## 目录说明

```text
modules/          各客户端模块
scripts/          核心脚本
worker/           选点页面与链接解析服务源码
shortcuts/        快捷指令说明
```
