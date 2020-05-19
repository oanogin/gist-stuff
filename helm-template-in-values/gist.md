Just add this in _helpers.tpl:
```go
{{/*
Renders a value that contains template.
Usage:
{{ include "replaceMe.tplValue" ( dict "value" .Values.path.to.the.Value "context" $) }}
*/}}
{{- define "replaceMe.tplValue" -}}
{{- if typeIs "bool" .value }}
{{- tpl (.value | quote) .context }}
{{- else if typeIs "string" .value }}
{{- tpl (.value | quote) .context }}
{{- else }}
{{- tpl (.value | toYaml) .context }}
{{- end }}
{{- end -}}
```

Then set anything in values as (env block ex.):
```yaml
image:
  repository: nginx
  tag: "stable"

env:
  IMAGE: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
  HOST: "{{ (index .Values.ingress.hosts 0).host }}"

ingress:
  hosts:
    - host: chart-example.local
      paths: []
```

NOTE: "" around template required to determine that it is a string and it needs to be parsed

And usage in templates/:
```go
containers:
  - name: test
    env:
      {{- range $key, $value := .Values.env }}
      - name: {{ $key }}
        value: {{ include "replaceMe.tplValue" ( dict "value" $value "context" $) }}
      {{- end }}
```

Result:
![ResultImage](https://raw.githubusercontent.com/oanogin/gist-stuff/master/helm-template-in-values/result.png)