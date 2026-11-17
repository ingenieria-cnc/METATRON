# METATRON — Fork CNC

METATRON es un asistente local para tareas de reconocimiento, análisis y validación de seguridad.

Este repositorio corresponde a un fork del proyecto original **METATRON**, adaptado y extendido en el marco del Laboratorio del Centro Nacional de Ciberseguridad (CNC), con foco en su utilización como herramienta de apoyo para analistas de ciberseguridad.

El objetivo de este fork no es reemplazar el criterio técnico del analista ni automatizar completamente un proceso de pentesting. METATRON busca organizar evidencia, relacionar resultados provenientes de distintas herramientas y orientar la investigación técnica posterior.

> **Los hallazgos generados por METATRON requieren validación del analista antes de considerarse vulnerabilidades confirmadas.**

---

## 1. Objetivo

Las herramientas tradicionales de reconocimiento pueden generar una gran cantidad de información técnica que luego debe ser interpretada y correlacionada manualmente.

METATRON busca asistir en esa tarea mediante la combinación de:

- herramientas tradicionales de reconocimiento;
- un modelo de lenguaje ejecutado localmente;
- persistencia estructurada de resultados;
- una metodología orientada a distinguir evidencia, validaciones pendientes y vulnerabilidades candidatas.

El analista conserva siempre la responsabilidad sobre la interpretación, contextualización y validación final de los resultados.

---

## 2. Alcance

METATRON está orientado a:

- reconocimiento técnico de activos;
- recolección y organización de evidencia;
- identificación de observaciones relevantes;
- propuesta de validaciones adicionales;
- identificación de vulnerabilidades candidatas;
- generación de mitigaciones;
- elaboración de un resumen de riesgo;
- almacenamiento histórico de resultados.

METATRON **no realiza explotación automática de vulnerabilidades** y no debe interpretarse como un sistema autónomo de pentesting.

---

## 3. Modelo de análisis

El procesamiento de resultados se estructura conceptualmente en tres niveles:

### OBSERVATION

Hecho directamente observado mediante una herramienta.

Ejemplos:

- un puerto TCP se encuentra abierto;
- un determinado servicio fue identificado;
- un certificado es autofirmado;
- un servidor responde con una determinada cabecera.

Una observación no implica por sí misma una vulnerabilidad.

### VALIDATION

Condición que requiere una comprobación adicional.

Ejemplos:

- verificar si un servicio DNS permite recursión abierta;
- comprobar si un servicio admite autenticación anónima;
- validar una configuración detectada por una herramienta.

### VULN

Condición que cuenta con evidencia suficiente para ser considerada una vulnerabilidad candidata.

La clasificación realizada por METATRON no reemplaza la validación del analista.

Antes de aceptar un hallazgo se recomienda comprobar:

- qué evidencia concreta lo sustenta;
- si la condición fue realmente verificada;
- si se trata de una vulnerabilidad o de una recomendación de hardening;
- si la severidad asignada es adecuada;
- si existe alguna validación pendiente;
- si puede tratarse de un falso positivo.

---

## 4. Arquitectura

La arquitectura principal se divide en tres componentes:

### `metatron.py`

Orquestador principal.

Responsable de:

- interacción CLI;
- selección de herramientas;
- creación de sesiones;
- coordinación del análisis;
- persistencia de resultados.

### `llm.py`

Motor de análisis.

Responsable de:

- interacción con el modelo de lenguaje;
- procesamiento de evidencia;
- generación de observaciones;
- identificación de validaciones;
- clasificación de vulnerabilidades candidatas;
- control de rondas de análisis;
- prevención de llamadas repetidas a herramientas;
- generación de mitigaciones y resumen final.

### `db.py`

Capa de persistencia.

Responsable de:

- conexión con MariaDB;
- almacenamiento del historial;
- almacenamiento de observaciones;
- almacenamiento de vulnerabilidades;
- almacenamiento de mitigaciones;
- almacenamiento de resúmenes.

---

## 5. Herramientas integradas

Según la selección realizada durante el análisis, METATRON puede utilizar herramientas tradicionales como:

