# ci-templates

kakusan0 のリポジトリで共通に使う GitHub Actions のワークフロー置き場です。

## docker-publish.yml

Docker イメージをビルドして GHCR（`ghcr.io`）に公開します（`docker/build-push-action`）。

- タグ：`latest` と `sha-<コミット先頭7文字>`
- GitHub Actions のキャッシュでビルドを短縮
- 認証は呼び出し元の `GITHUB_TOKEN`（secrets の登録は不要）

### 使い方

呼び出し側のリポジトリの `.github/workflows/` に、次のようなジョブを書きます。

```yaml
jobs:
  publish:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write
    uses: kakusan0/ci-templates/.github/workflows/docker-publish.yml@main
    with:
      context: .
```

イメージが複数あるときは matrix を使います。

```yaml
jobs:
  publish:
    strategy:
      matrix:
        service: [backend, frontend]
    permissions:
      contents: read
      packages: write
    uses: kakusan0/ci-templates/.github/workflows/docker-publish.yml@main
    with:
      context: ./${{ matrix.service }}
      image: ghcr.io/${{ github.repository }}-${{ matrix.service }}
```

### 入力

| 名前 | 必須 | 既定値 | 内容 |
|---|---|---|---|
| `context` | 必須 | | ビルドするフォルダ |
| `image` | | `ghcr.io/<呼び出し元のリポジトリ>` | 公開先のイメージ名（自動で小文字にする） |
| `file` | | `<context>/Dockerfile` | Dockerfile のパス |
| `platforms` | | `linux/amd64` | CPU アーキテクチャ |

### 出力
`image`（公開したイメージ名）、`digest`（ダイジェスト）

### 注意
- 秘密情報（API キーなど）はイメージに入れないでください。実行時に環境変数で渡します。
- 非公開リポジトリのイメージは非公開になります。無料枠（GitHub Free）はアカウント全体で保存 500MB・転送 1GB/月です。
