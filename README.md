# onepassworditem (expectedbehaviors fork)

> **Fork:** This repository is a fork of [vquie/OnePasswordItem-helm](https://github.com/vquie/OnePasswordItem-helm). Original author: [@vquie](https://github.com/vquie). We do not claim credit for the original design; only our changes (schema, namespace-from-release, hook) are ours.

Helm chart to sync [1Password](https://1password.com/) items into Kubernetes Secrets via the [1Password Kubernetes Operator](https://github.com/1Password/onepassword-operator). Each item becomes a `OnePasswordItem` custom resource; the operator creates the Secret in the same namespace.

**This is a fork of the original chart by [vquie](https://github.com/vquie/OnePasswordItem-helm)** (chart also published at https://vquie.github.io/helm-charts). We do not claim credit for the original design. This fork changes the values schema and behavior as described below.

## Changes in this fork

- **Namespace from release:** OnePasswordItem resources are always created in **`.Release.Namespace`**.
- **Simplified schema:** Use a single list **`items`** instead of **`secrets.<namespace>: [ ... ]`**.
- **`enabled` toggle:** Set **`enabled: false`** to create no OnePasswordItem resources.

## Prerequisites

- [1Password Kubernetes Operator](https://github.com/1Password/onepassword-operator) installed and configured.
- Items and vaults exist in 1Password. Paths use the form `vaults/<vault>/items/<item>`. **Vault and item** can be 1Password **ID or exact title** (name).

## Values

| Key            | Type    | Default | Description |
|----------------|--------|--------|-------------|
| `defaultVault` | string | `""`   | Default vault (id or title). Used when an entry does not set `vault`. You must set either `defaultVault` or each entry's `vault` unless the entry uses a full path in `item`. |
| `enabled`      | bool   | `true` | If `false`, no OnePasswordItem resources are rendered. |
| `items`        | list   | `[]`   | List of 1Password items to sync. See below. |

Each entry in `items`:

| Key               | Type   | Required | Description |
|-------------------|--------|----------|-------------|
| `vault`           | string | no*      | Vault (id or title) for this entry. Falls back to top-level `defaultVault` if unset. *Required when `item` is not a full path unless `defaultVault` is set. |
| `item`            | string | yes      | Item (id or title), or **full path** `vaults/<vault>/items/<item>` (backward compatible). If not a full path, path is built from `vault` or `defaultVault` + `item`. |
| `name`            | string | no       | Name of the Kubernetes Secret to create. Defaults to `item` (or the last path segment of `item` when `item` is a full path). |
| `type`            | string | no       | Kubernetes Secret type (default `Opaque`). |
| `annotations`     | map    | no       | Annotations to apply to the Secret (post-install hook). |
| `labels`          | map    | no       | Labels to apply to the Secret (post-install hook). |

**Template validation:** Each entry must have `item`. When `item` is not a full path (`vaults/.../items/...`), the chart requires either the entry's `vault` or the top-level `defaultVault`; otherwise the template will fail with an error.

## Secret annotations and labels

If you need the Secret to be annotated or labeled (e.g. for [kubernetes-replicator](https://github.com/mittwald/kubernetes-replicator)), add **`annotations`** and/or **`labels`** to any item in **`items`**. A post-install/upgrade Helm hook (Job + RBAC) patches the Secret after the operator creates it. No hook is created if no item has `annotations` or `labels`.

**Vault + item (recommended):** Set `defaultVault` and use short `item` names; override `vault` per entry if needed. Vault and item accept **ID or exact title**.

```yaml
defaultVault: Kubernetes
items:
  - item: myapp
    type: Opaque
  - vault: AnotherVault
    item: other-secret
    name: other-secret
```
(When `name` is omitted, the Secret name defaults to the item id/title, or the last segment of a full path.)

Path is built as `vaults/<vault>/items/<item>` where `vault` is the entry's `vault` or the top-level `defaultVault`. You must provide either `defaultVault` or each entry's `vault` when `item` is not a full path.

**Full path (backward compatible):** You may still use a full path in `item`:

```yaml
items:
  - item: vaults/Kubernetes/items/myapp
    name: myapp
    type: Opaque
```

Example with annotations (e.g. for kubernetes-replicator):

```yaml
defaultVault: Kubernetes
items:
  - item: myapp
    type: Opaque
    annotations:
      replicator.v1.mittwald.de/replication-allowed: "true"
      replicator.v1.mittwald.de/replication-allowed-namespaces: "*"
    labels: {}
```

## Installation

```bash
helm repo add expectedbehaviors https://expectedbehaviors.github.io/OnePasswordItem-helm
helm install my-secrets expectedbehaviors/onepassworditem -n my-namespace -f values.yaml
```

## Using as a subchart

In your chart's `Chart.yaml`:

```yaml
dependencies:
  - name: onepassworditem
    version: "1.1.0"
    repository: https://expectedbehaviors.github.io/OnePasswordItem-helm
```

Pass values under the dependency name (e.g. `onepassworditem.enabled`, `onepassworditem.items`). OnePasswordItem resources are created in the parent release's namespace.

## Publishing (this repo)

This chart is published from **expectedbehaviors/OnePasswordItem-helm**. Merge to `main` triggers release and Helm publish to **gh-pages**. Enable **Settings → Pages → branch: gh-pages** once.

## Original chart

- **Author:** [vquie](https://github.com/vquie/OnePasswordItem-helm)
- **Helm repo:** https://vquie.github.io/helm-charts  
- Original schema uses `secrets.<namespace>: [ { item, name, type } ]`. This fork uses `enabled` + `items` and derives namespace from the Helm release.

---

## Support this project

I build tools to get the best homelab experience I can from what's available and to grow as a programmer along the way. If you'd like to contribute, donations go toward homelab operating costs and subscriptions that keep this tooling maintained. Optional and appreciated.

[Donate with PayPal](https://www.paypal.com/donate/?business=9RHVW92WMWQNL&no_recurring=0&item_name=Optional+donations+help+support+Expected+Behaviors+open+source+work.+Thank+you.&currency_code=USD)
