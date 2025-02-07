# 📄 CL BUPA - Estándar de Integración HL7

## **1. Información General**
El documento define los lineamientos para la integración de sistemas entre **BUPALAB** y **LIS de Dedalus**, utilizando el estándar de mensajería **HL7 versión 2.5** sobre conexiones **TCP/IP**.

Se describen los procesos para:
- **Intercambio de mensajes HL7**  
- **Confirmación de recepción (ACK)**  
- **Definición de flujos de trabajo clínicos**

---

## **2. Listado de Integraciones HL7**

| **#** | **Integración**                     | **Tipo de Mensaje HL7** | **Origen** | **Destino** |
|-------|------------------------------------|-------------------------|------------|-------------|
| 1     | Creación de Solicitud (NW)         | OML                     | HIS        | LIS         |
| 2     | Cancelación/Anulación de Solicitud (CA) | OML               | HIS        | LIS         |
| 3     | Cambios de Estado (SC)             | OML                     | LIS        | HIS         |
| 4     | Informe de Resultados (PDF Base64) | MDM                     | LIS        | HIS         |
| 5     | Resultados Estructurados           | ORU                     | LIS        | HIS         |

---

## **3. Estructura de Mensajes HL7**

### 🔹 **Segmentos Comunes (MSH, PID, PV1)**
Estos segmentos están presentes en todos los tipos de mensajes.
- **MSH (Message Header):** Define metadatos del mensaje (origen, destino, tipo de mensaje).
- **PID (Patient Identification):** Contiene la información del paciente.
- **PV1 (Patient Visit):** Datos relacionados con la visita del paciente.

#### ✅ **Ejemplo de Segmento MSH:**
```hl7
MSH|^~\&|CLC|CLC|DNLAB|LIS|20230522141200||OML^O21^OML_O21|000055|P|2.5||||||UTF8
```
- **OML^O21:** Mensaje para la creación de órdenes de laboratorio.
- **000055:** ID único del mensaje.
- **2.5:** Versión del protocolo HL7.

#### ✅ **Ejemplo de Segmento PID:**
```hl7
PID|1||259068274^^^^RU||RODRIGUEZ^ELIANA||19901019|F|||SANTIAGO||955669988
```
- **259068274:** RUT del paciente.
- **RODRIGUEZ ELIANA:** Nombre del paciente.
- **19901019:** Fecha de nacimiento.
- **F:** Género femenino.

---

## **4. Flujos de Integración y Ejemplos**

### 🚀 **4.1 Creación de Solicitudes (NW)**
Este mensaje crea una orden de laboratorio.

#### **Metodología:**
- **Tipo de mensaje:** `OML^O21`
- **Segmentos:** `MSH`, `PID`, `PV1`, `ORC`, `TQ1`, `OBR`, `OBX`

#### ✅ **Ejemplo de Creación de Solicitud:**
```hl7
MSH|^~\&|CLC|CLC|DNLAB|LIS|20230522141200||OML^O21^OML_O21|000055|P|2.5
PID|1||259068274^^^^RU||RODRIGUEZ^ELIANA||19901019|F
PV1|1|A|FABCLCLB
ORC|NW|CLC00005501||CLC000055^000055|IP|||||||1111111111^MEDICO^NOMBRE|||20230522140500
OBR|1|CLC00005501||0305029ZT^PANEL MOLECULAR POLENES|||20230522140500
```

#### **Respuesta de Aceptación (ACK):**
```hl7
MSH|^~\&|DNLAB|LIS|CLC|CLC|20221130155715||ACK|78z26380102|P|2.5
MSA|AA|78z26380102|CREACION DE SOLICITUD CORRECTA|30010000
```

---

### ❌ **4.2 Cancelación/Anulación de Solicitud (CA)**
Permite cancelar parcial o totalmente una solicitud de laboratorio.

#### ✅ **Ejemplo de Cancelación Completa:**
```hl7
MSH|^~\&|CLC|CLC|DNLAB|LIS|20230522141200||OML^O21^OML_O21|000055|P|2.5
PID|1||PASABC003^^^^PS||PACIENTE^PRUEBA||19670315|F
ORC|CA|CLC00005501||CLC000055^000055|CA|||||||1111111111^MEDICO^NOMBRE|||20230522140500
OBR|1|CLC00005501||0305029ZT^PANEL MOLECULAR POLENES|||20230522140500
```

---

### 🔄 **4.3 Cambios de Estado (SC)**
Actualiza el estado de una solicitud, como **toma de muestra**, **muestra enviada**, o **check-in**.

#### ✅ **Ejemplo de Cambio de Estado a “Toma de Muestra”:**
```hl7
MSH|^~\&|DNLAB|LIS|CLC|CLC|20220520191441||OML^O21|20052022191441121000|P|2.5
ORC|SC|CLC00014604|60004339|CLC000146^000146|TM||||20230920110000
OBR|1|CLC00014604||0302015-001^CALCIO||20230919153200
SPM|1|6000433901||20^SANGRE||||20230919153200
```

---

### 📊 **4.4 Informe de Resultados (PDF en Base64)**
Envía resultados en formato PDF codificado en Base64.

#### ✅ **Ejemplo de Informe de Resultados:**
```hl7
MSH|^~\&|DNLAB|LIS|CLC|CLS|20210422165631||MDM^T04^MDM_T04|220420211656318|P|2.5
ORC|NW|10034639|20035638||CM||||20191022191229
OBX|1|ED|PDF_REPORT^PDF_REPORT||aHR0cDovL2V4YW1wbGUuY29tL3Jlc3VsdC5wZGY=|||||F
```

---

### 🧪 **4.5 Resultados Estructurados (ORU^R01)**
Envía resultados estructurados, como análisis clínicos y microbiológicos.

#### ✅ **Ejemplo de Resultados de Laboratorio:**
```hl7
MSH|^~\&|DNLAB|LIS|CLC|CLC|20230921154815||ORU^R01^ORU_R01|21092023154815083000|P|2.5
ORC|SC|CLC00014604|60004339|CLC000146^000146|CM||||20230920110000
OBR|1|CLC00014604||26312^PERFIL BIOQUIMICO
OBX|1|NM|GLU_SP^GLUCOSA||90|mg/dL|70-99|N|||F
```

---

## ⚠️ **Consideraciones Técnicas Importantes**
- **Control de Errores:** Se devuelve un **ACK** para cada mensaje recibido.
- **Homologación de Códigos:** Códigos de exámenes, microorganismos y antibióticos deben estar homologados.
- **Rechazo de Solicitudes:** No se permiten cancelaciones si existen resultados validados.

---


