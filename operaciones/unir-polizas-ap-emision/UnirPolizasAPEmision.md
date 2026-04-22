---
name: unir-polizas-ap-emision
description: "Procesa una carpeta de emisión de póliza AP (Accidentes Personales): primero elimina los PDFs llamados REMISION, TARJETA DE AGENTE y FOLLETO DE DERECHOS, y luego une los PDFs restantes en un solo archivo siguiendo el orden específico de emisión AP — CARTA DE BIENVENIDA, CARATULA DE LA POLIZA, DESIGNACION DE BENEFICIARIOS, CONDICIONES GENERALES, REEMBOLSO GMM, MUERTE ACCIDENTAL, ENDOSO DE SERV ASISTENCIA MED, OTROS ENDOSOS, y al final ENDOSO DE CLAUSULA AGRAVACION. El PDF final se guarda en la misma carpeta con el nombre YYYYMMDD POLIZA AP.pdf usando la fecha actual. Usa esta habilidad siempre que el usuario pida unir pólizas AP, unir una emisión de AP, unir póliza de accidentes personales, procesar una carpeta AP, juntar los documentos de emisión AP, o mencione una carpeta con archivos tipo AP0000685157 que contengan CARTA DE BIENVENIDA, CARATULA, CONDICIONES, ENDOSOS, etc. Actívala también cuando el usuario pida borrar REMISION / TARJETA DE AGENTE / FOLLETO y luego unir el resto en un PDF con fecha."
---

# Unir Pólizas AP Emisión

## Propósito

Automatizar el cierre de una emisión de póliza de **Accidentes Personales (AP)**: limpiar la carpeta de archivos que no forman parte del expediente entregable (remisión, tarjeta del agente, folleto de derechos) y dejar un único PDF consolidado con las secciones en el orden con el que el área de emisión acostumbra entregar la póliza al cliente.

## Qué archivos se eliminan

Antes de unir, deben eliminarse de la carpeta los PDFs cuyo nombre contenga cualquiera de estos términos (case-insensitive, con o sin acentos):

- `REMISION`
- `TARJETA DE AGENTE`
- `FOLLETO` — cubre "FOLLETO DE DERECHOS", "FOLLETO DERECHOS BASICOS...", etc.

Estos archivos no forman parte del expediente final que se le entrega al asegurado.

## Orden de unión (obligatorio)

Los PDFs restantes se unen en este orden exacto. Si alguna sección no existe en la carpeta, simplemente se omite y se continúa con la siguiente.

1. **CARTA DE BIENVENIDA**
2. **CARATULA DE LA POLIZA**
3. **DESIGNACION DE BENEFICIARIOS**
4. **CONDICIONES GENERALES**
5. **REEMBOLSO GMM**
6. **MUERTE ACCIDENTAL**
7. **ENDOSO DE SERV ASISTENCIA MED** (endoso de asistencia médica / ambulancia)
8. **OTROS ENDOSOS** — cualquier endoso adicional que no sea de asistencia ni de agravación
9. **ENDOSO DE CLAUSULA AGRAVACION** — siempre al final, aunque sea el único endoso

La regla clave es que el **endoso de cláusula de agravación del riesgo siempre va al final del documento**, por debajo de cualquier otro endoso. Si aparece algún PDF que no encaje en ninguna categoría explícita, colócalo junto al grupo de "OTROS ENDOSOS" (antes del de agravación).

## Nombre del archivo de salida

Formato exacto: `YYYYMMDD POLIZA AP.pdf`

- `YYYYMMDD` es la fecha actual sin separadores: año (4 dígitos) + mes (2 dígitos) + día (2 dígitos). **Nota: el orden es año-mes-día, no día-mes-año.**
- Un espacio separa la fecha de la palabra `POLIZA AP`.
- `POLIZA AP` va en mayúsculas, sin acento.

Ejemplo: si hoy es 22 de abril de 2026, el archivo se llama `20260422 POLIZA AP.pdf`.

## Ubicación del archivo de salida

El PDF unido se guarda **en la misma carpeta** donde están los PDFs de entrada. No lo muevas a una carpeta de outputs genérica a menos que el usuario lo pida explícitamente.

## Cómo ejecutar

Usa el script `scripts/unir_polizas_ap.py` que viene con esta habilidad. Hace los tres pasos (borrar descartables → clasificar → unir) de una sola pasada:

```bash
python3 scripts/unir_polizas_ap.py "/ruta/a/la/carpeta/de/la/poliza"
```

El script imprime:

- qué archivos se descartaron (REMISION / TARJETA DE AGENTE / FOLLETO),
- el orden en el que se unió cada sección,
- cualquier aviso si falta una sección importante (carta de bienvenida, carátula o condiciones generales),
- la ruta del PDF final y el número total de páginas.

### Permisos de borrado en Cowork

En Cowork, borrar archivos dentro de la carpeta que el usuario seleccionó requiere permiso explícito. Si el primer `os.remove` falla con `PermissionError`, llama al tool `allow_cowork_file_delete` pasándole la ruta de cualquiera de los archivos a borrar, y vuelve a correr el script.

Si por alguna razón no se puede obtener el permiso (o el usuario prefiere no borrar), corre el script con `--no-delete`: los archivos REMISION / TARJETA DE AGENTE / FOLLETO quedan en la carpeta pero se omiten del merge.

```bash
python3 scripts/unir_polizas_ap.py "/ruta/a/la/carpeta" --no-delete
```

## Cómo responderle al usuario

1. Confirma brevemente qué se borró y el orden en que se unió el PDF (el script lo imprime).
2. Comparte un enlace `computer://` directo al PDF final dentro de la carpeta del usuario, por ejemplo:
   `[Ver el PDF](computer:///sessions/.../mnt/<carpeta>/20260422%20POLIZA%20AP.pdf)`
3. Mantén la respuesta breve — el usuario puede abrir el archivo para revisarlo. No describas en prosa lo que ya se ve en el documento.

## Casos especiales

- **No aparece alguna sección (p. ej. no hay MUERTE ACCIDENTAL)**: el script la omite silenciosamente y continúa. Sólo emite aviso si faltan las tres secciones que toda emisión AP normalmente trae: carta de bienvenida, carátula y condiciones generales.
- **Hay más de un endoso "otros"**: se unen todos juntos en la posición 8, ordenados alfabéticamente, y el de agravación sigue yendo al final.
- **Archivos con nombres ambiguos**: el clasificador usa substrings (mayúsculas, sin acentos). Si un archivo tiene un nombre raro que no encaja en ninguna categoría, se coloca en "OTROS" (justo antes del endoso de agravación). Revisa el orden impreso y, si algo quedó en el lugar equivocado, pide al usuario renombrarlo o avísale antes de entregar.
- **Ya existe un `YYYYMMDD POLIZA AP.pdf` del mismo día en la carpeta**: el script lo excluye de la entrada para no auto-incluirse y lo sobreescribe al terminar.
- **Carpeta inexistente o vacía después de descartar**: el script aborta con un mensaje claro.
- **PDFs protegidos con contraseña**: `pypdf` lanzará un error; el script reporta el archivo específico para que el usuario lo resuelva.

## Por qué este orden

Las aseguradoras entregan una emisión de AP siguiendo el recorrido natural de lectura del asegurado: primero la bienvenida, después la carátula con las sumas aseguradas, la designación de beneficiarios, las condiciones generales del contrato, y por último las coberturas adicionales y endosos. El endoso de **agravación del riesgo** se coloca al final porque es una cláusula que modifica responsabilidades del asegurado y conviene que cierre el documento como nota de cumplimiento, no que quede mezclado con las coberturas.