- Nmap
- Nikto
- WhatWeb
- curl
- dig
- WHOIS

Estas herramientas generan la evidencia técnica que posteriormente es analizada y correlacionada.

---

## 6. Inteligencia artificial local

El fork está diseñado para trabajar con un modelo de lenguaje ejecutado localmente mediante Ollama.

Este enfoque permite que:

- la evidencia permanezca dentro de la infraestructura del organismo;
- no sea necesario enviar resultados de análisis a servicios externos de IA;
- el organismo mantenga control sobre el modelo utilizado;
- la infraestructura de ejecución pueda ser auditada y administrada localmente.

El modelo de lenguaje no debe considerarse una fuente autoritativa.

Puede producir:

- falsos positivos;
- interpretaciones incorrectas;
- inferencias incompletas;
- severidades discutibles;
- clasificaciones que requieran revisión.

El criterio final corresponde al analista.

---

## 7. Controles incorporados en este fork

Este fork incorpora modificaciones destinadas a mejorar el comportamiento del modelo y la calidad del análisis.

Entre ellas:

- separación entre observaciones y vulnerabilidades;
- soporte para validaciones adicionales;
- instrucciones orientadas a evidencia;
- reducción de inferencias no sustentadas;
- filtros para determinados falsos positivos recurrentes;
- control de llamadas repetidas a herramientas;
- límite de rondas de análisis;
- generación de salida estructurada;
- generación de resultados en español;
- almacenamiento de observaciones en MariaDB.

El objetivo de estos controles es reducir ruido y mejorar la utilidad del resultado, no eliminar completamente los falsos positivos.

---

## 8. Base de datos

METATRON utiliza MariaDB para almacenar el historial de análisis.

El esquema utilizado por esta versión se incluye en:

```text
metatron_schema.sql
```

La estructura incluye las tablas:

```text
history
observations
vulnerabilities
fixes
exploits_attempted
summary
```

El archivo de esquema permite recrear la estructura de la base sin incluir datos históricos de análisis.

Ejemplo:

```bash
sudo mysql -e "CREATE DATABASE metatron;"
sudo mysql metatron < metatron_schema.sql
```

---

## 9. Instalación

Se recomienda desplegar METATRON en una máquina virtual dedicada.

Cada organismo debería realizar su propia instalación y controlar:

- sistema operativo;
- permisos;
- credenciales;
- acceso a Internet;
- VLAN o segmentos de red accesibles;
- reglas de firewall;
- recursos asignados;
- versión del modelo de lenguaje.

### Clonar el repositorio

```bash
git clone https://github.com/ingenieria-cnc/METATRON
cd METATRON
```

### Crear entorno virtual

```bash
python3 -m venv venv
source venv/bin/activate
```

### Instalar dependencias

```bash
pip install -r requirements.txt
```

### Configurar MariaDB

Crear la base:

```bash
sudo mysql -e "CREATE DATABASE metatron;"
```

Importar el esquema:

```bash
sudo mysql metatron < metatron_schema.sql
```

### Configurar Ollama

Instalar Ollama y descargar o crear el modelo definido para la implementación.

La configuración del modelo puede encontrarse en:

```text
Modelfile
```

### Ejecutar METATRON

```bash
python3 metatron.py
```

---

## 10. Modelo de distribución

Este fork se distribuye siguiendo un enfoque basado en:

### Auditable

El código puede ser revisado y comparado con el proyecto original.

Los organismos receptores pueden identificar exactamente qué modificaciones fueron incorporadas.

### Controlado

Cada organismo despliega METATRON en su propia infraestructura.

El organismo mantiene el control sobre:

- VM;
- red;
- credenciales;
- permisos;
- segmentos autorizados;
- modelo de lenguaje;
- acceso externo.

### Trazable

Se recomienda utilizar releases versionadas.

Cada release debería incluir:

- código fuente;
- número de versión;
- notas de cambios;
- esquema de base de datos;
- documentación;
- hashes SHA-256 de los archivos distribuidos.

