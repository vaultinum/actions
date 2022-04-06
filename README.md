# Vaultinum's actions

### How to use an action

1. Add a step with `action/checkout@v2` to import this repository
2. Add a second step to define the action file

```yaml
      - name: Import vaultinum's actions
        uses: actions/checkout@v2
        with:
          repository: vaultinum/actions
          ref: main
          token: ${{ secrets.NPM_PACKAGE_ACCESS }}
          path: .github/actions
      - name: Do an action
        uses: ./.github/actions/folder-action
```