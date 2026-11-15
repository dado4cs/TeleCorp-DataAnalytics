# Caso de Estudio: Red TeleCorp

## 1. Contexto y Problemática

**Red TeleCorp** es una empresa de telecomunicaciones que brinda servicios de telefonía móvil, Internet y conectividad a millones de usuarios distribuidos en diferentes ciudades y zonas geográficas. Debido a la gran cantidad de clientes, dispositivos, estaciones base y elementos que forman parte de su infraestructura, mantener la continuidad y calidad del servicio representa uno de los principales desafíos operativos de la organización.

La experiencia de un usuario no depende únicamente de que su dispositivo tenga señal. Una interrupción en una antena, una degradación del servicio, una saturación de la infraestructura o una falla localizada puede afectar llamadas, mensajes, navegación por Internet y otros servicios. En determinadas situaciones, el problema puede afectar solamente a algunos usuarios, mientras que en otras puede impactar a cientos o miles de personas ubicadas dentro de una misma zona.

---

### 1.1. Fuentes de Información y Parámetros de Simulación

Actualmente, la empresa recibe información proveniente de diferentes fuentes:
1. **Métricas generadas por las antenas.**
2. **Reportes realizados por los usuarios.**
3. **Registros generados por los sistemas internos.**

El volumen de información puede crecer considerablemente durante determinados periodos, por lo que resulta necesario contar con mecanismos capaces de procesar estos acontecimientos de manera oportuna.

Como parte del desafío, cada antena de la red genera periódicamente información relacionada con la cantidad de tráfico que está procesando. Para efectos de la hackathon, se establecen los siguientes umbrales operativos:

| Estado de la Antena | Rango de Tráfico (`GB / minuto`) |
| :--- | :--- |
| **Inhabilitada** | `0 GB/min` |
| **Caída** | `> 0 GB/min` y `< 60 GB/min` |
| **Normal** | `60 GB/min` a `129 GB/min` (inclusive) |
| **Saturada** | `> 129 GB/min` |

---

### 1.2. Gestión de Incidencias y Reportes de Usuarios

La información proveniente de las antenas no es la única fuente que debe ser considerada. **Red TeleCorp** desea mejorar el proceso mediante el cual los propios usuarios pueden reportar problemas relacionados con su servicio:

* **Portal de gestión de incidencias:** Debe permitir que una persona registre un inconveniente proporcionando:
  * Identificación del usuario.
  * Número asociado al servicio.
  * Ubicación aproximada.
  * Fecha y hora del inconveniente.
  * Tipo de problema.
  * Descripción de lo ocurrido.
* **Ciclo de vida y trazabilidad:** Los reportes deberán poder ser consultados y administrados posteriormente. Cada incidencia deberá mantener un **estado** y conservar un **historial** de su evolución:
  `Registrada` → `En revisión` → `Siendo atendida` → `Solucionada` → `Cerrada`

---

### 1.3. Dolores Clave a Resolver

1. **Detección de incidentes colectivos:**
   * Incidentes que inicialmente parecen aislados pueden corresponder a un problema generalizado (por ejemplo, múltiples reportes de falta de Internet en un mismo sector a lo largo de las horas).
   * **Regla de negocio:** Si se detecta que **10 o más usuarios diferentes** reportan un inconveniente relacionado con el servicio dentro de una **misma zona** en un **intervalo menor a 24 horas**, la organización deberá considerar que existe un **posible incidente colectivo** y realizar una verificación de la infraestructura correspondiente.
2. **Comunicación proactiva con los usuarios afectados:**
   * Cuando se determine una afectación zonal, los usuarios que reportaron el inconveniente deberán recibir comunicaciones por **correo electrónico y/o teléfono celular (SMS)** informando si el inconveniente está siendo evaluado, si se identificó una incidencia en la zona o si el servicio fue restablecido.
3. **Priorización inteligente:**
   * No todos los incidentes tienen la misma importancia según la cantidad de afectados (5 personas vs. miles de usuarios) y el tipo de zona afectada (**residencial** vs. **zona industrial, institución educativa, centro médico o servicios críticos**).
4. **Identificación de patrones y zonas críticas:**
   * Conocer qué zonas presentan mayor cantidad de incidentes, qué antenas fallan con mayor frecuencia y qué tipos de inconvenientes se reportan con mayor regularidad para apoyar la toma de decisiones.
5. **Detección de problemas estructurales (incidentes recurrentes):**
   * Una antena puede presentar varios problemas en días diferentes y *aparentemente recuperarse* tras cada evento. La acumulación de estos incidentes indica un posible **problema estructural** que requiere atención especializada.
6. **Reducción del tiempo de detección y correlación:**
   * Evitar que los problemas sean detectados primero por los usuarios antes que por la supervisión interna, estableciendo una correlación entre los reportes de usuarios y el comportamiento de la infraestructura.
7. **Resiliencia ante eventos masivos simultáneos:**
   * Soportar escenarios como cortes de energía, eventos climáticos o fallas masivas que generen miles de reportes en pocos minutos, sin pérdida de registros ni degradación considerable del servicio.
8. **Análisis histórico:**
   * Analizar la evolución de los incidentes, cuánto tiempo permanecieron activos, zonas afectadas, usuarios involucrados y frecuencia de fallas por elemento de infraestructura.
