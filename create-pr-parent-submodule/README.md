# GitHub Action: Creates Pull Request when Submodules are Updated
_Forked from [releasehub-com/github-action-create-pr-parent-submodule](https://github.com/releasehub-com/github-action-create-pr-parent-submodule)._

This GitHub action creates a new branch and pull request against the parent repository when a submodule is updated

**The end goal of this action:**

- Create a new branch on the parent repository and get all submodules updates
- Create a pull request from newly created branch
- Add a custom label to the pull request

## How to use

To use this **GitHub** Action you will need to complete the following in the submodule repository:

1. Parent repository has `.gitmodules` configured
2. Store GitHub token into secrets with permissions to write to parent repository
3. Create a new file in your repository, for example called `.github/workflows/submodule-update.yml`
4. Copy the example workflow from below into that new file and modify as needed
5. Commit that file to a new branch
6. Observe the action working

This file should have the following code:

```yaml
jobs:
  submodule_update:
    name: Submodule update
    runs-on: ubuntu-latest
    env:
      PARENT_REPOSITORY: vaultinum/submodule-repository
      CHECKOUT_BRANCH: main
      PR_AGAINST_BRANCH: main
      OWNER: vaultinum
      SUBMODULE_NAME: submodule-name 

    steps:
      - name: Import vaultinum's actions
        uses: actions/checkout@v3
        with:
          repository: vaultinum/actions
          ref: main
          token: ${{ secrets.NPM_PACKAGE_ACCESS }}
          path: .github/actions

      - name: Add PR to parent repo with submodule commits
        uses: ./.github/actions/create-pr-parent-submodule
        with:
          github_token: ${{ secrets.NPM_PACKAGE_ACCESS }}
          parent_repository: ${{ env.PARENT_REPOSITORY }}
          checkout_branch: ${{ env.CHECKOUT_BRANCH}}
          pr_against_branch: ${{ env.PR_AGAINST_BRANCH }}
          owner: ${{ env.OWNER }}
          submodule_name: ${{ env.SUBMODULE_NAME }} 
```