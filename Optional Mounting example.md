# Example of altering deployments to include optional secrets and configmaps from `values.yaml`


## `values.yaml`

In the `values.yaml` file, the names of secrets and configmap which are to be mounted can be provided as such. Additional flag field, `mountEnabled` is created for the case when no mounts are used (A design to avoid errors in the `deployment.yaml`). The `values.yaml` of an helm operator is created from the Custom Resource, and the following fields can be provided to the `spec` section of the custom resource.

```
mountEnabled: true

secretMount:
  keyStoreSecret: "keystore"
  anotherSecret: ""

configMapMount:
  anotherconfigMap: "another"
```

## `deployment.yaml`

The chart template allows us to use conditional statements to validate non-empty fields from the Custom resource/values.yaml file. The chart templates must be altered for the approproate manifest files in `templates/`.

```
<removed for brevity>
      
      containers:
        
        <removed for brevity>
        
        #Check is mountEnabled is true
        {{- if .Values.mountEnabled }}
        volumeMounts:
        #Check if keyStoreSecret is not empty
        {{- if .Values.secretMount.keyStoreSecret }}
        - name: key-store
          mountPath: /resources/certs/
        {{- end }}
        #Check if anotherconfigMap is not empty
        {{- if .Values.secretMount.anotherconfigMap }}
        - name: another-cm
          mountPath: /resources/config/
        {{- end }}
        {{- end }}
      volumes:
        # Check if keyStoreSecret is not empty
        {{- if .Values.secretMount.keyStoreSecret }}
        - name: key-store
          secret:
            secretName: {{ .Values.secretMount.keyStoreSecret }}
        {{- end }}
        # Check if anotherconfigMap is not empty
        {{- if .Values.configMapMount.anotherconfigMap }}
        - name: another-cm
          configMap:
            name: {{ .Values.configMapMount.anotherconfigMap }}
        {{- end }}

```