Esto permite identificar exactamente qué versión fue probada y desplegada.

---

## 11. Relación con el proyecto original

Este repositorio es un fork de un proyecto open source preexistente.

Se recomienda mantener configurado el repositorio original como `upstream` para facilitar el seguimiento de cambios futuros.

Ejemplo:

```bash
git remote -v
```

Configuración esperada:

```text
origin    https://github.com/ingenieria-cnc/METATRON
upstream  https://github.com/sooryathejas/METATRON
```

Esto permite:

- identificar diferencias;
- incorporar selectivamente mejoras del proyecto original;
- evitar actualizaciones automáticas no evaluadas;
- mantener trazabilidad sobre las modificaciones institucionales.

---

## 12. Recomendaciones de despliegue

Para una primera implementación se recomienda:

1. desplegar la herramienta en laboratorio;
2. utilizar activos deliberadamente vulnerables o sistemas de prueba;
3. validar el comportamiento del modelo;
4. evaluar falsos positivos;
5. revisar las herramientas ejecutadas;
6. verificar la persistencia de resultados;
7. limitar inicialmente los segmentos de red accesibles;
8. ampliar progresivamente el alcance.

No se recomienda otorgar acceso irrestricto a múltiples redes desde el primer despliegue.

---

## 13. Uso autorizado

METATRON debe utilizarse únicamente sobre:

- activos propios;
- laboratorios;
- ambientes de prueba;
- redes expresamente autorizadas;
- infraestructura para la cual exista autorización formal de evaluación.

El uso de esta herramienta contra sistemas sin autorización puede violar normas, políticas internas o legislación aplicable.

---

## 14. Limitaciones conocidas

METATRON puede producir resultados incorrectos o incompletos.

Entre las limitaciones conocidas:

- falsos positivos;
- identificación incorrecta de servicios;
- inferencias basadas en información insuficiente;
- clasificación discutible de severidad;
- dependencia de la calidad de salida de herramientas externas;
- ausencia de contexto de negocio;
- dependencia del modelo de lenguaje utilizado.

Un hallazgo generado por METATRON no debe incorporarse automáticamente a un informe técnico sin revisión.

---

## 15. Rol del analista

METATRON debe considerarse un asistente.

La responsabilidad del analista incluye:

- revisar la evidencia;
- confirmar las condiciones detectadas;
- descartar falsos positivos;
- contextualizar el impacto;
- determinar la severidad final;
- realizar validaciones técnicas adicionales cuando corresponda;
- decidir qué hallazgos deben formar parte de un informe.

> **La IA propone y organiza; el analista valida, contextualiza y decide.**

---

## 16. Estado del proyecto

Estado actual:

```text
Beta funcional
```

La versión actual fue diseñada para evaluación controlada y uso asistido por analistas.

Las futuras mejoras deberían surgir principalmente de:

- experiencia real de uso;
- feedback de analistas;
- identificación de falsos positivos recurrentes;
- nuevas necesidades de integración;
- mejoras provenientes del proyecto upstream.

---

## 17. Seguridad y reporte de problemas

Los problemas relacionados con:

- funcionamiento;
- seguridad;
- falsos positivos sistemáticos;
- comportamiento del modelo;
- compatibilidad;
- dependencias;

deberían reportarse mediante los mecanismos definidos en este repositorio.

No incluir en los reportes públicos:

- credenciales;
- direcciones internas sensibles;
- evidencia de incidentes reales;
- información clasificada;
- datos personales;
- configuraciones de infraestructura que no deban divulgarse.

---

## 18. Licencia

Este fork mantiene la licencia correspondiente al proyecto original.

Se deben conservar los avisos de copyright y licencia exigidos por dicha licencia.

Consultar:

```text
LICENSE
```

para conocer los términos aplicables.

---

## 19. Principio de diseño

METATRON no busca reemplazar al analista ni automatizar completamente su criterio.

Su propósito es:

> **combinar herramientas tradicionales, inteligencia artificial local y criterio humano para mejorar la capacidad de interpretar evidencia y orientar una investigación técnica.**