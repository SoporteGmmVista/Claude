---
name: unir-polizas-gmm-cambios
description: "Une múltiples PDFs de una póliza GMM (Gastos Médicos Mayores) en un solo archivo PDF, siguiendo un orden específico — primero CREDENCIALES, luego CARATULA, después ENDOSOS, y al final cualquier otro documento. El PDF final se guarda en la misma carpeta con el nombre en formato YYYYMMDD POLIZA.pdf usando la fecha actual. Usa esta habilidad siempre que el usuario pida unir pólizas, unir pdfs de póliza, juntar documentos de GMM, combinar póliza, unir caratula credenciales y endosos, o mencione una carpeta de póliza GMM que necesite consolidarse en un solo PDF. Actívala también cuando el usuario se refiera a archivos con nombres tipo GM0000259482 o similares que contengan CARATULA, CREDENCIALES y/o ENDOSOS."
---

# Unir Pólizas GMM Cambios

## Propósito

Consolidar todos los documentos PDF de una póliza de Gastos Médicos Mayores (GMM) en un solo archivo PDF, respetando un orden específico definido por el usuario para facilitar la lectura y archivado de la póliza.

## Orden de unión (obligatorio)

Los PDFs deben unirse en este orden exacto:

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

Usa el script `scripts/unir_polizas.py` bundleado con esta habilidad. Recibe la carpeta como argumento y hace todo el trabajo:

```bash
python3 scripts/unir_polizas.py "/ruta/a/la/carpeta/de/la/poliza"
```

El script:
- lee todos los `.pdf` de la carpeta,
- los clasifica y ordena según las reglas de arriba,
- genera el PDF unido con la fecha de hoy,
- lo guarda en la misma carpeta,
- imprime el orden usado y la ruta del resultado para que puedas confirmarlo al usuario.

Si el script detecta que ya existe un archivo `YYYYMMDD POLIZA.pdf` en la carpeta (por ejemplo por una corrida previa ese mismo día), lo excluye de la entrada para no incluirse a sí mismo en el merge.

## Cómo responderle al usuario

Después de correr el script:

1. Confirma el orden en que se unieron los archivos (el script lo imprime).
2. Comparte un enlace `computer://` al PDF final dentro de la misma carpeta del usuario.
3. Mantén la respuesta breve — el usuario puede abrir el archivo directamente.

## Casos especiales

- **Sin archivos CREDENCIALES**: es posible; simplemente empieza con CARATULA.
- **Sin CARATULA**: avisa al usuario antes de unir, porque normalmente toda póliza trae carátula.
- **Archivos duplicados o con nombres muy parecidos**: mantén el orden alfabético dentro del grupo.
- **La carpeta no existe o está vacía**: avisa al usuario con un mensaje claro.
- **Archivos protegidos con contraseña**: reporta cuál archivo falló para que el usuario pueda resolverlo.

## Por qué este orden

Las aseguradoras en México típicamente arman el expediente de la póliza GMM así: primero las credenciales que el asegurado va a usar día a día, luego la carátula que resume coberturas y sumas aseguradas, después los endosos que modifican la póliza, y al final cualquier anexo. Respetar este orden hace que el documento final sea fácil de hojear y consistente con cómo el usuario entrega la información a sus clientes.
