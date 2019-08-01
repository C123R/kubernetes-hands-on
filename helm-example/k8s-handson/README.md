# nginx: A example chart

This helm chart just provide basic feature of HELM Templating.

- A `ConfigMap` is used to store the files the server will serve.
  ([templates/configmap.yaml](templates/configmap.yaml))
- A `Deployment` is used to create a Replica Set of nginx pods.
  ([templates/deployment.yaml](templates/deployment.yaml))
- A `Service` is used to create a gateway to the pods running in the
  replica set ([templates/service.yaml](templates/service.yaml))

The [values.yaml](values.yaml) exposes a few of the configuration options in the
charts.

The [templates/_helpers.tpl](templates/_helpers.tpl) file contains helper templates. The leading
underscore (`_`) on the filename is semantic. It tells the template renderer
that this file does not contain a manifest. That file declares some
templates that are used elsewhere in the chart.

Helpers (usually called "partials" in template languages) are an
advanced way for developers to structure their templates for optimal
reuse.

You can deploy this chart with `helm install helm-example/nginx`. Or
you can see how this chart would render with `helm install --dry-run --debug helm-example/nginx`.
