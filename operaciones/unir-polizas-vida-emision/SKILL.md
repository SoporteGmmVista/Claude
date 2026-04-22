---
name: unir-polizas-vida-emision
description: "Procesa una carpeta de emisión de póliza de VIDA: elimina los PDFs llamados REMISION, TARJETA DE AGENTE y FOLLETO, y une los restantes en un solo archivo siguiendo el orden oficial de emisión de vida — CARTA DE BIENVENIDA, CARATULA, TABLAS DE SUMA ASEGURADA Y VALORES GARANTIZADOS, TABLA DE PAGO DE PRIMAS, CONDICIONES GENERALES, BIT, BAIT/BITAE, BMA, BAM, AV, otros endosos, más la sección AVE si aplica y cualquier endoso suelto al final. El PDF final se guarda en la misma carpeta como AAMMDD POLIZA.pdf (año de DOS dígitos). Usa esta habilidad siempre que el usuario pida unir una póliza de vida, unir una emisión de vida, procesar la carpeta de emisión de vida, juntar CARTA DE BIENVENIDA + CARATULA + CONDICIONES + BIT/BAIT/BMA/BAM/AV, o mencione archivos tipo VI0002985084 con CARATULA IMAGINA SER, BIT, BAIT, BMA, BAM, AV, ENDOSO DE DISTRIBUCION, etc. Actívala también cuando pida borrar REMISION / TARJETA DE AGENTE / FOLLETO de una carpeta de vida y luego unir el resto en un PDF con fecha."
---

# Unir Pólizas Vida Emisión

## Propósito

Automatizar el cierre de una emisión de póliza de **Vida** (por ejemplo, Imagina Ser): limpiar la carpeta de archivos que no forman parte del expediente entregable (remisión, tarjeta del agente, folleto de derechos) y dejar un único PDF consolidado con las secciones en el orden con el que el área de emisión entrega la póliza al cliente.

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
3. **TABLA DE SUMA ASEGURADA Y VALORES GARANTIZADOS** — aquí entran también "TABLA DE VALOR EN EFECTIVO" y "TABLA DE COSTOS DE SEGURO" (son las tablas anexas a la carátula).
4. **TABLA DE PAGO DE PRIMAS** — también aparece como "PLAN DE PAGO DE PRIMAS".
5. **CLAUSULA MANCOMUNADO**
6. **CONDICIONES GENERALES**
7. **ENDOSO EXCLUSION** — ojo: en muchas emisiones ya viene incluido dentro de las condiciones generales y no llega como archivo separado.
8. **CARTA DE CONDICIONES DE REHABILITACION**
9. **ENDOSO DE PRESCRIPCION**
10. **ENDOSO DE TRANSCRIPCION**
11. **BIT** — Exención de pago de primas por invalidez.
12. **BAIT / BITAE** — Beneficio de pago adicional de suma asegurada por invalidez.
13. **BMA** — Beneficio por muerte accidental.
14. **BAM** — Cláusula adicional de asistencia médica.
15. **AV** — Cláusula adicional de apoyo en vida (puede excluirse por extraprima).
16. **CLAUSULA NO FUMADOR**
17. **ENDOSO ADICIONAL RELATIVO A LA PRESCRIPCION**
18. **CARTA EXTRAPRIMA** — sólo si la póliza la trae.

### Sección AVE (sólo si la póliza trae Aumento de Valor en Efectivo)

Si la emisión incluye AVE, estas piezas van *después* del bloque 1–18, en este orden:

19. **ENDOSO DE VALOR ADICIONAL / AUMENTO DE VALOR EN EFECTIVO**
20. **TABLA DE SUMA ASEGURADA Y VALORES GARANTIZADOS AVE**
21. **RECORDATORIO DE PAGO AVE**
22. **TABLA DE SUMA ASEGURADA Y VALORES GARANTIZADOS AVE CP**
23. **RECORDATORIO DE PAGO AVE CP**

### Otros documentos

Cualquier PDF que no encaje en ninguna categoría explícita (por ejemplo `ENDOSO DE DISTRIBUCION`, endosos sueltos, etc.) se coloca al final del documento, después de AVE. Esto evita perder información y mantiene los endosos atípicos agrupados en la cola.

## Nombre del archivo de salida

