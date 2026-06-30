# TDY Battle-Lab — Guía de Uso del Schema de Cartas v1.3
## Documento de referencia para agentes de Antigravity

> **Cambios en v1.2 respecto a v1.1:** El proyecto pasa de alcance regional (Perú/Loreto) a alcance **mundial** por defecto. El contexto regional se mantiene como bloque opcional (`uso_clinico_regional`). Los sets dejan de ser monotemáticos (`SET-01-HONGOS`) y pasan a ser **mixtos de 30 cartas** con una distribución fija de categorías clínicas. Se agrega el campo `categoria_clinica` y el uso obligatorio del **índice maestro de cartas** para evitar duplicados. Se elimina el prefijo de ID `FARM` (ver tablas de nomenclatura corregidas más abajo).
>
> **Cambios en v1.3 respecto a v1.2:** Tras validar las dos cartas de ejemplo (Candida albicans y Fluconazol) contra el schema v1.1 con un validador real, se detectaron 5 errores recurrentes que no eran evidentes a simple vista: exceso de caracteres en `texto_efecto`, `lore` y `dato_educativo`, y uso de `null` en campos opcionales (`stats`, `farmacocinetica`, `efectos_adversos`) donde el schema exige que el campo se omita en vez de declararse vacío. Se reforzaron las reglas 2-5 y 9, y se agregó la regla 17 de verificación final obligatoria antes de entregar cualquier carta.

---

## CONVENCIONES DE NOMENCLATURA

### Sistema de IDs
Formato: `[PREFIJO_CATEGORIA]-[PREFIJO_SUBTIPO]-[NUMERO]`

**Estas son dos tablas independientes.** No existe una correspondencia fila-a-fila entre ellas — cualquier categoría clínica puede combinarse con cualquier subtipo de carta. Por ejemplo, "Bacterias" puede generar cartas de subtipo Microorganismo, Compuesto Activo, Virulencia, Intervención Médica o Diagnóstico, no solo uno de ellos.

**Tabla 1 — Prefijo de categoría clínica** (corresponde 1:1 con el enum `categoria_clinica` del schema):

| Categoría clínica | Prefijo |
|---|---|
| Hongos | FUNG |
| Bacterias | BACT |
| Virus | VIRU |
| Parasitos | PARA |
| Entorno | ENTR |
| Soporte | SOPO |

**Tabla 2 — Prefijo de subtipo de carta** (corresponde al `tipo_carta` y a la zona de tablero donde se juega, ver reglamento v1.2):

| Tipo de carta | Prefijo | Zona de tablero |
|---|---|---|
| Microorganismo | MICRO | Confrontación (bando Patógeno) |
| Compuesto Activo | COMP | Confrontación (bando Fármaco) |
| Virulencia | VIRU | Soporte (bando Patógeno) |
| Intervención Médica | INTE | Soporte (bando Fármaco) |
| Diagnóstico | DIAG | Soporte (bando Fármaco) |
| Búsqueda | BUSC | Soporte (bando Patógeno) |
| Entorno | ENTR | Zona de Entorno (Neutral) |

**Regla clave — no existe el prefijo `FARM`:** un fármaco (Compuesto Activo) NO tiene categoría clínica propia. Usa el prefijo de la categoría que combate. Ejemplos:

- Fluconazol (antifúngico) → `FUNG-COMP-001` — categoría Hongos, subtipo Compuesto Activo
- Un antibiótico contra *S. aureus* → `BACT-COMP-001` — categoría Bacterias, subtipo Compuesto Activo
- Un antiparasitario contra *Plasmodium* → `PARA-COMP-001` — categoría Parasitos, subtipo Compuesto Activo

Ejemplos válidos adicionales:
- `FUNG-MICRO-001` → Primer microorganismo fúngico (puede vivir en cualquier set mixto)
- `BACT-MICRO-001` → Primer microorganismo bacteriano
- `VIRU-VIRU-001` → Primer factor de virulencia de origen viral (no confundir las dos "VIRU": la primera es categoría = Virus, la segunda es subtipo = Virulencia)

**Importante:** el número de set (`SET-01`, `SET-02`...) y el número correlativo del ID son independientes entre sí. Una carta `FUNG-MICRO-004` puede terminar publicada en `SET-03` si así lo determina el orden de generación — el ID no predice el set.

---

## REGLA DE COMPOSICIÓN DE SETS MIXTOS (nuevo en v1.2)

Cada set generado contiene **exactamente 30 cartas**, distribuidas según la proporción fija que refleja la carga real de morbimortalidad infecciosa:

| Categoría clínica | % objetivo | Cartas por set de 30 |
|---|---|---|
| Bacterias | 30% | 9 |
| Virus | 30% | 9 |
| Parásitos | 20% | 6 |
| Hongos | 10% | 3 |
| Entorno | 5%* | 2 (sets impares) / 1 (sets pares) |
| Soporte | 5%* | 1 (sets impares) / 2 (sets pares) |

