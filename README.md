# adnull

`adnull` 本地 DNS 过滤器的规则分发仓库。

## 文件说明

| 文件 | 用途 |
| --- | --- |
| `blocklist.txt` | 规则本体。每行一个域名，匹配该域及其全部子域；`#` 开头为注释，`!` 开头为白名单 |
| `blocklist.meta.json` | 元数据。含版本号、条目数与 **`rules_sha256`** |

## 客户端如何校验

客户端（`com.adnull.filter`）拉取规则时的顺序固定：

1. `GET blocklist.meta.json` → 读出 `rules_sha256`
2. `GET blocklist.txt` → 本地重算 SHA-256，与上一步声明的值比对
3. **比对通过**才原子替换本地规则文件；任何一步失败都保留旧规则

因此 **`rules_sha256` 必须与 `blocklist.txt` 的实际字节一致**（对完整文件内容算哈希，
包含注释行与白名单行）。改规则时两个文件必须同时更新，缺一不可 ——
只更新其一会导致客户端校验失败、静默沿用旧规则。

## 客户端配置的地址

- 主（CDN，国内可达性更好）：`https://cdn.jsdelivr.net/gh/Remnantdusk/adnull@main/blocklist.txt`
- 备（GitHub raw）：`https://raw.githubusercontent.com/Remnantdusk/adnull/main/blocklist.txt`

## 更新流程

规则由生成脚本产出，不要手工编辑 `blocklist.txt`：

```bash
python tools/dev/emit_release.py     # 产出 blocklist.txt 与 blocklist.meta.json
```

该脚本会对「最终写盘的完整字节」算哈希，并用下载者视角独立复算一次做自检。
