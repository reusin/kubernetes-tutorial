# An example for adding optional values to Helm Operator

## View Manifest file

The values will eventually be used inside a manifest file in the Helm chart as shown. Note the condition statement, which makes the use of values optional.

```
<removed for brevity>
spec:
  {{ if .Values.spec.secretEnabled }}
    secret: {{.Values.spec.secretName}}
  {{ end }}
<removed for brevity>
```

## View `values.yaml`

The default values can be specified in `values.yaml`

```
<removed for brevity>
  spec:
    secretEnabled: false
    #secretName: ""
<removed for brevity>
```

## Modify `watches.yaml`
The values from custom resource can be overridden in `watches.yaml` file as shown.
```
<removed for brevity>
  overrideValues:    
    spec.secretEnabled: $SECRET_ENABLED
    spec.secretName: $SECRET_NAME
<removed for brevity>
```

## Modify `manager.yaml`
The values for `secretEnabled` and `secretName` can be passed as environment variables to the `manager.yaml`, which creates the `watches.yaml`
```
<removed for brevity>
          env:
            - name: SECRET_ENABLED
              value: true
            - name: SECRET_NAME
              value: mySecret
<removed for brevity>
```