*Entorno y Soporte no dividen exacto en números enteros sobre 30 cartas (5% = 1.5). Para mantener el promedio real en 5%/5% a largo plazo sin romper la regla de 30 cartas fijas, se alterna:
- **Sets impares** (SET-01, SET-03, SET-05...): 2 Entorno + 1 Soporte
- **Sets pares** (SET-02, SET-04, SET-06...): 1 Entorno + 2 Soporte

Total por set: 9 + 9 + 6 + 3 + 3 = 30. ✅

**Antes de generar un set**, el agente debe declarar la tabla de cartas a generar usando esta distribución, y verificarla contra el índice maestro (ver abajo) para confirmar que ningún nombre se repite respecto a sets anteriores.

---

## ÍNDICE MAESTRO DE CARTAS (obligatorio desde v1.2)

Archivo: `schema/indice_maestro_cartas.json`

**Antes de generar cualquier carta nueva**, el agente DEBE:

1. Normalizar el nombre del agente/fármaco a generar (sin tildes, minúsculas, sin espacios extra).
2. Buscar ese nombre en la lista `cartas` del índice maestro.
3. Si el nombre **ya existe** → detener la generación de esa carta puntual y alertar al usuario con el ID ya asignado. No generar un ID nuevo para el mismo agente/fármaco aunque el set sea distinto.
4. Si el nombre **no existe** → generar la carta normalmente y, al finalizar, agregar una entrada nueva al índice con: `id`, `nombre`, `categoria_clinica`, `set`, `tipo_carta`.
5. Actualizar `conteo_por_categoria` y `conteo_por_set` en el índice tras cada carta agregada.

Esto evita dos problemas distintos: IDs duplicados (control de formato) y **contenido clínico duplicado bajo nombres o IDs diferentes** (control de fondo, más importante).

---

## RANGOS DE STATS POR RAREZA Y NIVEL

(Sin cambios respecto a v1.1 — esta tabla aplica igual sin importar la categoría clínica de la carta.)

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
2. `texto_efecto` debe referenciar mecánicas del reglamento v1.2 (REVELACIÓN, Efecto Inmediato, Fenotipo Oculto) y tiene un **límite duro de 200 caracteres**. Contar caracteres ANTES de escribir el texto final, no después — escribir primero la versión completa y luego recortarla suele dejar el texto por encima del límite y obliga a reescribir dos veces.
3. `mecanismo_corto` tiene un **límite duro de 50 caracteres**. Contar antes de incluir.
4. `lore` tiene un **límite duro de 120 caracteres**. Contar antes de incluir. Este es el campo donde más frecuentemente se excede el límite por intentar combinar dato clínico + narrativa en una sola frase — preferir una frase corta y contundente sobre una frase larga y descriptiva.
5. `dato_educativo` tiene un **límite duro de 300 caracteres**. Contar antes de incluir. Preferir un solo dato memorable bien explicado sobre varios datos encadenados.
6. `espectro` debe incluir al menos 3 organismos para cartas de Compuesto Activo.
7. `fuente.año` mínimo 2018 — no usar fuentes anteriores a 2018.
8. `sinergias` deben referenciar IDs existentes en el repositorio (consultar índice maestro).
9. Para cartas de Soporte: el campo `stats` se OMITE por completo (no se incluye en el JSON). No usar `"stats": null` — el schema exige `"type": "object"` para `stats` y rechaza `null` como valor. La misma regla aplica a cualquier campo opcional sin datos: si no aplica, se omite el campo entero, nunca se declara en `null`. Esto incluye `farmacocinetica` y `efectos_adversos` en cartas de Microorganismo (no tienen farmacocinética ni efectos adversos propios — se omiten, no se ponen en null).
10. `prompt_imagen` debe generarse en inglés para mejores resultados con APIs de imagen.
11. `meta.categoria_clinica` es OBLIGATORIO y debe coincidir con el prefijo del `id` (ej. un ID `BACT-*` debe tener `categoria_clinica: "Bacterias"`). Para cartas de Compuesto Activo (fármacos): la categoría es la del patógeno que combaten, NO existe categoría "Fármacos" — un antifúngico usa prefijo `FUNG` y `categoria_clinica: "Hongos"`, no `FARM`.
12. `meta.set` debe seguir el formato `SET-NN` (sin sufijo de categoría) y debe respetar la distribución de la tabla de composición de sets mixtos.
13. **Antes de generar:** consultar el índice maestro para evitar duplicados (ver sección dedicada arriba).
14. La información clínica es de **alcance mundial por defecto**. NO asumir contexto de ningún país en `mecanismo_accion_completo`, `indicaciones_principales`, `efectos_adversos` ni `resistencia` — esos campos deben ser válidos para cualquier lector informado, sin importar su país.
15. `uso_clinico_regional` es OPCIONAL y debe usarse solo cuando exista una razón clínica real para un país/región específico (ej. una micosis endémica de cierta zona, una particularidad de disponibilidad en cierto sistema de salud). No forzar un bloque regional en cartas donde no aplica.
16. Si se incluye `uso_clinico_regional`, el sub-campo `nota_regional` debe ser específico y verificable (cita fuente o normativa), no una generalización vaga.
17. **Verificación final obligatoria antes de entregar cada carta:** contar caracteres de `texto_efecto` (≤200), `mecanismo_corto` (≤50), `lore` (≤120) y `dato_educativo` (≤300); confirmar que ningún campo opcional sin datos quedó como `null` en vez de omitido; confirmar que `categoria_clinica` coincide con el prefijo del `id`. Esta carta NO se considera terminada hasta pasar esta verificación — los 5 errores más frecuentes detectados en la validación de la Sesión 3 fueron justamente estos.

