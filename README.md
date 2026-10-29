# 在线更新分发仓库

本目录是一个**独立的 git 仓库**，远端根目录就是它自己：
Gitee `jeay/doye-php-eshop2026`、GitHub `jeeaay/doye-php-eshop2026`。
仓库里**只有增量包与清单**，不含任何源码；源码只在维护者本地。

站点端地址（公开仓库，免鉴权，硬编码在 `config/update.php`）：

- Gitee（优先）：`https://gitee.com/jeay/doye-php-eshop2026/raw/main/update.json`
- GitHub（兜底）：`https://raw.githubusercontent.com/jeeaay/doye-php-eshop2026/main/update.json`
- 资产：`{上述前缀}/v{版本}/{文件名}`

## 目录结构

```
update.json                   # 链式清单（累积，每次发布追加一条 step）
update.json.sig               # update.json 的 Ed25519 分离签名（信任根）
v1.0.1/
  delta-1.0.0-to-1.0.1.zip    # 只含 1.0.0 → 1.0.1 变动的文件
  delta-1.0.0-to-1.0.1.json   # 逐文件 add/modify/delete + before/after sha256
  vendor-1.0.1.zip            # 仅 composer.json / composer.lock 变更时生成
```

## 发布

```bash
# 1) 源码仓库：改 version.json 的 version，提交并打 tag
git commit -am "chore: bump version to 1.0.1"
git tag v1.0.1

# 2) 源码仓库：出包，并在本目录（独立仓库）里提交 + 推送 main 到 gitee/github
php think release:build --version=1.0.1 --push

# 3) 校验两个平台的 raw 清单与资产
php think release:verify
```

`--push` 只提交本目录、只推送本仓库的 `main`；源码仓库的提交与 tag 不会出现在这里。
远端历史被重建（例如手动清空过仓库）时加 `--force-push`。

## 禁止事项

- **不要删除历史版本目录**：链式增量依赖上一版到本版的连续性，删掉任何一个版本都会让停在那个版本的站点永久无法升级。
- **不要手工修改 `update.json`**：任何改动都会让签名失效；改内容必须走 `release:build` 重新签发。
- **私钥不在本仓库**：签名私钥在维护者机器的 `runtime/release-signing.key`，已被 `.gitignore` 忽略。
- **不要把源码放进本仓库**：这里对外公开，只允许 `update.json`、`update.json.sig` 与 `v*/` 下的增量包。