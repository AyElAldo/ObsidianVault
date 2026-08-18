# FORMATO DE CADENA DE CUSTODIA

## SECCIÓN I: INFORMACIÓN DEL CASO

| Campo                          | Descripción                                               |
|------------------------------- |----------------------------------------------------------|
| Número de Caso / Referencia    | Identificador único asignado al caso                      |
| Fecha de Inicio de Cadena      | dd/mm/aaaa                                               |
| Agencia/Organismo Investigador | Nombre completo                                           |
| Jurisdicción                   | Ciudad / región                                           |
| Tipo de Investigación          | Penal / civil / administrativa / otra                     |
| Tipo de Delito/Asunto          | Describir el delito o asunto                              |
| Breve Descripción              | Resumen de hechos relevantes                              |
| Lugar del Incidente            | Dirección completa / ubicación                            |
| Fecha y Hora del Incidente     | dd/mm/aaaa hh:mm (24h)                                    |

---

## SECCIÓN II: INFORMACIÓN DEL ELEMENTO/EVIDENCIA

| Campo               | Descripción                                   |
|---------------------|-----------------------------------------------|
| Número Único        | Código o folio de la evidencia                |
| Cantidad            | Número de unidades / volumen                  |
| Unidad de Medida    | Unidades, hojas, litros, gramos, KB, etc      |
| Tipo de Evidencia   | Física / digital / biológica / documental     |
| Clasificación       | Arma, droga, dispositivo, documento, etc      |
| Descripción Completa| Color, tamaño, marcas, condición              |
| Características Particulares | Serie, marcas, daños                  |
| Condición inicialmente | Intacta / dañada / alterada / contaminada   |
| Fotografía adjunta  | Sí / No                                       |
| Localización exacta | Coordenadas, croquis, descripción precisa      |

**Evidencia Digital:**  
- Tipo de dispositivo  
- Número de serie  
- Marca/modelo  
- Capacidad  
- Hash criptográfico (SHA-256/MD5)  

**Evidencia Biológica:**  
- Subtipo  
- Volumen aproximado  
- ¿Requiere congelación?  

**Documento:**  
- Formato (original/copia/digital)  
- Cantidad de hojas  
- Idioma  
- Condición  

---

## SECCIÓN III: RECOLECCIÓN INICIAL

| Campo                     | Descripción                                 |
|---------------------------|---------------------------------------------|
| Nombre del Recolector     | Apellidos y nombres completos               |
| Número ID                 | Cédula / matrícula                          |
| Cargo                     | Investigador / perito / policía / otro      |
| Agencia/Dependencia       | Institución                                 |
| Número de Placa/Credencial| Identificación                              |
| Contacto telefónico       | Número                                      |
| Correo electrónico        |                                             |
| Fecha de recolección      | dd/mm/aaaa                                  |
| Hora de recolección       | hh:mm (24h)                                 |
| Autorización              | Judicial / consentimiento / hallazgo        |
| Testigos presentes        | Nombres/roles                               |
| Método de recolección     | Procedimiento específico                    |
| Precauciones tomadas      | Ej.: guantes, equipo estéril, fotografía    |
| Observaciones             | Circunstancias relevantes                   |

---

## SECCIÓN IV: EMBALAJE Y ETIQUETADO

| Campo                   | Descripción                                  |
|-------------------------|----------------------------------------------|
| Tipo de embalaje        | Bolsa, caja, frasco, contenedor especial     |
| Material del embalaje   | Papel, plástico, vidrio, metal               |
| Cantidad de contenedores| Número de contenedores/bolsas                |
| Marcas / nº de contenedor| Ej.: 1 de 3                                  |
| Sello aplicado          | Tipo de sello de seguridad                   |
| Color del sello         |                                              |
| Nº de folio de etiqueta | Identificación                               |
| Descripción breve       |                                              |
| Fecha y hora recolección|                                              |
| Nombre y firma recolector|                                             |
| Peligros especiales     | Biohazard/corrosivo/toxico/etc               |
| ¿Fue sellado?           | Sí / No                                      |
| Integridad del sello    |                                             |

---

## SECCIÓN V: ALMACENAMIENTO TEMPORAL

