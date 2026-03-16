# Setting namespace in Kustomize is weird

In this repo, I try to exemplify how Kustomize `v5.7.1` and `v5.8.1` handles namespaces for Helm charts differently.

## Usage

```shell
TARGET_DIR="rendered-by-kustomize-$(kustomize version)"
rm -rf "./${TARGET_DIR}"
for k in ns-*; do
    mkdir -p "${TARGET_DIR}/${k}"
    kustomize build --enable-helm "${k}" --output "${TARGET_DIR}/${k}/"
done
```

> [!NOTE]
> Because of a bug in Kustomize v5.7.1, your `helm` binary must be Helm 3 and not Helm 4.

## Background details

In Kustomize `v5.8.1`, a significant bug fix was introduced that makes Kustomize pass down the namespace parameter to Helm rather than naively replacing the `.metadata.namespace` field of all resources (https://github.com/kubernetes-sigs/kustomize/pull/5940).
A very obvious case for when this is required is when `.Release.Namespace` is embedded within strings in the Helm templates, for example inside of a `ConfigMap` resource.
Simply changing `.metadata.namespace` would break the logic for such an application, so I believe this is clearly a good change.

While indeed an important fix, many people are likely to depend on the old broken behavior. If a Helm chart always sets `.metadata.namespace: {{ .Release.Namespace }}` for all namespaced resources and never uses `.Release.Namespace` elsewhere, the use-case would render identically using Kustomize `v5.7.1` and `v5.8.1`. Otherwise, some differences are expected.

_Notable caveats:_
  - Since Kustomize `v5.8.1` never touches the `namespace` after Helm has hydrated its templates, any resources where `.metadata.namespace` is unset by the chart templates will remain unset.
  - If some resources have `.metadata.namespace` set while other resources do not, Kustomize will detect this as the case of having "more than one namespace amongst the resources". If used with the `--output dir/` flag, this will cause Kustomize to to render files with a prefix for each namespace. Somewhat misleadingly, resources with `.metadata.namespace` unset will get file names with the prefix `default_` despite not targeting the literal `default` namespace (but rather whatever namespace is set as the default namespace for the current context when the user runs `kubectl apply`).
