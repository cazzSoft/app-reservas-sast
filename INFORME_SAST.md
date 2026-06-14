# Practica SAST - App Reservas

Fecha de ejecucion: 13 de junio de 2026  
Repositorio analizado: https://github.com/agcudco/app-reservas  
Autores: Vera Zambrano Jorge Maron (`maronv64`) y Cristhian Alfredo Zambrano Zambrano (`cazzSoft`)  
Proyecto: ReservasEC, aplicacion fullstack con microservicios Node.js/Express, frontend Next.js, MongoDB y Docker.

## 1. Preparacion del entorno

Se clono el repositorio original:

```powershell
git clone https://github.com/agcudco/app-reservas.git
cd app-reservas
```

Se instalo Semgrep en un entorno virtual local para no modificar Python global:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install semgrep
```

Como el entorno no tenia permiso para escribir en `C:\Users\kevin\.semgrep`, se redirigieron los archivos locales de Semgrep:

```powershell
$env:SEMGREP_LOG_FILE='C:\MGSRT\Practicas\SAST\app-reservas\.semgrep-local\semgrep.log'
$env:SEMGREP_SETTINGS_FILE='C:\MGSRT\Practicas\SAST\app-reservas\.semgrep-local\settings.yaml'
$env:SEMGREP_VERSION_CACHE_PATH='C:\MGSRT\Practicas\SAST\app-reservas\.semgrep-local\version-cache'
```

## 2. Analisis 1: Semgrep local con configuracion auto

Comando ejecutado:

```powershell
.\.venv\Scripts\semgrep.exe scan --config auto --json-output semgrep-auto.json .
```

Resultado general:

| Metrica | Resultado |
|---|---:|
| Archivos escaneados | 80 |
| Reglas ejecutadas | 248 |
| Hallazgos | 13 |
| Hallazgos blocking | 13 |
| Lineas parseadas | ~100% |

Hallazgos por tipo:

| Tipo de hallazgo | Cantidad | Riesgo |
|---|---:|---|
| Dockerfile sin usuario no root | 5 | Los contenedores se ejecutan como root, aumentando el impacto si un atacante compromete el servicio. |
| Peticiones HTTP sin cifrar | 5 | Comunicaciones internas por `http://`, susceptibles a interceptacion o manipulacion si la red no esta protegida. |
| Falta de middleware CSRF en Express | 3 | Las rutas Express no evidencian proteccion CSRF; puede ser relevante si se usan cookies/sesiones o navegadores. |

Hallazgos relevantes:

| Archivo | Linea | Hallazgo |
|---|---:|---|
| `auth-service/Dockerfile` | 14 | No se define `USER` no root. |
| `booking-service/Dockerfile` | 12 | No se define `USER` no root. |
| `frontend/Dockerfile` | 26 | No se define `USER` no root. |
| `notification-service/Dockerfile` | 13 | No se define `USER` no root. |
| `user-service/Dockerfile` | 13 | No se define `USER` no root. |
| `auth-service/src/controllers/auth.controller.js` | 16 | Peticion HTTP interna a `user-service`. |
| `booking-service/src/routes/booking.routes.js` | 61 | Peticion HTTP interna a `notification-service`. |
| `booking-service/src/routes/booking.routes.js` | 102 | Peticion HTTP interna a `notification-service`. |
| `auth-service/src/app.js` | 8 | No se detecta middleware CSRF. |
| `booking-service/src/app.js` | 7 | No se detecta middleware CSRF. |
| `notification-service/index.js` | 4 | No se detecta middleware CSRF. |

## 3. Analisis 2: Semgrep local con OWASP Top 10

Comando ejecutado:

```powershell
.\.venv\Scripts\semgrep.exe scan --config p/owasp-top-ten --json-output semgrep-owasp.json .
```

Resultado general:

| Metrica | Resultado |
|---|---:|
| Archivos escaneados | 82 |
| Reglas ejecutadas | 103 |
| Hallazgos | 8 |
| Hallazgos blocking | 8 |
| Lineas parseadas | ~100% |

Hallazgos por tipo:

| Tipo de hallazgo | Cantidad | Riesgo |
|---|---:|---|
| Dockerfile sin usuario no root | 5 | Riesgo de privilegios excesivos dentro del contenedor. |
| Peticiones HTTP sin cifrar | 3 | Riesgo de transporte inseguro entre servicios. |

Observacion: el pack OWASP Top 10 fue mas especifico y reporto menos hallazgos que `--config auto`. No incluyo los avisos de CSRF detectados por el analisis automatico.

## 4. Analisis 3: Semgrep con cuenta Developer

Estado actual: completado.

Se conecto el repositorio propio `cazzSoft/app-reservas-sast` con Semgrep Cloud y tambien se inicio sesion desde la CLI con:

```powershell
.\.venv\Scripts\semgrep.exe login
.\.venv\Scripts\semgrep.exe ci
```

Resultado de Semgrep Cloud:

| Metrica | Resultado |
|---|---:|
| Repositorio | `cazzSoft/app-reservas-sast` |
| Rama | `main` |
| Tipo de scan | `full` |
| Estado | `Completed` |
| Duracion | 46s |
| Total findings | 168 |
| Code findings | 20 |
| Secrets findings | 0 |
| Supply Chain findings | 148 |

Resultado de `semgrep ci` desde consola:

| Metrica | Resultado |
|---|---:|
| Estado | CI scan completed successfully |
| Hallazgos | 166 |
| Hallazgos blocking | 0 |
| Reglas ejecutadas | 124441 |
| Archivos escaneados | 79 |
| Lineas parseadas | ~100% |
| Motor | Semgrep Pro Engine 1.166.0 |

Semgrep Developer agrego valor frente al analisis local porque ejecuto reglas Pro y analisis de Supply Chain, ademas de registrar los resultados en el dashboard web del proyecto.

## 5. Analisis 4: SonarQube Cloud

Estado actual: completado.

Se importo el repositorio propio `cazzSoft/app-reservas-sast` en SonarQube Cloud y se ejecuto el analisis del proyecto.

Resultado observado en el resumen de SonarQube Cloud:

| Metrica | Resultado |
|---|---|
| Proyecto | `app-reservas-sast` |
| Rama | `main` |
| Lineas de codigo | 1.9k |
| Ultimo analisis | Ejecutado correctamente |
| Quality Gate | Not computed |
| Security | 16 open issues, calificacion E |
| Reliability | 13 open issues, calificacion B |
| Maintainability | 24 open issues, calificacion A |
| Security Hotspots | 19 |
| Accepted issues | 0 |
| Coverage | No configurado |
| Duplications | 5.1% en 2.1k lineas |

SonarQube Cloud aporto una vista mas amplia de calidad, mostrando seguridad, confiabilidad, mantenibilidad, duplicacion y estado del Quality Gate. En este caso, el Quality Gate aparece como `Not computed`, por lo que se recomienda ejecutar un nuevo analisis o revisar la configuracion del Quality Gate para que el estado quede calculado.

## 6. Comparativa

| Herramienta / modo | Hallazgos obtenidos | Fortalezas | Limitaciones |
|---|---:|---|---|
| Semgrep `--config auto` | 13 | Mayor cobertura en este proyecto; detecto Dockerfile root, HTTP inseguro y ausencia de CSRF. | Puede incluir avisos que requieren interpretacion contextual, como CSRF en APIs que no usan cookies. |
| Semgrep `p/owasp-top-ten` | 8 | Enfocado en categorias OWASP; resultados mas concretos para seguridad web. | Menor cobertura; no reporto algunos avisos de auditoria detectados por `auto`. |
| Semgrep Developer | 166 por CLI / 168 en Cloud | Dashboard, historico, reglas Pro, politicas por proyecto y Supply Chain. | Requiere cuenta, GitHub App y login/token. |
| SonarQube Cloud | 53 issues abiertos visibles por categoria | Excelente para calidad integral: seguridad, confiabilidad, mantenibilidad, duplicacion y quality gate. | El Quality Gate quedo como `Not computed` y la cobertura no esta configurada. |

## 7. Cual es mejor y por que

Para esta practica, el mejor analisis tecnico inicial fue `semgrep scan --config auto`, porque encontro mas tipos de riesgos concretos en el proyecto: ejecucion de contenedores como root, transporte HTTP sin cifrado y posibles ausencias de proteccion CSRF.

Sin embargo, para un flujo profesional completo, SonarQube Cloud suele ser mas fuerte como plataforma de seguimiento continuo, porque no solo se centra en vulnerabilidades sino tambien en mantenibilidad, bugs, duplicacion, deuda tecnica y quality gates. Semgrep es mas directo y util para reglas de seguridad en codigo; SonarQube Cloud es mejor para gobernanza general de calidad y seguridad.

Mi conclusion: usaria Semgrep para deteccion rapida y especifica de patrones de seguridad, y SonarQube Cloud para seguimiento continuo del estado del proyecto en el tiempo. No son excluyentes; se complementan.

## 8. Reflexion final

El analisis muestra que una aplicacion puede funcionar correctamente y aun asi tener debilidades de seguridad relevantes. En este caso, los principales riesgos aparecen en la configuracion de despliegue y comunicacion entre servicios: contenedores ejecutados como root y uso de HTTP interno. Tambien aparecen advertencias sobre CSRF en servicios Express, que deben evaluarse segun el mecanismo real de autenticacion y exposicion de cada API.

La practica confirma que el SAST ayuda a descubrir problemas temprano, antes del despliegue. Tambien demuestra que ninguna herramienta debe tomarse como verdad absoluta: los hallazgos deben revisarse, priorizarse y contextualizarse. La mejor estrategia es combinar herramientas, comparar resultados y convertir los hallazgos en acciones concretas de remediacion.

## 9. Acciones recomendadas de remediacion

1. Agregar usuarios no root en todos los Dockerfile.
2. Revisar la comunicacion HTTP entre microservicios y considerar HTTPS/mTLS o una red interna controlada con politicas claras.
3. Evaluar CSRF segun el tipo de autenticacion. Si se usan cookies, agregar proteccion CSRF; si se usa JWT por header, documentar por que el riesgo es menor.
4. Importar el repositorio propio en SonarQube Cloud.
5. Registrar metricas de SonarQube Cloud: bugs, vulnerabilities, security hotspots, code smells, duplicacion y quality gate.
6. Adjuntar capturas de los dashboards de Semgrep y SonarQube Cloud en la presentacion final.

## 10. Archivos generados

| Archivo | Contenido |
|---|---|
| `semgrep-auto.json` | Resultado completo del analisis Semgrep `--config auto`. |
| `semgrep-owasp.json` | Resultado completo del analisis Semgrep `p/owasp-top-ten`. |
| `.semgrep-local/` | Logs/configuracion local de Semgrep para este entorno. |