| Campo                        | Descripción                         |
|------------------------------|-------------------------------------|
| Ubicación almacenamiento     | Lugar específico/bodega/laboratorio |
| Tipo de almacenamiento       | Cámara fría/bóveda/armario seguro   |
| Temperatura requerida        | °C                                  |
| Condiciones ambientales      | Luz/humedad/etc                     |
| Acceso restringido           | Sí / No y quién accede              |
| Sistema de seguridad         | Ej.: video, alarma                  |
| Responsable almacen          | Nombre/firma                        |
| Conservación especial        | Sí / No y especificar               |
| Verificaciones periódicas    | Diario/semanal/etc                  |

---

## SECCIÓN VI: REGISTRO DE CONTINUIDAD (TRANSFERENCIAS)

| Fecha   | Hora  | Entrega (nombre/firma/ID) | Recibe (nombre/firma/ID) | Cargo | Dependencia | Propósito | Observaciones |
|---------|-------|---------------------------|--------------------------|-------|-------------|-----------|--------------|
|         |       |                           |                          |       |             |           |              |
|         |       |                           |                          |       |             |           |              |
|         |       |                           |                          |       |             |           |              |
*Agregar filas según cambios de custodia*

---

## SECCIÓN VII: ANÁLISIS Y PROCESAMIENTO

| Campo                | Descripción                                |
|----------------------|--------------------------------------------|
| Laboratorio / perito | Nombre / dirección / acreditación          |
| Analista             | Nombre / matrícula / especialidad          |
| Fecha de recepción   | dd/mm/aaaa hh:mm                           |
| Fecha análisis       | dd/mm/aaaa                                 |
| Procedimiento        | Descripción                                |
| Resultado / hallazgos| Síntesis                                   |
| Copia de trabajo     | Sí / No                                    |
| Método de copia      | Clonación/imaging/bit-a-bit                |
| Hash de verificación | SHA-256/MD5                                |
| Original preservado  | Sí / No                                    |

---

## SECCIÓN VIII: DISPOSICIÓN FINAL

| Campo                      | Descripción                        |
|----------------------------|------------------------------------|
| Condición actual           | Intacta/deteriorada/parcialmente   |
| Porcentaje utilizado       | 0-100%                             |
| Cantidad remanente         |                                    |
| Observaciones deterioro    |                                    |
| Disposición autorizada     | Devuelto, destruido, archivado     |
| Autorización legal         | Nº de orden/juez/fiscal            |
| Fecha de disposición       | dd/mm/aaaa                         |
| Responsable disposición    | Nombre/firma                       |
| Método de destrucción      | Incineración/trituración/etc       |
| Certificado destrucción    | Nº/referencia                      |
| Receptor final             | Nombre, firma, documentación       |

---

## SECCIÓN IX: CERTIFICACIONES Y FIRMAS

| Elemento                                          | Sí | No | Iniciales |
|---------------------------------------------------|----|----|-----------|
| Cadena de custodia continua                       |    |    |           |
| Documentación completa y legible                  |    |    |           |
| Firmas y sellos presentes                         |    |    |           |
| Almacenamiento adecuado                           |    |    |           |
| Sin indicios de manipulación/contaminación        |    |    |           |

| Rol                      | Nombre | Firma | Fecha | Hora |
|--------------------------|--------|-------|-------|------|
| Recolector principal     |        |       |       |      |
| Testigo                  |        |       |       |      |
| Responsable de bodega    |        |       |       |      |
| Analista / perito        |        |       |       |      |
| Supervisor cadena custodia|       |       |       |      |
| Autoridad competente     |        |       |       |      |

> Certifico bajo protesta de decir verdad que la información contenida en este documento es correcta y completa, que la cadena de custodia ha sido mantenida sin interrupciones, y que la evidencia ha sido tratada de conformidad con protocolos y estándares establecidos.  
> Firma: ________________  Nombre: ________________ Fecha: ________________ Cargo: ________________

---

## SECCIÓN X: NOTAS Y OBSERVACIONES

| Campo                  | Descripción                      |
|------------------------|----------------------------------|
| Incidentes reportados  | Irregularidades                  |
| Daños o pérdidas       | Descripción                      |
| Contactos seguimiento  |                                  |
| Referencias externas   |                                  |
| Notas del gestor       | Comentarios/recomendaciones      |

---

## INSTRUCCIONES DE LLENADO

- Completar todos los campos, usar “N/A” donde no aplique
- Letra legible o digital
- Registrar eventos en orden cronológico
- Firmas originales
- Adjuntar fotografía y documentación extra donde se requiera

---

## APÉNDICES

- Fotografías
- Croquis/mapa
- Órdenes judiciales
- Reportes de laboratorio
- Certificados de verificación digital
- Otra documentación relevante

---


