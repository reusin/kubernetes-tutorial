# Creating an optional configMap

The creation of a configMap can be condition based with the help of chart template as shown. Where these values are obtained from `values.yaml`.

```
{{- if .Values.newSpecificConfig -}}
apiVersion: v1
  data:
    {{- range .Values }}
    {{ .key | nindent 4}}: {{ .val }}
    {{- end }}
    
  kind: ConfigMap
  metadata:
    name: {{ .Values.newSpecificConfig.name }}
    namespace: default
{{- end }}
```

## Contents of values.yaml

In this example the literals to populate the configMap are mentioned as a list, as shown.

```
newSpecificConfig:
  - key: "fileName1"
    value: "foo.txt"
  - key: "fileName2"
    value: "bar.txt"
```
