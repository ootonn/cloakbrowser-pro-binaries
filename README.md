# cloakbrowser-pro-binaries

面向 [finger-chromium](https://github.com/ootonn/finger-chromium) / StealthBrowser 的 **Pro Chromium 公共 Release 镜像**。客户端经自有 License 网关（如 `https://prism.ootonn.com`）解析版本并 302 到此仓库，避免每台机器直连 Cloak API。

## 必读：自签租约签名消息（exit 78 根因之一）

Pro 152 验证 `session/start` 返回的 **两段式 token**（`base64url(claims).base64url(signature)`）时，Ed25519 验签输入是 **canonical JSON 的 UTF-8 原始字节**（`json.dumps(claims, sort_keys=True, separators=(",", ":"))`），**不是** token 第一段的 base64url 文本。

| 网关签名对象 | Python `verify_token` 若同样验错对象 | 二进制行为 |
|---|---|---|
| `segment.encode("ascii")`（base64 段） | 可能 **自检通过** | `POST …/session/start` **HTTP 200**，约 1s 后写 **denial 78**，通常 **无 heartbeat** |
| **payload 原始 JSON 字节** | 与二进制一致 | 正常打开页面，`native_denial_code` 为空 |

78 的文案是 “license server unreachable / connection problem”，**验签失败也会走同一码**，不能单凭 78 断定是网络问题。DNS-only 源站若仍启用 `prism_trusted_edge` 444 也会 78，需与签名问题区分。

因此本镜像默认发布的 **`-prism`** 包替换了 `chrome.dll` 内嵌公钥，且配套网关必须用 **payload 字节** 签名（见 finger-chromium `lease_signing.py`）。未补丁的 vendor zip 只能验厂商 token，不能验 Prism 自签。

实现细节与验收：[finger-chromium `docs/PRISM_SELF_SIGNED_ACCEPTANCE.md`](https://github.com/ootonn/finger-chromium/blob/main/docs/PRISM_SELF_SIGNED_ACCEPTANCE.md)、[`docs/PRISM_PRO_BINARY_STRATEGY.md`](https://github.com/ootonn/finger-chromium/blob/main/docs/PRISM_PRO_BINARY_STRATEGY.md)。

## Release 布局

| 项目 | 约定 |
|---|---|
| Git tag | `chromium-v{version}-pro`（例：`chromium-v152.0.7977.82.1-pro`） |
| **Windows（finger-chromium 默认）** | `cloakbrowser-windows-x64-prism.zip` + `SHA256SUMS.prism` |
| Windows（厂商对照） | `cloakbrowser-windows-x64.zip` |
| Linux x64 | `cloakbrowser-linux-x64.tar.gz` |
| Linux arm64 | `cloakbrowser-linux-arm64.tar.gz` |

### 当前 Windows Prism 包（152.0.7977.82.1）

| 资产 | SHA256 |
|---|---|
| `cloakbrowser-windows-x64-prism.zip` | `e577822e6756291d370b5c48743b8c3ef80ec1b5e2a85127880dc440adb2cc47` |

下载：[Release `chromium-v152.0.7977.82.1-pro`](https://github.com/ootonn/cloakbrowser-pro-binaries/releases/tag/chromium-v152.0.7977.82.1-pro) 中的 `cloakbrowser-windows-x64-prism.zip`。

客户端校验：同 tag 下的 **`SHA256SUMS.prism`**（非 CloakHQ 签名的 `SHA256SUMS`），环境变量 `STEALTHBROWSER_PRO_PRISM_MIRROR=true`、`STEALTHBROWSER_PRO_ARCHIVE_SUFFIX=-prism`。

## 如何发布 / 更新 -prism 包

在 finger-chromium 仓库根目录，本机已有 **已补丁** 的 `chrome.dll`（与网关 `signing.key` / `verify.pub` 成对）：

```powershell
$env:GITHUB_MIRROR_REPO = "ootonn/cloakbrowser-pro-binaries"
# GITHUB_TOKEN 或 gh auth login
python scripts/prism_publish_patched_mirror.py --version 152.0.7977.82.1
```

全平台同步（含 vendor 下载 + Windows 补丁流水线）：

```powershell
$env:PRISM_PATCH_MIRROR = "1"
.\scripts\sync-pro-mirror.ps1
```

## Disclaimer

Binaries are **Pro** builds; use only if your license allows. This mirror is maintained for private infrastructure convenience and is **not** affiliated with CloakHQ or StealthHQ. Do not commit `license.key`, `signing.key`, or tokens to any repository.
