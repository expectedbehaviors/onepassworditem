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
- Items and vaults exist in 1Password; paths use the form `vaults/<vault_id_or_title>/items/<item_id_or_title>`.

## Values

| Key            | Type    | Default | Description |
|----------------|--------|--------|-------------|
| `defaultVault` | string | `""`   | Optional. Default vault (id or title) used when an item does not set `vault` and `item` is not a full path. Enables short form: `item: "myapp"` → `vaults/<defaultVault>/items/myapp`. |
| `enabled`      | bool   | `true` | If `false`, no OnePasswordItem resources are rendered. |
| `items`        | list   | `[]`   | List of 1Password items to sync. See below. |

Each entry in `items`:

| Key               | Type   | Required | Description |
|-------------------|--------|----------|-------------|
| `item`            | string | yes      | Either a **full path** (`vaults/<vault>/items/<item>`) or the **item part only** (e.g. `myapp`). If not a full path, the path is built from `vault` or `defaultVault` + `item`. |
| `vault`           | string | no       | Vault (id or title) for this item. Used only when `item` is not a full path. Falls back to top-level `defaultVault` if unset. |
| `name`            | string | yes      | Name of the Kubernetes Secret to create. |
| `type`            | string | no       | Kubernetes Secret type (default `Opaque`). |
| `annotations`     | map    | no       | Annotations to apply to the Secret (post-install hook). |
| `labels`          | map    | no       | Labels to apply to the Secret (post-install hook). |

## Secret annotations and labels

If you need the Secret to be annotated or labeled (e.g. for [kubernetes-replicator](https://github.com/mittwald/kubernetes-replicator)), add **`annotations`** and/or **`labels`** to any item in **`items`**. A post-install/upgrade Helm hook (Job + RBAC) patches the Secret after the operator creates it. No hook is created if no item has `annotations` or `labels`.

**Full path (backward compatible):**

```yaml
items:
  - item: vaults/Kubernetes/items/myapp
    name: myapp
    type: Opaque
```

**Vault + item (good for autofill / default vault):**

```yaml
defaultVault: Kubernetes
items:
  - item: myapp
    name: myapp
    type: Opaque
  - vault: AnotherVault
    item: other-secret
    name: other-secret
```

When `item` does not start with `vaults/`, the path is built as `vaults/<vault>/items/<item>` where `vault` is the entry's `vault` or, if unset, the top-level `defaultVault`. This lets you set a default vault once and only specify the item (and optional per-item vault override).

Example with annotations (e.g. for kubernetes-replicator):

```yaml
defaultVault: Kubernetes
items:
  - item: myapp
    name: myapp
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
    version: "1.0.0"
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
