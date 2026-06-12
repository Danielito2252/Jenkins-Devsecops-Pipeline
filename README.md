# 🏗️ Jenkins DevSecOps CI/CD Pipeline Orquestador

🗂️ **Repositorio 2 de 3** | Ecosistema de Automatización e Ingeniería de Software

[![Jenkins](https://img.shields.io/badge/Jenkins-2.440-D24939?style=for-the-badge&logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Docker](https://img.shields.io/badge/Docker-24.0-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Trivy](https://img.shields.io/badge/Security-Trivy_SAST-4AA629?style=for-the-badge&logo=aquasecurity&logoColor=white)](https://aquasecurity.github.io/trivy/)
[![OWASP ZAP](https://img.shields.io/badge/Security-OWASP_DAST-005EA6?style=for-the-badge&logo=owasp&logoColor=white)](https://www.zaproxy.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

---

## 📝 Descripción del Proyecto

Este repositorio aloja la configuración centralizada de un **Pipeline Declarativo de DevSecOps** desarrollado en Jenkins. La infraestructura está diseñada bajo el principio de "Seguridad por Defecto" (*Security by Design*), actuando como un orquestador inteligente que audita, analiza y condiciona el despliegue de software basándose en métricas estrictas de riesgo.

Este componente forma parte de un ecosistema interconectado mediante referencias cruzadas, interactuando directamente con nuestra suite de pruebas de caja negra: **[cypress-e2e-suite](https://github.com/Danielito2252/cypress-e2e-suite)**.

---

## 🔄 Arquitectura del Pipeline e Integración del Ecosistema

El pipeline no trabaja de forma aislada; implementa una integración continua descargando dependencias y repositorios en caliente dentro de un entorno reproducido por contenedores.

[ 1. Checkout ] ➡️ [ 2. SCA (NPM) ] ➡️ [ 3. SAST (Trivy) ] ➡️ [ 4. Security Gate 🛑 ] ➡️ [ Descargar Cypress ] ➡️ [ 5. DAST (OWASP) ]




### Flujo Detallado de las Etapas:
1. **1. Checkout SCM:** Clonado automático del código fuente de la infraestructura utilizando la instrucción nativa `checkout scm`.
2. **2. Security Scan - SCA:** Análisis de composición de software para verificar firmas y dependencias mediante `npm audit`.
3. **3. Security Scan - SAST:** Escaneo estático profundo del sistema de archivos generando un reporte estructurado en formato JSON (`trivy-report.json`).
4. **4. Security Gate (El Guardián):** Bloque de control lógico programado en Groovy. Lee el JSON de la etapa anterior, contabiliza los riesgos y **aborta la ejecución con un estado de FAILURE** si se detecta al menos una vulnerabilidad de criticidad `HIGH` o `CRITICAL`.
5. **Descargar Cypress Suite:** Conexión modular que clona en caliente la suite de pruebas E2E desde el Repositorio 1 en el espacio de trabajo activo.
6. **5. Security Scan - DAST:** Análisis dinámico automatizado en tiempo de ejecución, simulando vectores de ataque contra el entorno objetivo y emitiendo un reporte interactivo en HTML (`zap-report.html`).

---

## 📊 Evidencias de Ejecución (Datos Reales)

A continuación se presentan las capturas de pantalla reales extraídas directamente de la consola de administración de Jenkins en el entorno local:

### 1. Flujo Exitoso Secuencial (Pipeline Graph View)
*Demostración del paso firme por cada una de las validaciones requeridas de ingeniería de software:*

![Jenkins Stage View](https://github.com/Danielito2252/Jenkins-Devsecops-Pipeline/blob/main/images/stage-view-success.png?raw=true)

### 2. Comportamiento del Security Gate (Aprobado)
*Logs de la consola de Groovy leyendo el archivo JSON y permitiendo el paso del pipeline al no encontrar amenazas críticas:*

![Security Gate Log Exitoso](https://github.com/Danielito2252/Jenkins-Devsecops-Pipeline/blob/main/images/security-gate-log.png?raw=true)

### 3. Reporte Dinámico Generado (OWASP ZAP DAST)
*Estructura del artefacto interactivo HTML generado de forma automatizada en el Workspace tras finalizar el escaneo de vulnerabilidades web:*

![OWASP ZAP Report HTML](https://github.com/Danielito2252/Jenkins-Devsecops-Pipeline/blob/main/images/zap-report.png?raw=true)

---

## ⚙️ Requisitos e Instalación

Para clonar y replicar este pipeline en un servidor Jenkins local, asegúrese de cumplir con los siguientes componentes:

### Requisitos del Sistema
- **Docker Desktop** (con soporte para contenedores Linux ejecutándose en Windows).
- **Jenkins LTS** montado sobre Docker compartiendo el socket nativo: `-v /var/run/docker.sock:/var/run/docker.sock`.

### Plugins de Jenkins Requeridos
- `Pipeline Utility Steps` (Indispensable para habilitar la función `readJSON` encargada de procesar el Security Gate).
- `Docker Pipeline`

### Instrucciones de Despliegue
1. Instale el plugin **Pipeline Utility Steps** desde la administración de Jenkins.
2. Cree una nueva tarea de tipo **Pipeline** llamada `jenkins-devsecops-pipeline`.
3. En la sección de configuración de la canalización, seleccione **Pipeline script from SCM**.
4. Configure el SCM como **Git**, pegue la URL de este repositorio y defina la rama como `*/main`.
5. Haga clic en **Build Now** (Construir ahora).