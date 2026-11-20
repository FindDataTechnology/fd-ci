# fd-ci — FindDataTechnology 的共享 CI

被整个组织复用的 GitHub Actions workflow。**零拓扑、零凭据**——这里只有流水线定义。

## 为什么有一个独立的仓

31 个服务仓各有自己的 `.github/workflows/image.yml`，干同一件事（checkout → buildx
→ login → build-push → 内容闸门铸 `sha-<7>` tag → step summary），但行数从 98 到 207
不等。改一处发布逻辑要改 31 遍，且没有机制能证明改全了。

GitHub 的 `workflow_call` 让这些逻辑可以只写一次。跨仓调用要求**被调用的 workflow
所在仓对调用方可见**——所以本仓必须公开。

## 怎么用

服务仓的 `.github/workflows/image.yml`：

```yaml
name: image
on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build-push:
    uses: FindDataTechnology/fd-ci/.github/workflows/image-build.yml@main
    secrets: inherit
    with:
      image: fd-text-core        # 必填
      namespace: finddata        # 必填
      # dockerfile: Dockerfile   # 可选，默认 ./Dockerfile
      # context: .               # 可选，默认 .
```

多镜像的仓写多个 job：

```yaml
jobs:
  web:
    uses: FindDataTechnology/fd-ci/.github/workflows/image-build.yml@main
    secrets: inherit
    with: { image: fd-lex-web, namespace: finddata-business, context: web, dockerfile: web/Dockerfile }
  api:
    uses: FindDataTechnology/fd-ci/.github/workflows/image-build.yml@main
    secrets: inherit
    with: { image: fd-lex-api, namespace: finddata-business, context: server, dockerfile: server/Dockerfile }
```

## 凭据

共享 workflow 同时认两个名字：`TCR_USERNAME`/`TCR_PASSWORD`（仓级）与
`TCR_USER`/`TCR_PASS`（组织级）。**名字不匹配时 GitHub 静默给空串，报错长得像
「密钥缺失」**——2026-11-20 实撞过（`fd-data-quality` 没有任何仓级密钥，
login 报 `Username and password required`）。

缺凭据时 workflow 会直接报出该 set 哪两个 secret，而不是让 `docker/login-action`
去猜。

## 保留的语义

`image-build.yml` 逐条对照了 31 份原实现：

- `provenance: false` —— **硬约束**。开了 attestation，同内容两次构建 digest 必然
  不同，内容闸门会永久判「变了」而失效。
- `platforms: linux/amd64` —— 现状全是单平台。
- 内容闸门 —— digest 与 `main` 当前相同则不铸新 tag（`openspec: gate-image-tags-on-content-change`）。
- 不可变 `sha-<7>` tag，禁 `latest`。

## 豁免

因客观原因不能使用共享 workflow 的仓登记在这里，每条带理由：

（暂无）

## License

MIT — 见 `LICENSE`。
