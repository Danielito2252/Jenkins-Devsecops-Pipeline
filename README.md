# 🏗️ Jenkins DevSecOps Pipeline

[![Jenkins](https://img.shields.io/badge/Jenkins-2.440-D24939?logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Docker](https://img.shields.io/badge/Docker-24.0-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Trivy](https://img.shields.io/badge/Security-Trivy_SAST-4AA629?logo=aquasecurity&logoColor=white)](https://aquasecurity.github.io/trivy/)
[![OWASP ZAP](https://img.shields.io/badge/Security-OWASP_DAST-005EA6?logo=owasp&logoColor=white)](https://www.zaproxy.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Este repositorio aloja la configuración centralizada de un pipeline automatizado de Integración Continua enfocado en Seguridad (**DevSecOps**). El objetivo de esta infraestructura es auditar, analizar y garantizar la calidad del código antes de cualquier interacción con producción.

---

## 🔄 Arquitectura e Integración del Ecosistema
Este pipeline funciona como el orquestador principal de nuestro entorno de pruebas. Mantiene una **referencia cruzada** directa con nuestro proyecto estrella de pruebas automatizadas:
👉 Los análisis dinámicos y funcionales de extremo a extremo son ejecutados trayendo en caliente el código de nuestro repositorio madre: **[cypress-e2e-suite](https://github.com/TU_USUARIO/cypress-e2e-suite)**.

### Diagrama Visual del Pipeline
[ Checkout ] ➡️ [ SCA (NPM Audit) ] ➡️ [ SAST (Trivy) ] ➡️ [ Security Gate 🛑 ] ➡️ [ Clone Cypress Suite ] ➡️ [ DAST (OWASP ZAP) ]

## 🚀 Etapas del Pipeline (Jenkinsfile)

El archivo `Jenkinsfile` implementa un flujo declarativo con 5 etapas críticas de validación:

1. **1. Checkout:** Clonado automatizado del código fuente del repositorio del pipeline utilizando `checkout scm`.
2. **2. Security Scan - SCA:** Análisis de composición de software en dependencias mediante `npm audit` para detectar librerías obsoletas o vulnerables.
3. **3. Security Scan - SAST:** Escaneo estático profundo del sistema de archivos mediante un contenedor efímero de `aquasec/trivy`.
4. **4. Security Gate:** Bloque de control lógico en Groovy. Lee el reporte JSON generado por Trivy y **aborta la construcción inmediatamente** si se encuentra más de 0 vulnerabilidades de criticidad `HIGH` o `CRITICAL`.
5. **5. Security Scan - DAST:** Escaneo dinámico en tiempo de ejecución utilizando `owasp/zap2docker-stable` apuntando a un entorno controlado, generando un reporte interactivo en HTML.

---

## 📊 Evidencias de Ejecución (Datos Reales)

### 1. Visualización del Flujo (Stage View en Jenkins)
*Aquí se evidencia el correcto paso secuencial por las etapas requeridas:*
![Jenkins Stage View](ruta-a-tu-captura-del-stage-view.png)

### 2. Control del Security Gate (Consola de Jenkins)
*Validación lógica donde el filtro analiza el JSON de Trivy y aprueba o rechaza la compilación:*
![Security Gate Log](ruta-a-tu-captura-del-log.png)

### 3. Reporte Dinámico Generado (OWASP ZAP DAST)
*Captura de los resultados interactivos generados en tiempo de ejecución:*
![OWASP ZAP Report](ruta-a-tu-captura-del-zap-report.png)

---

## 🛠️ Infraestructura de Ejecución
El entorno se despliega localmente de manera reproducible utilizando contenedores. Jenkins cuenta con acceso al socket nativo mediante volumen:
- Mapeo de volumen: `/var/run/docker.sock:/var/run/docker.sock` (Docker-out-of-Docker).
- Plugin requerido: `Pipeline Utility Steps` (para lectura nativa del objeto JSON).