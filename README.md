# Issue

<link>

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
