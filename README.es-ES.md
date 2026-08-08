

# cc-statusline

Una línea de estado de dos líneas para [Claude Code](https://code.claude.com), escrita en **shell POSIX puro + awk** — no se requiere `node`, `bun` ni `jq`. Lee todo el JSON de estado que Claude Code envía por `stdin`, por lo que se ejecuta como un solo proceso rápido y funciona en una máquina con instalación nativa fresca.

```
Opus 4.8 [xhigh] ~ ⎈ prod(api) feat/x (+3)
20:45:17 | ⛁ ███░░░░░░░ 32% | 5h ◔○○○○ 6% ↻2h15m | 7d ●○○○○○○ 15% ↻5d14h │ $0.61 ⏱ 2m
```

**Línea 1** — modelo, esfuerzo de razonamiento opcional, directorio de trabajo, contexto opcional de kubectl, rama de git + contador de cambios no confirmados.

**Línea 2** — reloj (ubicado debajo del nombre del modelo), barra de ventana de contexto ⛁, luego las ventanas de límite de velocidad de 5 horas y 7 días, y finalmente el costo y la duración de la sesión.

## Los puntos del límite de velocidad

Cada ventana es una fila de puntos: **5 puntos para la ventana de 5 horas, 7 para la de 7 días**:

- **Relleno = % de uso** consumido en la ventana: los puntos actúan como un indicador de 5 o 7 segmentos de cuánto del límite has gastado. El punto en progreso se rellena por cuartos en pasos `◔ ◑ ◕` (cada parte = ¼ de un punto, que se muestra una vez consumido) para una precisión a nivel de sub-punto: una verdadera ventaja en la ventana de 7 días, donde cada punto representa ~14%; los no consumidos son `○`.
- **Color = ritmo de consumo** — tu uso en comparación con cuánto has avanzado en la ventana:
  - **azul** — usado ≤ transcurrido: en o por debajo del ritmo, sobra bastante
  - **amarillo** — usado ≤ 1.5× transcurrido: ajustado
  - **rojo** — usado > 1.5× transcurrido: se consume demasiado rápido para aguantar la ventana
  - **umbral** — por debajo del % de uso de `CC_SL_PACE_FLOOR` (predeterminado 5), el color se mantiene azul sin importar el ritmo: una ventana casi vacía no puede agotarse, y esto evita que una ventana recién reiniciada (transcurrido ≈ 0) marque falsamente en rojo
- `↻2h15m` es el tiempo restante hasta que se reinicie esa ventana.

Así, `5h ●●◑○○ 50%` en rojo significa que has gastado la mitad del límite de 5 horas, pero aún no has llegado a la mitad de la ventana: te estás consumiendo demasiado rápido. `5h ●○○○○ 20%` en azul: 20% gastado, en el ritmo esperado.

## Instalación

```sh
claude plugin marketplace add AndrewP-GH/cc-statusline
claude plugin install cc-statusline@cc-statusline
```

Los plugins no pueden configurar la línea de estado principal automáticamente, así que habilítala una vez:

```
/cc-statusline:setup
```

Esto escribe una entrada `statusLine` en tu archivo `~/.claude/settings.json` (haciendo una copia de seguridad del archivo primero). La línea aparecerá en el siguiente renderizado del prompt. Para eliminarla, ejecuta `/cc-statusline:setup` y pide desactivarla.

### Alternativa manual

Si prefieres no usar la habilidad, agrega esto a `~/.claude/settings.json` con la **ruta absoluta** del script. El comando `statusLine` no tiene el `bin/` del plugin en el `PATH`, por lo que un `cc-statusline` sin ruta renderizaría una línea vacía. Resuelve la ruta primero:

```sh
ls ~/.claude/plugins/marketplaces/*/plugins/cc-statusline/bin/cc-statusline
```

luego:

```json
{
  "statusLine": { "type": "command", "command": "bash /ABSOLUTE/PATH/TO/cc-statusline", "padding": 0 }
}
```

## Requisitos

- Un shell POSIX, `awk` y `date`: presentes en macOS y Linux de forma nativa.
- `git` y `kubectl` son **opcionales**; sus segmentos solo aparecen cuando la herramienta está instalada y es relevante.

## Personalización

Sobrescribe los glifos de los puntos y el umbral del color de ritmo mediante variables de entorno:

| Var | Predeterminado | Significado |
|-----|---------|---------|
| `CC_SL_FULL`  | `●` | segmento consumido |
| `CC_SL_Q1`    | `◔` | punto en progreso, ~¼ |
| `CC_SL_HALF`  | `◑` | punto en progreso, ~½ |
| `CC_SL_Q3`    | `◕` | punto en progreso, ~¾ |
| `CC_SL_EMPTY` | `○` | segmento no consumido |
| `CC_SL_PACE_FLOOR` | `5` | % de uso por debajo del cual el color de ritmo de la ventana se mantiene azul |

## Cómo funciona

Claude Code envía un objeto JSON por `stdin` que incluye `model`, `workspace`, `context_window.used_percentage`, `cost`, y `rate_limits.{five_hour,seven_day}` con `used_percentage` y una marca de tiempo Unix en `resets_at`. `bin/cc-statusline` lo analiza en una sola pasada de `awk` y renderiza ambas líneas con aritmética entera: sin punto flotante ni bibliotecas de análisis de fechas.

## Licencia

MIT — ver [LICENSE](LICENSE).
