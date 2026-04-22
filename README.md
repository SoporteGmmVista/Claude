# 📚 Company Skills

Repositorio central de skills para Claude. Cada skill le enseña a Claude a ejecutar tareas específicas de forma automática — solo descríbele lo que necesitas y él sabe qué pasos seguir.

---

## 🗂️ Cómo está organizado

```
Claude/
├── README.md               ← estás aquí
├── operaciones/            ← skills del área de Operaciones
│   ├── unir-polizas-ap-emision/
│   └── unir-polizas-gmm-cambios/
├── marketing/              ← skills del área de Marketing
│   └── (próximamente)
└── shared/                 ← skills que usa cualquier área
    └── (próximamente)
```

---

## 🚀 Cómo usar un skill

1. Entra a [Claude.ai](https://claude.ai) → **Settings → Skills**
2. Agrega la URL raw del archivo `SKILL.md` del skill que quieres activar
3. Listo — Claude lo usará automáticamente cuando lo necesites

> 💡 No necesitas mencionar el nombre del skill. Claude detecta cuándo aplicarlo por el contexto de tu mensaje.

---

## 📦 Skills disponibles

### Operaciones

| Skill | ¿Qué hace? | Actívalo diciendo… |
|---|---|---|
| [unir-polizas-ap-emision](./operaciones/unir-polizas-ap-emision/SKILL.md) | Limpia y une los PDFs de una emisión de póliza AP en un solo archivo ordenado | *"Une esta carpeta de póliza AP"* |
| [unir-polizas-gmm-cambios](./operaciones/unir-polizas-gmm-cambios/SKILL.md) | Une los PDFs de una póliza GMM (carátula, credenciales, endosos) en el orden correcto | *"Junta los documentos de esta póliza GMM"* |

### Marketing

| Skill | ¿Qué hace? | Actívalo diciendo… |
|---|---|---|
| *(vacío por ahora)* | — | — |

### Shared — skills transversales

| Skill | ¿Qué hace? | Actívalo diciendo… |
|---|---|---|
| *(vacío por ahora)* | — | — |

---

## ➕ Cómo agregar un skill nuevo

1. Crea una carpeta en el área correspondiente: `operaciones/nombre-del-skill/`
2. Adentro pon al menos:
   ```
   nombre-del-skill/
   ├── SKILL.md        ← obligatorio (instrucciones para Claude)
   └── examples/       ← opcional pero recomendado
   ```
3. Agrega una fila en la tabla de arriba con descripción y frase de activación
4. Abre un Pull Request para que el equipo lo revise antes de publicar

### Estructura mínima de un SKILL.md

```markdown
---
name: nombre-del-skill
description: "Descripción corta que Claude usará para saber cuándo activar este skill."
---

# Nombre del Skill

## Propósito
Qué problema resuelve y cuándo se usa.

## Pasos
1. Paso uno
2. Paso dos
...
```

---

## 🔄 Flujo de trabajo del equipo

```
main          → versión estable, solo skills aprobados
└── operaciones/... → rama para proponer cambios de ese área
└── marketing/...   → rama para proponer cambios de ese área
```

- Cada área trabaja en su propia rama
- Los cambios entran a `main` solo por Pull Request
- El responsable del área aprueba antes de hacer merge

---

## 🙋 ¿Dudas o mejoras?

Abre un **Issue** en este repositorio describiendo:
- Qué tarea quisieras automatizar
- En qué área aplica
- Con qué frecuencia la haría tu equipo

---

*Última actualización: abril 2026*
