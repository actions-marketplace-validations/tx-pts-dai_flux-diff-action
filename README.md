# Flux diff Github Action

This Github Action runs `flux diff kustomization` and outputs the result in the github action step summary.

The command `flux diff kustomization` returns all the changes that will be applied at the next flux reconcile. The idea is to run this action when PRs are created having a preview of the differences with the currently deployed resources before merging into `main`.

# Requirements:

- kubectl (configured with a valid kubeconfig)
- yq
- flux

# Usage

## Basic usage:

```yaml
      - uses: tx-pts-dai/flux-diff-action@main
        with:
          cluster-kustomization-path: clusters/prod
          kustomization-directories: folder-1 folder-2 folder-3
```

## Extended example

In this example several other actions are used to: 

- get the list of modified directory and pass them to the `tx-pts-dai/flux-diff-action`
- configure aws credentials (assuming AWS OIDC authentication for github workflows is in place)
- get the kubeconfig using the `aws cli`
- install the missing required tools (`kubectl`, `flux`)

```yaml
      - name: Get changed directories
        id: changed-directories
        uses: tj-actions/changed-files@v35
        with:
          dir_names: "true" # only get the directory names
          dir_names_max_depth: 2

      - uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-region: <<my-region>>
          role-to-assume: <<my-aws-role-arn>>

      - name: Load Kubeconfig
        id: kubeconfig
        run: |
          aws eks --region <<my-region>> update-kubeconfig --name <<my-cluster-name>>

      - uses: alexellis/setup-arkade@v2
      - uses: alexellis/arkade-get@master
        with:
          kubectl: v1.24.12
          flux: v0.41.2

      - uses: tx-pts-dai/flux-diff-action@main
        with:
          cluster-kustomization-path: clusters/prod
          kustomization-directories: ${{ steps.changed-directories.outputs.all_changed_files }}
```