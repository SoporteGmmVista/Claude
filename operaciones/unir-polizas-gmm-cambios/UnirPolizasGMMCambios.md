---
name: unir-polizas-gmm-cambios
description: "Procesa una carpeta de póliza GMM (Gastos Médicos Mayores): primero elimina los PDFs llamados REMISION y TARJETA DE AGENTE, y luego une los PDFs restantes en un solo archivo siguiendo un orden específico — primero CREDENCIALES, luego CARATULA, después ENDOSOS, y al final cualquier otro documento. El PDF final se guarda en la misma carpeta con el nombre en formato YYYYMMDD POLIZA.pdf usando la fecha actual. Usa esta habilidad siempre que el usuario pida unir pólizas, unir pdfs de póliza, juntar documentos de GMM, combinar póliza, unir caratula credenciales y endosos, o mencione una carpeta de póliza GMM que necesite consolidarse en un solo PDF. Actívala también cuando el usuario pida borrar REMISION / TARJETA DE AGENTE de una carpeta GMM y luego unir el resto en un PDF con fecha, o cuando se refiera a archivos con nombres tipo GM0000259482 o similares que contengan CARATULA, CREDENCIALES y/o ENDOSOS."
---

# Unir Pólizas GMM Cambios

## Propósito

Consolidar todos los documentos PDF de una póliza de Gastos Médicos Mayores (GMM) en un solo archivo PDF, respetando un orden específico definido por el usuario para facilitar la lectura y archivado de la póliza. Antes de unir, se limpia la carpeta de archivos que no forman parte del expediente entregable (remisión y tarjeta del agente).

## Qué archivos se eliminan

Antes de unir, deben eliminarse de la carpeta los PDFs cuyo nombre contenga cualquiera de estos términos (case-insensitive, con o sin acentos):

- `REMISION`
- `TARJETA DE AGENTE`

Estos archivos no forman parte del expediente final que se le entrega al asegurado.

## Orden de unión (obligatorio)

Los PDFs restantes deben unirse en este orden exacto:

1. **CREDENCIALES** — todos los archivos cuyo nombre contenga "CREDENCIALES" (puede haber varios, una por asegurado)
2. **CARATULA** — el archivo cuyo nombre contenga "CARATULA"
3. **ENDOSOS** — todos los archivos cuyo nombre contenga "ENDOSO" (inclusiones, exclusiones, etc.)
4. **Otros documentos** — cualquier otro PDF que no caiga en las categorías anteriores

Dentro de cada grupo, los archivos se ordenan alfabéticamente por nombre para que el resultado sea reproducible.

## Nombre del archivo de salida

Formato exacto: `YYYYMMDD POLIZA.pdf`

- `YYYYMMDD` es la fecha actual sin guiones ni otros separadores (año completo, mes con dos dígitos, día con dos dígitos)
- Un espacio separa la fecha de la palabra `POLIZA`
- La palabra `POLIZA` va en mayúsculas, sin acento

Ejemplo: si hoy es 22 de abril de 2026, el archivo se llama `20260422 POLIZA.pdf`.

## Ubicación del archivo de salida

El PDF unido debe guardarse **en la misma carpeta** donde están los PDFs de entrada. No se mueve a una carpeta de outputs genérica a menos que el usuario lo pida explícitamente.

## Cómo ejecutar

Usa el script `scripts/unir_polizas.py` bundleado con esta habilidad. Hace los tres pasos (borrar descartables → clasificar → unir) de una sola pasada:

```bash
python3 scripts/unir_polizas.py "/ruta/a/la/carpeta/de/la/poliza"
```

El script:
- lee todos los `.pdf` de la carpeta,
- separa y borra los archivos REMISION / TARJETA DE AGENTE,
- clasifica y ordena el resto según las reglas de arriba,
- genera el PDF unido con la fecha de hoy,
- lo guarda en la misma carpeta,
- imprime qué se borró, el orden usado y la ruta del resultado para que puedas confirmarlo al usuario.

Si el script detecta que ya existe un archivo `YYYYMMDD POLIZA.pdf` en la carpeta (por ejemplo por una corrida previa ese mismo día), lo excluye de la entrada para no incluirse a sí mismo en el merge.

### Permisos de borrado en Cowork

En Cowork, borrar archivos dentro de la carpeta que el usuario seleccionó requiere permiso explícito. Si el primer `os.remove` falla con `PermissionError`, llama al tool `allow_cowork_file_delete` pasándole la ruta de cualquiera de los archivos a borrar, y vuelve a correr el script.

Si por alguna razón no se puede obtener el permiso (o el usuario prefiere no borrar), corre el script con `--no-delete`: los archivos REMISION / TARJETA DE AGENTE quedan en la carpeta pero se omiten del merge.

```bash
python3 scripts/unir_polizas.py "/ruta/a/la/carpeta" --no-delete
```

## Cómo responderle al usuario

Después de correr el script:

1. Confirma brevemente qué se borró y el orden en que se unieron los archivos (el script lo imprime).
2. Comparte un enlace `computer://` al PDF final dentro de la misma carpeta del usuario, por ejemplo:
   `[Ver el PDF](computer:///sessions/.../mnt/<carpeta>/20260422%20POLIZA.pdf)`
3. Mantén la respuesta breve — el usuario puede abrir el archivo directamente.

## Casos especiales

- **Sin archivos CREDENCIALES**: es posible; simplemente empieza con CARATULA.
- **Sin CARATULA**: avisa al usuario antes de unir, porque normalmente toda póliza trae carátula — pudo haber un archivo faltante.
- **Archivos duplicados o con nombres muy parecidos**: mantén el orden alfabético dentro del grupo; no intentes deduplicar por contenido.
- **La carpeta no existe o está vacía**: avisa al usuario con un mensaje claro.
- **Carpeta vacía después de descartar REMISION / TARJETA DE AGENTE**: el script aborta con un mensaje claro.
- **Archivos protegidos con contraseña**: `pypdf` lanzará un error; reporta cuál archivo falló para que el usuario pueda resolverlo.

## Por qué este orden

Las aseguradoras en México típicamente arman el expediente de la póliza GMM así: primero las credenciales que el asegurado va a usar día a día, luego la carátula que resume coberturas y sumas aseguradas, después los endosos que modifican la póliza, y al final cualquier anexo. La remisión y la tarjeta del agente son documentos internos que no forman parte del expediente del asegurado, por eso se descartan antes de unir. Respetar este orden hace que el documento final sea fácil de hojear y consistente con cómo el usuario entrega la información a sus clientes.
