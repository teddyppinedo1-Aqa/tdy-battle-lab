# TDY Battle-Lab — Guía de Uso del Schema de Cartas v1.0
## Documento de referencia para agentes de Antigravity

---

## CONVENCIONES DE NOMENCLATURA

### Sistema de IDs
Formato: `[CATEGORIA]-[SUBTIPO]-[NUMERO]`

| Categoría | Código | Subtipo | Código |
|---|---|---|---|
| Hongo | FUNG | Microorganismo | MICRO |
| Bacteria | BACT | Compuesto Activo | COMP |
| Virus | VIRU | Virulencia | VIRU |
| Parásito | PARA | Intervención Médica | INTE |
| Fármaco general | FARM | Diagnóstico | DIAG |
| Entorno | ENTR | Búsqueda | BUSC |
| Soporte | SOPO | Entorno | ENTR |

Ejemplos válidos:
- `FUNG-MICRO-001` → Primer microorganismo fúngico
- `FARM-COMP-001` → Primer compuesto activo farmacológico
- `BACT-MICRO-001` → Primer microorganismo bacteriano

---

## RANGOS DE STATS POR RAREZA Y NIVEL

| Rareza | Nivel | ATK range | DEF range |
|---|---|---|---|
| Común | 1-3 | 300–1600 | 100–1200 |
| Poco Común | 3-4 | 1200–2000 | 800–1700 |
| Rara | 4-5 | 1700–2400 | 1200–2200 |
| Ultra Rara | 5-6 | 2200–2800 | 1800–2600 |
| Legendaria | 7-8 | 2600–3500 | 2000–3000 |

Regla: cartas de nivel 5+ tienen `requiere_sacrificio: true`.

---

## REGLAS DE GENERACIÓN PARA AGENTES IA

1. Todo campo marcado como `required` en el schema es OBLIGATORIO.
2. `texto_efecto` debe referenciar mecánicas del reglamento v1.2: REVELACIÓN, Efecto Inmediato, Fenotipo Oculto.
3. `mecanismo_corto` máximo 50 caracteres — contar caracteres antes de incluir.
4. `lore` máximo 120 caracteres — contar caracteres antes de incluir.
5. `espectro` debe incluir al menos 3 organismos para cartas de Compuesto Activo.
6. `fuente.año` mínimo 2018 — no usar fuentes anteriores a 2018.
7. `sinergias` deben referenciar IDs existentes en el repositorio.
8. Para cartas de Soporte: `stats` es null (no tienen ATK/DEF).
9. `prompt_imagen` debe generarse en inglés para mejores resultados con APIs de imagen.
10. `uso_clinico_peru.nota_regional` siempre debe mencionar contexto de Loreto/Amazonía cuando aplique.

---

## TIPOS DE EFECTOS DISPONIBLES EN engine.js

| Tipo | Descripción | Valor esperado |
|---|---|---|
| REVELACION | Efecto al voltear carta de Fenotipo Oculto | Entero (daño/buff) |
| EFECTO_INMEDIATO | No puede ser negado por cadena | Entero |
| DAÑO_DIRECTO | Resta puntos a PSP o CI | Entero positivo |
| CURACION | Suma puntos a PSP o CI | Entero positivo |
| MODIFICAR_ATK | Modifica ATK de una carta | Entero (+ o -) |
| MODIFICAR_DEF | Modifica DEF de una carta | Entero (+ o -) |
| DESTRUIR_CARTA | Destruye una carta objetivo | null |
| ROBAR_CARTA | El jugador roba cartas del mazo | Nro de cartas |
| BUSCAR_MAZO | Busca carta específica en mazo | null |
| NEGAR_EFECTO | Cancela efecto de carta rival | null |
| INMUNIDAD_TURNO | Carta inmune a efectos este turno | null |
| DAÑO_CONTINUO | Daño por turno (Fase Mantenimiento) | Entero por turno |
| SINERGISMO | Bonus cuando otra carta específica está activa | Entero |

---

## PROMPT MAESTRO PARA ANTIGRAVITY

Usar este prompt como base al instruir al agente generador:

```
Eres un generador de cartas para TDY Battle-Lab, un juego de cartas educativo sobre enfermedades infecciosas tropicales. 

Genera UNA carta completa en formato JSON válido que cumpla estrictamente con el schema en card_schema.json.

Carta a generar: [NOMBRE DEL AGENTE O FÁRMACO]
Bando: [Patógeno / Fármaco]
Set: [SET-01-HONGOS / SET-02-BACTERIAS / etc.]
Rareza sugerida: [Común / Poco Común / Rara / Ultra Rara / Legendaria]

Reglas obligatorias:
1. Toda información clínica debe basarse en evidencia real (WHO, MINSA Perú, UpToDate, Mandell 9ed).
2. texto_efecto debe reflejar propiedades clínicas reales del agente.
3. mecanismo_corto máximo 50 caracteres.
4. lore máximo 120 caracteres.
5. prompt_imagen en inglés, estilo "anime científico médico".
6. Incluir nota_regional relevante para Loreto, Perú cuando aplique.
7. Responde SOLO con el JSON. Sin explicaciones, sin markdown, sin texto adicional.
```

---

## ESTRUCTURA DE ARCHIVOS EN REPOSITORIO

```
tdy-battle-lab/
├── schema/
│   ├── card_schema.json          ← Schema maestro (este archivo valida todo)
│   ├── GUIA_AGENTES.md           ← Este documento
│   └── examples/
│       ├── FUNG-MICRO-001_candida_albicans.json
│       └── FARM-COMP-001_fluconazol.json
├── data/
│   ├── SET-01-HONGOS/
│   │   ├── microorganismos/
│   │   └── farmacos/
│   ├── SET-02-BACTERIAS/
│   ├── SET-03-VIRUS/
│   └── SET-04-PARASITOS/
├── assets/
│   └── cards/                    ← PNGs generados
└── src/
    ├── engine.js
    ├── deck.js
    └── ui.js
```