Formato exacto: `AAMMDD POLIZA.pdf`

- `AAMMDD` es la fecha actual con **año de dos dígitos** (YY) + mes (2 dígitos) + día (2 dígitos). **Esto es distinto a la póliza AP, que usa año de 4 dígitos.** Si el usuario pide explícitamente 4 dígitos, usa el flag `--year4`.
- Un espacio separa la fecha de la palabra `POLIZA`.
- `POLIZA` va en mayúsculas, sin acento.

Ejemplo: si hoy es 22 de abril de 2026, el archivo se llama `260422 POLIZA.pdf`.

## Ubicación del archivo de salida

El PDF unido se guarda **en la misma carpeta** donde están los PDFs de entrada. No lo muevas a una carpeta de outputs genérica a menos que el usuario lo pida explícitamente.

## Cómo ejecutar

Usa el script `scripts/unir_polizas_vida.py` que viene con esta habilidad. Hace los tres pasos (borrar descartables → clasificar → unir) de una sola pasada:

```bash
python3 scripts/unir_polizas_vida.py "/ruta/a/la/carpeta/de/la/poliza"
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
python3 scripts/unir_polizas_vida.py "/ruta/a/la/carpeta" --no-delete
```

## Cómo responderle al usuario

1. Confirma brevemente qué se borró y el orden en que se unió el PDF (el script lo imprime).
2. Comparte un enlace `computer://` directo al PDF final dentro de la carpeta del usuario, por ejemplo:
   `[Ver el PDF](computer:///sessions/.../mnt/<carpeta>/260422%20POLIZA.pdf)`
3. Mantén la respuesta breve — el usuario puede abrir el archivo para revisarlo. No describas en prosa lo que ya se ve en el documento.

## Casos especiales

- **Nombres de archivo con variantes**: los PDFs de Vida suelen tener nombres largos tipo `VI0002985084_CLAUSULA ADICIONAL DE APOYO EN VIDA (AV)_217602643.pdf`. El clasificador busca las siglas/keywords (`AV`, `BIT`, `BAIT`, `BMA`, `BAM`, `CARATULA`, etc.) dentro del nombre normalizado, así que estas variantes funcionan bien.
- **Tablas duplicadas**: "TABLA DE VALOR EN EFECTIVO" y "TABLA DE COSTOS DE SEGURO" se unen ambas en la posición 3 (TABLA DE SUMA ASEGURADA Y VALORES GARANTIZADOS). Si hay una sola tabla, también queda ahí.
- **Sección AVE ausente**: si no hay PDFs de AVE, simplemente se salta y se sigue con los documentos "otros".
- **ENDOSO DE DISTRIBUCION u otros endosos sueltos**: si no encajan en ninguna categoría explícita se agrupan al final del documento. Revisa el orden impreso por el script y avísale al usuario si algo quedó colocado raro.
- **Ya existe un `AAMMDD POLIZA.pdf` del mismo día en la carpeta**: el script lo excluye de la entrada para no auto-incluirse y lo sobreescribe al terminar.
- **Carpeta inexistente o vacía después de descartar**: el script aborta con un mensaje claro.
- **PDFs protegidos con contraseña**: `pypdf` lanzará un error; el script reporta el archivo específico para que el usuario lo resuelva.

## Por qué este orden

Las aseguradoras entregan una emisión de vida siguiendo el recorrido natural del contrato: primero la bienvenida, después la carátula con las sumas aseguradas y sus tablas anexas, el plan de pago de primas, la cláusula mancomunada cuando aplica, y las condiciones generales. Vienen entonces las cartas y endosos de gestión del contrato (rehabilitación, prescripción, transcripción). A continuación se agregan las coberturas adicionales por siglas — primero las de invalidez (BIT, BAIT), luego las de muerte accidental, asistencia médica y apoyo en vida —, la cláusula de no fumador si aplica, el endoso adicional relativo a la prescripción, y al final cualquier carta de extraprima. Si la póliza contrata Aumento de Valor en Efectivo (AVE), sus tablas y recordatorios van después del bloque base. Los endosos inusuales (p. ej. `ENDOSO DE DISTRIBUCION`) quedan al final del documento como cola, lo que deja el expediente listo para entregar al asegurado sin perder información.
