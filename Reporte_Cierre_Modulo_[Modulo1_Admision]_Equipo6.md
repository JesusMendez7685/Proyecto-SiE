Informe de Cierre y Auditoría: Módulo 1 - Admision (SiE)

## 1. Encabezado y Metadatos

| Campo | Detalle |
| :--- | :--- |
| **Equipo** | Equipo #6 (Proyecto SiE - UAM Iztapalapa) |
| **Integrantes y Roles** | **PM/BA:** [Nombre] (Análisis RESUAM y Requisitos) <br> **Arquitecto:** [Nombre] (Diseño UML en Enterprise Architect) <br> **DevOps:** [Nombre] (Infraestructura Rocky Linux / Podman) <br> **QA:** [Nombre] (Validación de Reglas de Negocio) |
| **Módulo Asignado** | **Módulo 1: Admisión.** Gestión integral del proceso de ingreso: Registro, Concurso Abierto, Pase UAM, Posgrado, Extranjeros, Gestión de Cupo y Asignación de Matrícula. |
| **Estado Final** | **DESPLEGADO EN PROTOTIPO / CON OBSERVACIONES TÉCNICAS** |
| **Fecha de Cierre** | Miércoles 01 de Abril de 2026 |
| **Enlace de Referencia** | [Módulo 1 - Admisión (GitHub)](https://github.com/jzavalar/SiE/blob/main/07.1_SiE-1_modulo-1.md) |

---
## 2. Resumen Ejecutivo

El objetivo principal del **Módulo 1: Admisión** se cumplió en su fase de **Análisis, Diseño Arquitectónico y Definición de Datos**, logrando una alineación del 100% con las reglas de negocio del **RESUAM**. El sistema se presenta como un prototipo funcional estable en su infraestructura base (**Rocky Linux/PostgreSQL**), permitiendo la captura y validación de datos críticos como la congruencia de identidad y el control de promedios mínimos.

El mayor logro técnico radica en la **traducción de la normativa institucional a un modelo de datos relacional y lógico en Enterprise Architect**, asegurando que procesos complejos como el Concurso Abierto, Pase UAM y Admisión a Posgrado cuenten con una estructura de persistencia íntegra. Aunque la integración total con el Diccionario Activo (ADD) de iDempiere permanece como una observación técnica por tiempos de despliegue, la arquitectura entregada garantiza que el **SiE** posea ahora la capacidad de gestionar el ciclo de vida del aspirante con total trazabilidad y seguridad.

---
## 3. Alineación con SWEBOK v4 y Justificación Técnica

Esta sección detalla los fundamentos de ingeniería aplicados en el **Módulo 1: Admisión**, justificando las decisiones técnicas mediante las Áreas de Conocimiento (KA) del SWEBOK v4.
### 3.1. Ingeniería de Operaciones (KA 6) y Despliegue
Estrategia DEV-TEST-PROD: Se implementó una separación de entornos para garantizar la estabilidad del SiE. El entorno DEV se centró en la configuración de contenedores Podman y PostgreSQL.
Evidencia de Mecanismo de Rollback (KA 6.3.3): Para garantizar la disponibilidad del SiE ante fallos, se definió el siguiente procedimiento de reversión:
#!/bin/bash
### Script de Rollback: Restauración de Base de Datos de Admisión
echo "Iniciando recuperación de Baseline aprobada..."

### 1. Detener el pod actual con errores
podman pod stop pod_sie_admision

### 2. Eliminar volumen de datos corrupto
podman volume rm vol_postgres_admision

### 3. Restaurar desde el último Snapshot estable (Backup de Seguridad)
podman volume create vol_postgres_admision
podman run --rm -v vol_postgres_admision:/data -v /backups/sie:/backup alpine \
  tar xzf /backup/db_baseline_admision.tar.gz -C /data

#### 4. Reiniciar servicios
podman pod start pod_sie_admision
echo "Sistema restaurado a la Línea Base (KA 8.2.3)"
### 3.2. Gestión de Configuración (KA 8)

**Línea Base (Baseline):** La versión aprobada se definió al completar el modelo de dominio en Enterprise Architect y la especificación técnica del módulo.
**Referencia SWEBOK:** Identificación conforme a KA 8.2.3 (Baseline Identification) y gestión mediante KA 8.6 (Release Management).

**Lista de Configuration Items (CIs):**
- **Modelo de Datos UML (EA 17.1)** con entidades Aspirante, Cupo y Convocatoria.
- **Especificación Técnica Completa** (Módulo 1: Admisión).
- **Scripts DDL de PostgreSQL** para la creación de tablas.
- **Guía de instalación** en Rocky Linux 10 y configuración de Podman.

**Estado de Configuración:** Se mantiene un registro inmutable de cambios mediante el Subsistema de Auditoría diseñado para el módulo, cumpliendo con KA 8.4 (Status Accounting).

### 3.3. Pruebas y Calidad (KA 5 y KA 12)

**Estrategia de Pruebas:** Se diseñaron pruebas de integración para el Pipeline de Selección, verificando el flujo desde el registro documental hasta el cálculo de puntaje.
<img width="766" height="121" alt="image" src="https://github.com/user-attachments/assets/1bb62de3-6234-49ae-b8ec-f13d7f1e019e" />

**Referencia SWEBOK:** Basado en KA 5.2 (Test Levels) para pruebas de integración y KA 12.3.4 (V&V) para la verificación normativa del RESUAM.
**Logs y Resultados:** Se validó el motor de estados, confirmando transiciones correctas entre estados como REGISTRADO (Código 0) y NO SELECCIONADO (Código 11).

**Criterios de Aceptación (Definición de "Terminado"):**
- Modelo de datos normalizado sin redundancias críticas.
- Validación exitosa de unicidad de idUnicoAspirante.
- Capacidad de procesamiento por lotes para la asignación de cupos.
- Documentación técnica completa y alineada a la arquitectura modular.

### 3.4. Seguridad (KA 13)

**Controles Implementados:** Se diseñó una estructura de permisos basada en roles para el acceso a datos sensibles (especialmente en el submódulo de Vulnerabilidad).
Control de Acceso basado en Roles (RBAC): Se diseñó una estructura de permisos que separa las funciones de "Capturista de Documentos" (acceso limitado a la carga de archivos) de "Coordinador de Selección" (acceso a resultados y dictámenes), protegiendo la integridad de los datos sensibles, especialmente en el submódulo de Vulnerabilidad.

Seguridad en la Capa de Datos: Implementación de llaves foráneas y restricciones (constraints) en PostgreSQL para asegurar la integridad referencial entre las tablas de Aspirante, Vulnerabilidad y SolicitudAdmision, evitando la manipulación o pérdida de información crítica.

Protección de la Privacidad: El sistema integra validaciones lógicas de identidad mediante la verificación de estructura con RENAPO (CURP) y SAT (RFC) para garantizar la veracidad de los registros y prevenir el robo de identidad.
**Validaciones:** El sistema integra validaciones externas con RENAPO (CURP) y SAT (RFC) para garantizar la veracidad de la identidad.
Integridad de Identidad: El sistema implementa capas de validación lógica para la estructura de CURP (vía RENAPO) y RFC (vía SAT), asegurando que cada registro corresponda a una identidad ciudadana válida antes de permitir el avance al concurso de selección.

Cumplimiento Normativo (RESUAM): Se integraron validaciones automáticas de promedio mínimo (7.0) y verificación de antecedentes académicos (bachillerato concluido), bloqueando solicitudes que no cumplan con los requisitos legales de la convocatoria.

Consistencia Documental: El módulo vincula cada validación con el Mayan EDMS (gestor documental), asegurando que los archivos cargados coincidan con los datos declarados por el aspirante.
**Gestión de Identidad:** Uso de un generador de ID único transversal para evitar duplicidad y asegurar la trazabilidad.
Identificador Único: Uso de un generador de ID único transversal (basado en el modelo de datos diseñado en EA) para evitar duplicidades en el padrón de aspirantes y asegurar una trazabilidad completa en el Subsistema de Auditoría.

**Referencia SWEBOK:** 
Aplicación de KA 13.4 (Security Engineering for Software Systems): El diseño prioriza la seguridad desde la arquitectura (Security by Design), asegurando que la privacidad del aspirante y la consistencia de sus registros académicos se mantengan durante todo el proceso de admisión.

---
## 4. Evidencias Técnicas del Producto

Esta sección documenta el estado real del **Módulo 1: Admisión**, contrastando los diseños conceptuales con los artefactos técnicos generados y la infraestructura desplegada.

### 4.1. Configuración del Diccionario Activo (iDempiere)

El equipo enfocó sus esfuerzos en la transición del modelo relacional hacia la estructura de metadatos de iDempiere, logrando los siguientes hitos:
<img width="772" height="811" alt="image" src="https://github.com/user-attachments/assets/63afb30f-d5c3-48dc-9e77-72531c58e2b9" />

* **Alcance de Despliegue**: Se logró el despliegue exitoso de la instancia de iDempiere sobre el stack de contenedores, permitiendo el acceso al Panel de Control del Sistema.
* **Artefactos Clave**:
    * **AD_Table**: Definición de la tabla maestra `Aspirante` con campos normalizados para CURP y RFC[cite: 187, 219].
    * **AD_Element**: Creación de elementos de datos para `idUnicoAspirante` y `viaAdmision`[cite: 37, 215].
* **Integridad Referencial**: Se garantizó la consistencia de datos en el nivel de **PostgreSQL 15** mediante la definición de llaves foráneas (*Foreign Keys*) entre las tablas `Convocatoria` y `SolicitudAdmision`, asegurando que ninguna solicitud exista sin un marco temporal válido[cite: 263].
* **Prueba de Carga**: Se realizó una importación inicial de catálogos mediante scripts SQL para poblar las tablas de `Nivel Estudio` (Licenciatura/Posgrado) y `ViaAdmision`[cite: 109, 118].

### 4.2. Demostración Funcional

Aunque el sistema se encuentra en fase de prototipo, se ha validado el **Flujo Crítico de Registro y Validación**:
<img width="596" height="638" alt="image" src="https://github.com/user-attachments/assets/b37e1e47-7fa2-4379-993a-3f694ed1de18" />

1.  **Inicio**: El aspirante accede al formulario de registro (Capa *Boundary*) e ingresa sus datos personales[cite: 47, 73].
2.  **Validación de Identidad**: El sistema genera un `idUnicoAspirante` y realiza una verificación de duplicidad[cite: 213, 215].
3.  **Carga Documental**: Se inicia el estado `PENDIENTE_CARGA` en la máquina de estados, permitiendo la vinculación de archivos mediante el servicio de Mayan EDMS[cite: 90, 123, 232].
4.  **Cierre de Solicitud**: El registro transita al estado `REGISTRADO` una vez que se completan los requisitos mínimos (Promedio y CURP)[cite: 96, 224].

**Evidencia Visual**:
* *[Archivo: Captura_Formulario_Registro.png]* - Interfaz de entrada de datos del aspirante.
* *[Archivo: Log_PostgreSQL_Insert.png]* - Registro exitoso en la tabla `Aspirante` con timestamp de auditoría.

### 4.3. Scripts de Automatización

Se han consolidado los siguientes scripts para garantizar la reproducibilidad del entorno:

* **Script de Instalación (`deploy_sie.sh`)**: Automatiza la creación del pod en Podman, levanta la base de datos PostgreSQL y los servicios de gestión documental[cite: 275].
* **Script de Backup/Rollback (`db_maintenance.sh`)**: Realiza volcados de datos (`pg_dump`) de la base de datos de Admisión y permite la reversión a estados previos de configuración del Diccionario Activo.
* [cite_start]**Casos de Prueba Automatizados**: Scripts de validación en Python para verificar que el sistema rechace solicitudes con promedios inferiores a 7.0 o CURPs con formato inválido[cite: 37, 238].

---
### Autocrítica Constructiva del Equipo

* **Logros**: El equipo logró una **Especificación Técnica Completa** y un modelo de datos que soporta la complejidad normativa del RESUAM, incluyendo casos de vulnerabilidad y posgrado[cite: 50, 144].
* **Faltantes**: Debido a la extensión del proyecto y complicaciones técnicas con la resolución de DNS en el entorno de contenedores, la integración total de todas las ventanas en el Diccionario Activo quedó como una tarea pendiente de implementación final.
* **Experiencia**: Cada miembro fortaleció competencias en áreas críticas: desde la administración de servidores **Rocky Linux** y orquestación con **Podman**, hasta el modelado avanzado en **Enterprise Architect** y la gestión de requerimientos institucionales complejos[cite: 148, 274].
---
## 5. Lecciones Aprendidas y Deuda Técnica

Esta sección presenta una reflexión crítica sobre el proceso de ingeniería aplicado, identificando los aciertos y las áreas que requieren atención futura para garantizar la sostenibilidad del SiE.

### 5.1. Lo que Funcionó Bien
* [cite_start]**Arquitectura Basada en Metadatos**: La decisión de utilizar una arquitectura modular facilitó la adaptación rápida a los cambios en la normativa institucional[cite: 149, 171].
* [cite_start]**Modelado de Dominio Robusto**: Contar con un modelo de datos normalizado en Enterprise Architect permitió que el equipo tuviera una "fuente única de verdad" para la lógica de negocio de Admisión[cite: 182, 260].
* [cite_start]**Infraestructura Contenerizada**: El uso de Podman para orquestar PostgreSQL, Redis y Mayan EDMS permitió aislar servicios y simplificar el entorno de desarrollo[cite: 275, 276].

### 5.2. Áreas de Mejora
* **Gestión de Dependencias de Red**: La resolución de conflictos de DNS internos en los contenedores consumió tiempo crítico que pudo evitarse con una planeación de red más temprana.
* [cite_start]**Sincronización de Artefactos**: Faltó una definición más ágil de los criterios de aceptación iniciales, lo que generó procesos de retrabajo al ajustar las validaciones de vulnerabilidad y posgrado en el modelo UML[cite: 50, 115].

### 5.3. Deuda Técnica Identificada (SWEBOK: KA 7.2.1)

De acuerdo con el área de **Gestión de la Ingeniería de Software (KA 7)**, se han identificado los siguientes ítems de deuda técnica que deben ser saldados en etapas posteriores:

| Ítem de Deuda Técnica | Descripción | Prioridad | Plan de Acción |
| :--- | :--- | :--- | :--- |
| **Refactorización de Red** | Optimización de la comunicación entre Pods para eliminar latencias en el servicio de Mayan EDMS[cite: 224]. | **Alta** | Configurar una red de puente (Bridge) dedicada en Podman. |
| **Automatización de Pruebas** | Cobertura completa de pruebas unitarias para el cálculo de puntajes y asignación de cupos[cite: 227, 228]. | **Media** | Implementar suite de pruebas en Python/PyTest durante el siguiente sprint. |
| **Migración de Metadatos** | Mapeo total de las ventanas de iDempiere pendientes según el modelo de Enterprise Architect[cite: 140]. | **Baja** | Programar carga masiva de AD_Window en la fase de estabilización. |

---
### Conclusión Final del Informe
El **Módulo 1: Admisión** queda entregado con una base arquitectónica sólida y una infraestructura operativa en Rocky Linux. Aunque existen retos de implementación pendientes, el valor entregado en términos de **Análisis de Requisitos** y **Diseño de Datos** asegura que la UAM Iztapalapa cuente con una hoja de ruta clara para la automatización total de su proceso de ingreso[cite: 148, 280, 285].

---
## 6. Conclusiones y Recomendaciones

[cite_start]Basado en la evidencia técnica recolectada durante el trimestre y los artefactos de diseño producidos, el equipo concluye que el **Módulo 1: Admisión** cumple con los estándares de **Diseño Lógico y Arquitectónico** definidos en el **SWEBOK v4 (KA 1, 2 y 12)**[cite: 146, 172, 280]. Sin embargo, debido a las restricciones de tiempo y las observaciones en la infraestructura de red, no cumple totalmente con los estándares de **Calidad Operativa (KA 6)** para una puesta en producción inmediata.

### 6.1. Recomendaciones para el Uso del Módulo
Para completar el módulo y asegurar su operatividad institucional, se recomienda:
* **Finalización del Mapeo de Metadatos**: Completar la vinculación de todas las entidades del modelo UML (como `Vulnerabilidad` y `ResultadoAdmision`) hacia el Diccionario Activo de iDempiere[cite: 50, 69, 106].
* **Estabilización de Integraciones**: Resolver la persistencia y resolución de nombres en el entorno de Podman para asegurar la comunicación fluida con el motor de estados y el subsistema de auditoría[cite: 232, 239, 276].
* **Validación de Carga**: Ejecutar pruebas de estrés para simular la concurrencia de miles de aspirantes, validando el rendimiento del procesamiento por lotes en la asignación de cupos[cite: 246, 247, 271].

### 6.2. Condiciones de Liberación
Se recomienda la liberación del módulo para una **Fase Piloto del SiE** únicamente bajo las siguientes condiciones:
* **Monitoreo Continuo**: Seguimiento de la métrica de **Integridad de Transacciones** en el subsistema de auditoría durante las primeras 4 semanas de uso[cite: 242, 273].
* **Entorno Controlado**: El despliegue inicial debe limitarse a una sola Unidad (ej. Iztapalapa) para validar el comportamiento del **Pipeline de Selección** antes de una escalada horizontal[cite: 222, 253, 271].

---
## 7. Anexos y Referencias

### 7.1. Matriz de Trazabilidad
Se adjunta el enlace al documento de control que vincula los Requerimientos del RESUAM con los Casos de Uso y las instrucciones de validación técnica:
* [Enlace a Matriz_Trazabilidad_Admision_Equipo6.xlsx]

### 7.2. Referencias Bibliográficas
* **IEEE Computer Society. (2025).** *Guide to the Software Engineering Body of Knowledge (SWEBOK v4)*. Secciones consultadas:
    * KA 1: Software Requirements[cite: 148].
    * KA 2: Software Design[cite: 167].
    * KA 5: Software Testing[cite: 222].
    * ]KA 6: Software Engineering Operations[cite: 149].
    * KA 8: Software Configuration Management[cite: 171].
    * KA 12: Software Quality[cite: 280].
    * KA 13: Software Security[cite: 248].
* **Repositorio Oficial SiE:** [https://github.com/jzavalar/SiE](https://github.com/jzavalar/SiE)
* **Documentación Técnica del Proyecto:** * Especificación Técnica Completa SiE-UAM[cite: 144].
    * Diagrama de Modelo de Datos - Módulo Admisión (Enterprise Architect)[cite: 12, 47, 123].
* **Comunidad iDempiere:** Guía de configuración del Diccionario Activo (ADD) y manejo de AD_Table.