---

## TIPOS DE EFECTOS DISPONIBLES EN engine.js

(Sin cambios respecto a v1.1.)

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
Eres un generador de cartas para TDY Battle-Lab, un juego de cartas educativo sobre
enfermedades infecciosas tropicales y de relevancia mundial.

Antes de generar, consulta schema/indice_maestro_cartas.json y confirma que el
agente/fármaco solicitado no esté ya registrado bajo otro ID.

Genera UNA carta completa en formato JSON válido que cumpla estrictamente con el
schema en card_schema.json (v1.1).

Carta a generar: [NOMBRE DEL AGENTE O FÁRMACO]
Bando: [Patógeno / Fármaco]
Categoría clínica: [Hongos / Bacterias / Virus / Parasitos / Entorno / Soporte]
Set destino: [SET-NN]
Rareza sugerida: [Común / Poco Común / Rara / Ultra Rara / Legendaria]

Reglas obligatorias:
1. Toda información clínica debe basarse en evidencia real (WHO, UpToDate, Mandell 9ed,
   y normativa nacional solo cuando el dato sea específicamente regional).
2. La información debe ser de alcance mundial por defecto. No asumir el contexto de
   ningún país salvo que se use explícitamente el bloque opcional uso_clinico_regional.
3. texto_efecto debe reflejar propiedades clínicas reales del agente.
4. mecanismo_corto máximo 50 caracteres.
5. lore máximo 120 caracteres.
6. prompt_imagen en inglés, estilo "anime científico médico".
7. meta.categoria_clinica debe coincidir con el prefijo del ID generado.
8. meta.set debe usar el formato SET-NN (sin sufijo de categoría).
9. Si aplica una particularidad regional real (ej. micosis endémica amazónica),
   documentarla en uso_clinico_regional — nunca forzarla si no aplica.
10. Responde SOLO con el JSON. Sin explicaciones, sin markdown, sin texto adicional.
```

---

## ESTRUCTURA DE ARCHIVOS EN REPOSITORIO

> **Cambio en v1.2:** las carpetas de `data/` ya no se organizan por set monotemático (`SET-01-HONGOS`), sino por número de set mixto. Dentro de cada set, las 4 subcarpetas uniformes (`microorganismos/`, `farmacos/`, `soporte/`, `entorno/`) se mantienen igual que en v1.1 — eso no cambia, solo cambia qué categorías clínicas conviven dentro de cada set.

```
tdy-battle-lab/
├── schema/
│   ├── card_schema.json              ← Schema maestro v1.1 (valida todo)
│   ├── GUIA_AGENTES.md               ← Este documento (v1.3)
│   ├── indice_maestro_cartas.json    ← Índice global anti-duplicados + conteo por categoría
│   └── examples/
│       ├── FUNG-MICRO-001_candida_albicans.json   (a regenerar bajo schema v1.1)
│       └── FUNG-COMP-001_fluconazol.json          (a regenerar bajo schema v1.1, antes FARM-COMP-001)
├── data/
│   ├── SET-01/
│   │   ├── microorganismos/   ← Bacterias, Virus, Parasitos, Hongos (Microorganismo)
│   │   ├── farmacos/          ← Compuestos Activos de cualquier categoría
│   │   ├── soporte/           ← Virulencia, Intervención Médica, Diagnóstico, Búsqueda
│   │   └── entorno/           ← Cartas de Entorno
│   ├── SET-02/
│   │   ├── microorganismos/
│   │   ├── farmacos/
│   │   ├── soporte/
│   │   └── entorno/
│   └── ...
├── assets/
│   └── cards/                        ← PNGs generados
└── src/
    ├── engine.js
    ├── deck.js
    └── ui.js
```
