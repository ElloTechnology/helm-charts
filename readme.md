# Helm charts

A collection of charts for Ello Apps.

Checkout [this doc](https://citizix.com/how-to-create-and-host-helm-chart-repository-on-github-pages/) on how it is set up and released.

To validate

```sh
helm template neural-tracker-inference ~/Code/ello/helm-charts/charts/app \
  --version 0.1.0 \
  --namespace stage \
  -f deployments/helm/stage-eu-west-1-neural-tracker-inference.yml \
  --set image.tag=latest \
  --debug
```