9. **Eficiencia de costos y arquitectura Serverless:**
   * Menor costo operativo posible, escalabilidad elástica según el volumen de usuarios/eventos y **sin depender de infraestructura permanentemente encendida**.

---

## 2. Requerimientos Funcionales — Módulo GCP

> **Responsabilidad del Módulo GCP:** Data Engineering, Analytics e Inteligencia Artificial.

### A. Ingesta y Almacenamiento de Datos

| ID | Requerimiento | Descripción |
| :--- | :--- | :--- |
| **RF-GCP-01** | **Recepción de métricas de infraestructura** | El sistema deberá recibir las métricas históricas de tráfico y estados operativos de las antenas generadas por el módulo **AWS**. |
| **RF-GCP-02** | **Recepción de información de incidencias** | El sistema deberá recibir información de las incidencias individuales y colectivas gestionadas por el módulo **Azure**, incluyendo datos agregados sobre usuarios afectados, ubicación, tipo de problema, prioridad y estado. |
| **RF-GCP-03** | **Validación de datos recibidos** | El sistema deberá validar la estructura, integridad y consistencia de los datos recibidos, verificando los identificadores de antenas, zonas, incidencias y marcas temporales. |
| **RF-GCP-04** | **Almacenamiento histórico** | El sistema deberá almacenar de manera persistente los registros históricos de métricas de infraestructura e incidencias para su posterior procesamiento y análisis. |
| **RF-GCP-05** | **Integración de fuentes de información** | El sistema deberá integrar los datos provenientes de **AWS** y **Azure**, permitiendo relacionar las métricas de las antenas con las incidencias reportadas en sus respectivas zonas y periodos de tiempo. |

---

### B. Procesamiento y Analítica de Datos

| ID | Requerimiento | Descripción |
| :--- | :--- | :--- |
| **RF-GCP-06** | **Procesamiento de datos históricos** | El sistema deberá procesar, transformar y organizar los datos recibidos para facilitar su consulta y análisis. |
| **RF-GCP-07** | **Análisis de zonas críticas** | El sistema deberá identificar las zonas geográficas con mayor concentración de incidencias y usuarios afectados. |
| **RF-GCP-08** | **Identificación de antenas problemáticas** | El sistema deberá identificar las antenas que presentan mayor frecuencia de fallas, degradaciones del servicio y saturaciones. |
| **RF-GCP-09** | **Identificación de problemas recurrentes** | El sistema deberá identificar los tipos de problemas más frecuentes y analizar su recurrencia a lo largo del tiempo. |
| **RF-GCP-10** | **Análisis de correlación** | El sistema deberá analizar la relación entre las métricas operativas de las antenas y las incidencias reportadas por los usuarios, con el propósito de identificar patrones asociados a interrupciones o degradaciones del servicio. |
| **RF-GCP-11** | **Visualización de información analítica** | El sistema deberá proporcionar una interfaz de visualización que permita consultar indicadores, gráficos y estadísticas sobre el comportamiento de la infraestructura y las incidencias. |
| **RF-GCP-12** | **Visualización geoespacial** | El sistema deberá representar geográficamente la ubicación de las antenas y la distribución de las incidencias, permitiendo identificar zonas críticas y concentraciones de problemas. |

---

### C. Inteligencia Artificial y Predicción

| ID | Requerimiento | Descripción |
| :--- | :--- | :--- |
| **RF-GCP-13** | **Preparación de datos para Machine Learning** | El sistema deberá preparar los datos históricos de infraestructura e incidencias para su utilización en modelos de aprendizaje automático. |
| **RF-GCP-14** | **Entrenamiento de modelos predictivos** | El sistema deberá permitir entrenar modelos de aprendizaje automático utilizando información histórica para identificar patrones asociados a posibles fallas de las antenas. |
| **RF-GCP-15** | **Predicción de fallas** | El sistema deberá estimar la probabilidad de que una antena presente una falla o degradación del servicio dentro de un horizonte temporal determinado. |
| **RF-GCP-16** | **Registro de predicciones** | El sistema deberá almacenar las predicciones generadas, incluyendo la antena evaluada, probabilidad estimada de falla, estado previsto, horizonte temporal y versión del modelo utilizado. |
| **RF-GCP-17** | **Actualización de modelos** | El sistema deberá permitir actualizar y volver a entrenar los modelos predictivos utilizando nuevos datos históricos disponibles. |

---

### D. Integración y Comunicación

| ID | Requerimiento | Descripción |
| :--- | :--- | :--- |
| **RF-GCP-18** | **Exposición de predicciones** | El sistema deberá proporcionar una interfaz de comunicación para que el módulo **AWS** pueda consultar o recibir las predicciones generadas sobre sus antenas. |
| **RF-GCP-19** | **Envío de predicciones preventivas** | El sistema deberá enviar al módulo **AWS** las predicciones que cumplan las condiciones establecidas para generar alertas preventivas. |
| **RF-GCP-20** | **Consulta de resultados analíticos** | El sistema deberá permitir que los usuarios autorizados consulten los resultados de los análisis históricos y predictivos mediante su interfaz de visualización. |
