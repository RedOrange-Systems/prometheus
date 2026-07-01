<h1 align="center" style="border-bottom: none">
    <a href="https://prometheus.io" target="_blank"><img alt="Prometheus" src="/documentation/images/prometheus-logo.svg"></a><br>Prometheus
</h1>

<p align="center">Visita <a href="https://prometheus.io" target="_blank">prometheus.io</a> para la documentación completa,
ejemplos y guías.</p>

<div align="center">

[![CI](https://github.com/prometheus/prometheus/actions/workflows/ci.yml/badge.svg)](https://github.com/prometheus/prometheus/actions/workflows/ci.yml)
[![Docker Repository on Quay](https://quay.io/repository/prometheus/prometheus/status)][quay]
[![Docker Pulls](https://img.shields.io/docker/pulls/prom/prometheus.svg?maxAge=604800)][hub]
[![Go Report Card](https://goreportcard.com/badge/github.com/prometheus/prometheus)](https://goreportcard.com/report/github.com/prometheus/prometheus)
[![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/486/badge)](https://bestpractices.coreinfrastructure.org/projects/486)
[![govulncheck](https://github.com/prometheus/prometheus/actions/workflows/govulncheck.yml/badge.svg?event=schedule)](https://github.com/prometheus/prometheus/actions/workflows/govulncheck.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/prometheus/prometheus/badge)](https://securityscorecards.dev/viewer/?uri=github.com/prometheus/prometheus)
[![CLOMonitor](https://img.shields.io/endpoint?url=https://clomonitor.io/api/projects/cncf/prometheus/badge)](https://clomonitor.io/projects/cncf/prometheus)
[![Fuzzing Status](https://oss-fuzz-build-logs.storage.googleapis.com/badges/prometheus.svg)](https://bugs.chromium.org/p/oss-fuzz/issues/list?sort=-opened&can=1&q=proj:prometheus)

</div>

Prometheus, un proyecto de la [Cloud Native Computing Foundation](https://cncf.io/), es un sistema de monitorización de sistemas y servicios. Recopila métricas
de los objetivos configurados en intervalos determinados, evalúa expresiones de reglas,
muestra los resultados y puede activar alertas cuando se observan condiciones específicas.

Las características que distinguen a Prometheus de otros sistemas de métricas y monitorización son:

* Un modelo de datos **multidimensional** (series temporales definidas por el nombre de la métrica y un conjunto de dimensiones clave/valor)
* PromQL, un **lenguaje de consulta potente y flexible** que aprovecha esta dimensionalidad
* Sin dependencia de almacenamiento distribuido; **los nodos de servidor individuales son autónomos**
* Un **modelo de extracción (pull) por HTTP** para la recolección de series temporales
* El **envío de series temporales (push)** es compatible mediante una pasarela intermedia para trabajos por lotes
* Los objetivos se descubren mediante **descubrimiento de servicios** o **configuración estática**
* Múltiples modos de **compatibilidad con gráficos y paneles de control**
* Compatibilidad con **federación** jerárquica y horizontal

## Descripción general de la arquitectura

![Descripción general de la arquitectura](documentation/images/architecture.svg)

## Instalación

Existen varias formas de instalar Prometheus.

### Binarios precompilados

Los binarios precompilados de las versiones publicadas están disponibles en la
[sección de *descargas*](https://prometheus.io/download/)
en [prometheus.io](https://prometheus.io). Usar el binario de la última versión de producción
es la forma recomendada de instalar Prometheus.
Consulta el capítulo [Instalación](https://prometheus.io/docs/introduction/install/)
en la documentación para conocer todos los detalles.

### Imágenes Docker

Las imágenes Docker están disponibles en [Quay.io](https://quay.io/repository/prometheus/prometheus) o en [Docker Hub](https://hub.docker.com/r/prom/prometheus/).

Puedes lanzar un contenedor de Prometheus para probarlo con

```bash
docker run --name prometheus -d -p 127.0.0.1:9090:9090 prom/prometheus
```

Prometheus estará ahora accesible en <http://localhost:9090/>.

### Compilación desde el código fuente

Para compilar Prometheus desde el código fuente, necesitas:

* Go: La versión especificada en [go.mod](./go.mod) o superior.
* NodeJS: La versión especificada en [.nvmrc](./web/ui/.nvmrc) o superior.
* npm: Versión 10 o superior (verifica con `npm --version` y [aquí](https://www.npmjs.com/)).

Empieza clonando el repositorio:

```bash
git clone https://github.com/prometheus/prometheus.git
cd prometheus
```

Puedes usar la herramienta `go` para compilar e instalar los binarios `prometheus`
y `promtool` en tu `GOPATH`:

```bash
go install github.com/prometheus/prometheus/cmd/...
prometheus --config.file=your_config.yml
```

*Sin embargo*, al usar `go install` para compilar Prometheus, Prometheus esperará poder
leer sus recursos web desde directorios del sistema de archivos local bajo `web/ui/static`. Para que
estos recursos se encuentren, tendrás que ejecutar Prometheus desde la raíz del repositorio
clonado. Ten en cuenta también que este directorio no incluye la interfaz React a menos que se haya
compilado explícitamente usando `make assets` o `make build`.

Puedes encontrar un ejemplo del archivo de configuración anterior [aquí.](https://github.com/prometheus/prometheus/blob/main/documentation/examples/prometheus.yml)

También puedes compilar usando `make build`, que incluirá los recursos web compilados para que
Prometheus pueda ejecutarse desde cualquier lugar:

```bash
make build
./prometheus --config.file=your_config.yml
```

El Makefile ofrece varios objetivos (targets):

* *build*: compila los binarios `prometheus` y `promtool` (incluye la compilación e incorporación de los recursos web)
* *test*: ejecuta las pruebas
* *test-short*: ejecuta las pruebas cortas
* *format*: da formato al código fuente
* *vet*: verifica el código fuente en busca de errores comunes
* *assets*: compila la interfaz React

### Complementos de descubrimiento de servicios

Prometheus incluye muchos complementos de descubrimiento de servicios. Puedes personalizar
qué descubrimientos de servicios se incluyen en tu compilación usando etiquetas (tags) de compilación de Go.

Para excluir descubrimientos de servicios al compilar con `make build`, añade las etiquetas
deseadas al archivo `.promu.yml` bajo `build.tags.all`:

```yaml
build:
    tags:
        all:
            - netgo
            - builtinassets
            - remove_all_sd           # Excluir todos los SD opcionales
            - enable_kubernetes_sd    # Volver a habilitar solo kubernetes
```

Luego ejecuta `make build` como de costumbre. Alternativamente, al usar `go build` directamente:

```bash
go build -tags "remove_all_sd,enable_kubernetes_sd" ./cmd/prometheus
```

Etiquetas de compilación disponibles:
* `remove_all_sd` - Excluye todos los descubrimientos de servicios opcionales (mantiene file_sd, static_sd y http_sd)
* `enable_<name>_sd` - Vuelve a habilitar un SD específico cuando se usa `remove_all_sd`

Si agregas complementos externos al árbol del proyecto (out-of-tree), lo cual no respaldamos por el momento,
podrían ser necesarios pasos adicionales para ajustar los archivos `go.mod` y `go.sum`. Como
siempre, ten mucho cuidado al cargar código de terceros.

### Compilación de la imagen Docker

Puedes compilar una imagen Docker localmente con los siguientes comandos:

```bash
make promu
promu crossbuild -p linux/amd64
make common-docker-amd64
```

El objetivo `make docker` está destinado únicamente para su uso en nuestro sistema de CI y no
producirá una imagen totalmente funcional si se ejecuta localmente.

## Usar Prometheus como una biblioteca de Go

Dentro del proyecto Prometheus, repositorios como [prometheus/common](https://github.com/prometheus/common) y
[prometheus/client-golang](https://github.com/prometheus/client-golang) están diseñados como bibliotecas reutilizables.

El repositorio [prometheus/prometheus](https://github.com/prometheus/prometheus) compila un programa independiente y no
está diseñado para usarse como biblioteca. Somos conscientes de que la gente usa partes de él como tal,
y no ponemos ningún inconveniente deliberado en el camino, pero queremos que sepas que no se ha tenido
cuidado alguno para que funcione bien como biblioteca. Por ejemplo, podrías encontrar errores que solo
surgen cuando se usa como biblioteca.

### Remote Write

Estamos publicando nuestro protobuf de Remote Write de forma independiente en
[buf.build](https://buf.build/prometheus/prometheus/assets).

Puedes usarlo como una biblioteca:

```shell
go get buf.build/gen/go/prometheus/prometheus/protocolbuffers/go@latest
```

Esto es experimental.

### Base de código de Prometheus

Para cumplir con las reglas de [go mod](https://go.dev/ref/mod#versions),
los números de versión de Prometheus no coinciden exactamente con las versiones de los módulos de Go.

Para las versiones de Prometheus v3.y.z, estamos publicando etiquetas v0.3y.z equivalentes. La y en v0.3y.z siempre se completa con dos dígitos, con un cero inicial si es necesario.

Por lo tanto, un usuario que quisiera usar Prometheus v3.0.0 como biblioteca podría hacer:

```shell
go get github.com/prometheus/prometheus@v0.300.0
```

Para las versiones de Prometheus v2.y.z, publicamos las etiquetas v0.y.z equivalentes.

Por lo tanto, un usuario que quisiera usar Prometheus v2.35.0 como biblioteca podría hacer:

```shell
go get github.com/prometheus/prometheus@v0.35.0
```

Esta solución deja claro que podríamos romper nuestras APIs internas de Go entre
versiones menores orientadas al usuario, ya que [se permiten cambios incompatibles en la versión
principal cero](https://semver.org/#spec-item-4).

## Desarrollo de la interfaz React

Para más información sobre cómo compilar, ejecutar y desarrollar la interfaz basada en React, consulta el [README.md](web/ui/README.md) de la aplicación React.

## Más información

* La documentación de Godoc está disponible a través de [pkg.go.dev](https://pkg.go.dev/github.com/prometheus/prometheus). Debido a particularidades de los Módulos de Go, v3.y.z se mostrará como v0.3y.z (la y en v0.3y.z siempre se completa con dos dígitos, con un cero inicial si es necesario), mientras que v2.y.z se mostrará como v0.y.z.
* Consulta la [página de la Comunidad](https://prometheus.io/community) para saber cómo contactar a los desarrolladores y usuarios de Prometheus a través de varios canales de comunicación.

## Contribuir

Consulta [CONTRIBUTING.md](https://github.com/prometheus/prometheus/blob/main/CONTRIBUTING.md)

## Licencia

Apache License 2.0, consulta [LICENSE](https://github.com/prometheus/prometheus/blob/main/LICENSE).

[hub]: https://hub.docker.com/r/prom/prometheus/
[quay]: https://quay.io/repository/prometheus/prometheus
