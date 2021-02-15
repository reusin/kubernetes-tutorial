# Example of altering deployments to include optional secrets and configmaps from `values.yaml`


## `values.yaml`

```
mountEnabled: true

secretMount:
  keyStoreSecret: "keystore"
  anotherSecret: ""

configMapMount:
  anotherconfigMap: "another"
```

## `deployment.yaml` (Note the specification)

```
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- if .Values.mountEnabled }}
          volumeMounts:
          {{- if .Values.secretMount.keyStoreSecret }}
          - name: key-store
            mountPath: /resources/certs/
          {{- end }}
          {{- end }}
      volumes:
        {{- if .Values.secretMount.keyStoreSecret }}
        - name: key-store
          secret:
            secretName: {{ .Values.secretMount.keyStoreSecret }}
        {{- end }}
        {{- if .Values.configMapMount.anotherconfigMap }}
        - name: another-cm
          configMap:
            name: {{ .Values.configMapMount.anotherconfigMap }}
        {{- end }}

```

