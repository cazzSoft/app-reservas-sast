# Practica SAST - App Reservas

Fecha de ejecucion: 13 de junio de 2026  
Repositorio analizado: https://github.com/agcudco/app-reservas  
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

Estado actual: pendiente de credenciales.

Para completarlo se necesita iniciar sesion en Semgrep Developer o usar un token de Semgrep. Los comandos recomendados son:

```powershell
$env:SEMGREP_APP_TOKEN='TOKEN_DE_SEMGREP'
$env:SEMGREP_LOG_FILE='C:\MGSRT\Practicas\SAST\app-reservas\.semgrep-local\semgrep.log'
$env:SEMGREP_SETTINGS_FILE='C:\MGSRT\Practicas\SAST\app-reservas\.semgrep-local\settings.yaml'
$env:SEMGREP_VERSION_CACHE_PATH='C:\MGSRT\Practicas\SAST\app-reservas\.semgrep-local\version-cache'
.\.venv\Scripts\semgrep.exe ci --json-output semgrep-developer.json
```

Lo esperable es que Semgrep Developer agregue valor con politicas centralizadas, vista web, historico del proyecto, posibilidad de priorizacion y, segun la configuracion de la cuenta, reglas adicionales o gestion de hallazgos. Para presentacion, se debe incluir captura del dashboard de Semgrep con el proyecto analizado.

## 5. Analisis 4: SonarQube Cloud

Estado actual: pendiente de repositorio propio y credenciales.

La actividad exige subir el proyecto a un repositorio propio. Para completarlo se necesita:

| Requisito | Detalle |
|---|---|
| Repositorio propio | URL del repo de GitHub donde se subira `app-reservas`. |
| Cuenta SonarQube Cloud | Acceso a https://sonarcloud.io o SonarQube Cloud actual. |
| Organizacion y project key | Datos generados al crear/importar el proyecto en SonarQube Cloud. |
| Token | `SONAR_TOKEN` para ejecutar el analisis o configurar GitHub Actions. |

Comando base con scanner, una vez creado el proyecto en SonarQube Cloud:

```powershell
sonar-scanner `
  -Dsonar.projectKey=TU_PROJECT_KEY `
  -Dsonar.organization=TU_ORGANIZATION `
  -Dsonar.sources=. `
  -Dsonar.host.url=https://sonarcloud.io `
  -Dsonar.token=TU_SONAR_TOKEN
```

Para la entrega, se debe registrar:

| Metrica de SonarQube Cloud | Resultado a completar |
|---|---|
| Bugs | Pendiente |
| Vulnerabilities | Pendiente |
| Security Hotspots | Pendiente |
| Code Smells | Pendiente |
| Duplications | Pendiente |
| Quality Gate | Pendiente |

## 6. Comparativa

| Herramienta / modo | Hallazgos obtenidos | Fortalezas | Limitaciones |
|---|---:|---|---|
| Semgrep `--config auto` | 13 | Mayor cobertura en este proyecto; detecto Dockerfile root, HTTP inseguro y ausencia de CSRF. | Puede incluir avisos que requieren interpretacion contextual, como CSRF en APIs que no usan cookies. |
| Semgrep `p/owasp-top-ten` | 8 | Enfocado en categorias OWASP; resultados mas concretos para seguridad web. | Menor cobertura; no reporto algunos avisos de auditoria detectados por `auto`. |
| Semgrep Developer | Pendiente | Dashboard, historico, politicas por proyecto y mejor gestion para equipo. | Requiere cuenta/token y configuracion del proyecto. |
| SonarQube Cloud | Pendiente | Excelente para calidad integral: bugs, vulnerabilidades, hotspots, code smells, duplicacion y quality gate. | Requiere subir a repo propio y configurar proyecto/token. |

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
4. Configurar Semgrep Developer con token y ejecutar `semgrep ci`.
5. Subir el proyecto a un repositorio propio e importar en SonarQube Cloud.
6. Adjuntar capturas de los dashboards de Semgrep y SonarQube Cloud en la presentacion final.

## 10. Archivos generados

| Archivo | Contenido |
|---|---|
| `semgrep-auto.json` | Resultado completo del analisis Semgrep `--config auto`. |
| `semgrep-owasp.json` | Resultado completo del analisis Semgrep `p/owasp-top-ten`. |
| `.semgrep-local/` | Logs/configuracion local de Semgrep para este entorno. |
