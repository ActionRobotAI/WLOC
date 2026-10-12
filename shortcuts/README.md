# iOS 快捷指令

快捷指令用于从地图 App 分享链接，一键设置或恢复位置。

## 常用公共快捷指令（可直接安装）

| 名称 | 链接 | 说明 |
|------|------|------|
| wloc 设置地理位置 | https://www.icloud.com/shortcuts/a82717d8fdad4e6280866fcf911173f7 | 接收地图分享链接，解析后保存坐标 |
| wloc 清理恢复位置 | https://www.icloud.com/shortcuts/f42632d406504f24a2cd163af4fe012f | 清除已保存的虚拟坐标 |

其他可用版本：

- 设置位置（xepes0）：https://www.icloud.com/shortcuts/0a6465168d554135b78008a8b4bd7c01
- 设置位置（cyberhandyman）：https://www.icloud.com/shortcuts/182f3a014597468eb1b15b99261cdf22
- 清理位置（cyberhandyman）：https://www.icloud.com/shortcuts/0352d53ed79849d382f50e9adf050662

## 安装方法

1. 在 iPhone 的 Safari 中打开上面的 iCloud 链接。
2. 点击「获取快捷指令」。
3. 首次使用需在「设置 → 快捷指令」中开启「允许不受信任的快捷指令」。

## 使用方法

1. 确保代理模块已启用，MITM 证书已信任。
2. 在 Apple 地图或高德地图中长按选点。
3. 点击分享 → 选择「wloc 设置地理位置」。
4. 打开地图 App 验证位置。

恢复：运行「wloc 清理恢复位置」。

## 自部署 Worker 时的修改

公共快捷指令内部写死了公共 Worker 的解析地址。  
如果你部署了自己的 Worker，需要编辑快捷指令：

1. 打开「快捷指令」App，找到已安装的设置位置指令。
2. 编辑，找到「获取 URL 内容」动作。
3. 把解析地址（如 `wloc-spoofer.wloc.workers.dev` 或 `wloc.xepesw.workers.dev`）替换为你自己的 Worker 域名。
4. 保留路径 `/api/parse` 以及查询参数。
5. **不要修改** `https://gs-loc.apple.com/wloc-settings/save`，这是本地拦截的保存接口。

## 说明

Apple 快捷指令通过 iCloud 链接分享，无法直接打包成可离线导入的 `.shortcut` 文件放入本仓库。  
安装后按上面步骤修改 Worker 地址即可适配自部署环境。
