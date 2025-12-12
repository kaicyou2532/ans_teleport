# ans_teleport

Teleport node を複数の VM / Docker ホストに対して配布し、既存の Teleport Cluster に join させるための Ansible Playbook です。

## できること

- VM(systemd) 向け: Teleport パッケージ導入 → `teleport` サービス起動
- Docker 向け: `teleport` コンテナ起動（token を引数で渡す）
- join token / CA pin の配布（CA pin は設定ファイル、token は安全性を考慮して systemd 環境変数 or コンテナ引数で注入）

## 前提

- Teleport Cluster 側で node 用の join token を発行済み
- ターゲットへ SSH 接続できること
- VM は Debian/Ubuntu または RHEL系を想定
- Docker ホスト側は Docker Engine が動作していること（Playbookは best-effort で `docker` パッケージ導入も試みます）

> 注意: join token は秘密情報です。`group_vars/all.yml` をそのままコミットしない運用（Ansible Vault等）を推奨します。

## 使い方

### 1) Inventory を編集

`inventory/hosts.yml` を自分の VM / Docker ホストに合わせて更新してください。

- `teleport_vm`: systemd で動かすノード
- `teleport_docker`: Docker で動かすノード

### 2) 変数を設定

`group_vars/all.yml` の以下を設定します。

- `teleport_proxy_addr`: 例 `proxy.example.com:443`
- `teleport_join_token`: Teleport で発行した node join token
- `teleport_ca_pins`: 任意（推奨） 例 `["sha256:..."]`
- `teleport_node_labels`: 任意

### 3) community.docker を入れる（Dockerグループを使う場合）

`requirements.yml` を使って collection を入れます。

```bash
ansible-galaxy collection install -r requirements.yml
```

### 4) Playbook 実行

```bash
ansible-playbook playbooks/teleport_nodes.yml
```

## ファイル構成

- `playbooks/teleport_nodes.yml`: エントリポイント
- `roles/teleport_node_vm`: VM向け role
- `roles/teleport_node_docker`: Docker向け role
- `group_vars/all.yml`: 共有の変数例
- `requirements.yml`: Docker role 用 collection

## セキュリティメモ

- token は systemd drop-in (`/etc/systemd/system/teleport.service.d/10-join-token.conf`) で環境変数として注入します。
  - `teleport.yaml` に token を直書きしないため、設定ファイルの流出時のリスクを下げます。
- より安全にするなら:
  - `teleport_join_token` を Ansible Vault で暗号化
  - 期限付き token を使い、配布完了後に revoke

## トラブルシュート

- join に失敗する場合
  - `teleport_proxy_addr` が正しいか（443/TLS 経由など）
  - token が node 用か（`roles: [node]` 相当）
  - `teleport_ca_pins` を設定している場合は pin が正しいか

## ライセンス

必要なら追記してください。
