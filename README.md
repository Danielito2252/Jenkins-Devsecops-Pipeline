# Jenkins DevSecOps Pipeline

Este repositorio contiene la configuración de un pipeline de Integración y Despliegue Continuo (CI/CD) con un enfoque estricto en seguridad (DevSecOps)[cite: 2, 27].

## 🚀 Características del Pipeline
El flujo está diseñado bajo una arquitectura declarativa en Jenkins y consta de las siguientes etapas:
1. **Checkout:** Clonado automático del código fuente[cite: 50].
2. **SCA (Software Composition Analysis):** Escaneo de vulnerabilidades en dependencias mediante `npm audit`[cite: 51].
3. **SAST (Static Application Security Testing):** Análisis estático del sistema de archivos usando un contenedor de `Trivy`[cite: 51].
4.**Security Gate:** Filtro inteligente que aborta el proceso si se detectan riesgos altos o críticos[cite: 52].
5. **Integración con Cypress:** Descarga de la suite de pruebas automatizadas desde el repositorio externo[cite: 36].
6. **DAST (Dynamic Application Security Testing):** Análisis dinámico en tiempo de ejecución con `OWASP ZAP`[cite: 51].

## 🛠️ Requisitos del Entorno
[cite_start]El entorno es completamente reproducible mediante Docker[cite: 54]:
- [cite_start]Jenkins LTS con acceso al socket de Docker (`docker.sock`).
- Plugin `Pipeline Utility Steps` instalado en Jenkins.