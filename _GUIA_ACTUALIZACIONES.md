# GUÍA DE ACTUALIZACIONES — Reportes DIFARE

Referencia para no repetir errores conocidos al actualizar los reportes.

---

## 📅 SEMANAL DIFARE

### Constantes que SE ACTUALIZAN (⚠️ TODAS ESTAS 8)

En **`biogenet_farmacias_control.html`** (versión gerente, 8+ semanas):

| Constante | Qué contiene | De dónde sale |
|---|---|---|
| `WEEKLY_LABELS` | Array de labels de semanas | Agregar nueva al final |
| `WEEKLY_HISTORY` | Dict `{label: {data, stock, stock_prod}}` por semana | Agregar bloque nuevo |
| `WEEKLY_LABEL` | Semana por defecto que se muestra | Poner la nueva |
| `DATA_WEEKLY` | ⚠️ Lista de ventas de la semana actual (alimenta columna "ACTUAL AL XX") | Nueva semana |
| `STOCK_MAP` | `{cod: total_stock_pdv}` semana actual | Nueva semana |
| `STOCK_WEEKLY` | Igual que STOCK_MAP pero para uso interno | Nueva semana |
| `STOCK_PROD_MAP` | `{cod: {producto: stock}}` semana actual | Nueva semana |

En **`biogenet_farmacias.html`** (versión rep, 1 semana):

| Constante | Notas |
|---|---|
| `WEEKLY_LABEL` | Nueva semana |
| `DATA_WEEKLY` | ⚠️ Lista de ventas semana actual (columna "ACTUAL AL XX") |
| `STOCK_MAP` | Nueva semana |
| `STOCK_WEEKLY` | Nueva semana |
| `STOCK_PROD_MAP` | Nueva semana |

### Constantes que NO SE TOCAN (mensual histórico)

⚠️ NO reemplazar en semanal:
- `DATA_REP` — histórico mensual con campo `mes`
- `DATA_NOREP` — histórico mensual sin rep
- `GPF_DATA_REP` — histórico mensual GPF (Fybeca)
- `GPF_DATA_NOREP` — histórico mensual GPF sin rep
- `GPF_STOCK_MAP` — stock GPF mensual
- `GPF_STOCK_PROD` — stock GPF mensual por producto
- `GPF_COD_MAP` — mapeo códigos GPF

### Archivos externos

- `weekly_history.json` — agregar nueva semana al final del dict
- `farm_mappings.json` — mapeo `cod → rep` y `cod → pdv`

### Columna del XLSX a usar

⚠️ **`CODIGOPDV`** (NO `IDDIFARE`). El IDDIFARE tiene otro formato (000000...) que no matchea nuestros mappings.

### Estructura del bloque semanal

```json
{
  "data": [
    {"cod":"BPG06","pdv":"...","prod":"...","u":1.0,"v":13.29,"rep":"RICARDO SAN ANDRES"},
    ...
  ],
  "stock": {"cod": total_stock},
  "stock_prod": {"cod": {"producto": stock_producto}}
}
```

### Filtros para el rep DATA_REP / DATA_NOREP (solo semanal en rep file)

⚠️ En el rep file NO se sobreescriben `DATA_REP/DATA_NOREP` — esos son mensuales.

Los BAD_REPS que van a NOREP: `NO EXISTE`, `ROJO VIVO`, `CERRADA`, `NO SE VISITA`.

---

## 📆 MENSUAL DIFARE

### Constantes que SE ACTUALIZAN

En **ambos archivos** (`biogenet_farmacias.html` y `biogenet_farmacias_control.html`):

| Constante | Qué contiene | Acción |
|---|---|---|
| `DATA_REP` | Ventas históricas mensuales con `mes: "YYYYMM"` (reps asignados) | Agregar nuevo mes, eliminar el más antiguo (mantener ~7 meses) |
| `DATA_NOREP` | Igual pero sin rep asignado | Agregar/eliminar igual |

### Constantes que NO SE TOCAN en mensual

- `WEEKLY_HISTORY`, `WEEKLY_LABELS`, `WEEKLY_LABEL` — semanal
- `STOCK_MAP`, `STOCK_WEEKLY`, `STOCK_PROD_MAP` — semanal
- Todo lo de GPF (Fybeca) — es otro flujo mensual aparte

### Formato del mes

`mes: "YYYYMM"` sin guiones. Ejemplo: `"202605"` para mayo 2026.

---

## 📊 MENSUAL GPF (FYBECA)

### Constantes que SE ACTUALIZAN

Similar al mensual DIFARE pero con prefijo GPF:

| Constante | Notas |
|---|---|
| `GPF_DATA_REP` | Ventas mensuales GPF con rep |
| `GPF_DATA_NOREP` | Ventas mensuales GPF sin rep |
| `GPF_STOCK_MAP` | Stock total por cod GPF |
| `GPF_STOCK_PROD` | Stock por cod x producto |
| `GPF_COD_MAP` | `{cod: {pdv, rep}}` para búsqueda |

Los códigos GPF empiezan con `GPF` (ej. `GPF127`, `GPF72469`).

---

## 🚨 ERRORES CONOCIDOS Y SOLUCIÓN

### 1. Al actualizar semanal se pierde DATA_REP histórico
- **Causa:** El script sobreescribió `DATA_REP` con datos semanales sin campo `mes`
- **Síntoma:** Aparece columna "indefinido" en el reporte
- **Solución:** Recuperar `DATA_REP` y `DATA_NOREP` del archivo `biogenet_farmacias_control.html` (versión gerente) que sirve de backup

### 2. Stock incorrecto tras actualizar semanal
- **Causa:** Solo se actualizó `STOCK_MAP` pero no `STOCK_WEEKLY` ni `STOCK_PROD_MAP`
- **Síntoma:** Un PDV muestra producto X con stock que no coincide con la realidad (muestra la semana anterior)
- **Solución:** Actualizar SIEMPRE las 3 constantes de stock juntas

### 2b. Columna "ACTUAL AL XX" no muestra las ventas
- **Causa:** No se actualizó `DATA_WEEKLY`
- **Síntoma:** El PDV muestra ventas históricas mensuales pero la columna "ACTUAL AL 28 JUN 26" está vacía o muestra datos de la semana anterior
- **Solución:** Actualizar `DATA_WEEKLY` con la lista completa de ventas de la semana nueva (mismo formato que `WEEKLY_HISTORY[label].data`)

### 3. Confusión Digeros Forte vs Digeros Gotas
- **NOTA:** Son productos distintos:
  - `Digeros Forte Tabx120000uix1` (tabletas)
  - `Digeros Gotas Frasco X3ml` (frasco líquido)
- Al leer el reporte, verificar el nombre completo del producto

### 4. Cap de 1000 filas en Supabase
- Aplicar paginación al pull con `order=fecha.desc&limit=1000&offset=N`
- Esto no aplica al reporte DIFARE (es un HTML estático)

---

## 📝 Checklist rápido antes de subir a GitHub

### Semanal DIFARE

- [ ] `weekly_history.json` tiene el nuevo bloque
- [ ] `biogenet_farmacias_control.html`: WEEKLY_LABELS, WEEKLY_HISTORY, WEEKLY_LABEL, **DATA_WEEKLY**, STOCK_MAP, STOCK_WEEKLY, STOCK_PROD_MAP actualizados
- [ ] `biogenet_farmacias.html`: WEEKLY_LABEL, **DATA_WEEKLY**, STOCK_MAP, STOCK_WEEKLY, STOCK_PROD_MAP actualizados
- [ ] `DATA_REP`/`DATA_NOREP` **intactos** en ambos archivos (verificar cuenta de `"mes":` > 15000)
- [ ] Prueba: elegir un PDV con venta conocida (ej. FG136) y verificar que la columna "ACTUAL AL XX" muestra la venta correcta
- [ ] Tags `</html>`, `</script>`, `</body>` presentes al final
- [ ] Tamaño de archivos coherente (rep ~8-9 MB, control ~17 MB)

### Mensual DIFARE

- [ ] Nuevo mes agregado en `DATA_REP` y `DATA_NOREP` en ambos archivos
- [ ] Mes más antiguo eliminado si excede 7 meses de historia
- [ ] Formato mes: `"YYYYMM"`
- [ ] `WEEKLY_*` y `STOCK_*` **intactos**
- [ ] `GPF_*` **intactos**

---

## 🗂️ Archivos afectados

- `C:\Users\Wilbert Chica\Documents\PROYECTOS\SISTEMA DE REGISTRO\biogenet_farmacias.html` (rep, 1 semana + histórico mensual)
- `C:\Users\Wilbert Chica\Documents\PROYECTOS\SISTEMA DE REGISTRO\biogenet_farmacias_control.html` (gerente, 8+ semanas + histórico mensual)
- `outputs/weekly_history.json` (historial de semanas parseado, backup)
- `outputs/farm_mappings.json` (cod → rep, cod → pdv)

---

## 🔄 SINCRONIZAR ASIGNACIONES DE REPS (FARM_PANEL → reportes)

Cuando se actualiza el `FARM_PANEL` en `biogenet_visitas_24.html` (agregar/quitar farmacias, cambiar rep, agregar nuevo rep como Karina Zamora o Maria Pia Ortiz), los reportes de farmacias quedan desactualizados.

### Cuándo hacerlo

- Agregas o quitas farmacias del `FARM_PANEL`
- Cambias la asignación de una farmacia de un rep a otro
- Entra un rep nuevo con su panel
- Sale un rep (dejar de asignarle farmacias)

### Cómo hacerlo (procedimiento)

1. **Extraer** el `FARM_PANEL` actualizado del HTML del visitador
2. **Construir** dos índices:
   - `cod_rep`: `{codigo: rep}` - primera ocurrencia gana (aplica dedup)
   - `name_rep`: `{pdv_normalizado: rep}` - para matching GPF
3. **Guardar** `outputs/farm_mappings.json` con el nuevo `cod_rep`
4. **En cada archivo** (`biogenet_farmacias.html` + `biogenet_farmacias_control.html`):
   - Combinar `DATA_REP` + `DATA_NOREP`, reasignar rep con nuevo mapping, separar según si tiene rep válido
   - Reasignar rep en `DATA_WEEKLY`
   - Reasignar rep en cada semana de `WEEKLY_HISTORY[label].data` (solo CONTROL)
   - Actualizar rep en `GPF_COD_MAP` matching por nombre PDV
   - Reasignar rep en `GPF_DATA_REP` + `GPF_DATA_NOREP` matching por nombre PDV

### Reglas de matching

| Sistema | Match por | Fuente en FARM_PANEL |
|---|---|---|
| DIFARE (DATA_REP, DATA_WEEKLY, WEEKLY_HISTORY) | `cod` exacto | `f.cod` |
| GPF (GPF_COD_MAP, GPF_DATA_REP) | Nombre PDV normalizado (UPPER + trim) | `f.pdv` |

### Fallback

Si un cod/nombre no matchea con ningún rep en FARM_PANEL:
- Se marca `rep = 'NO EXISTE'` (para DIFARE)
- Se marca `rep = ''` (para GPF)
- La fila se mueve al array NOREP correspondiente

### Verificación post-sincronización

- [ ] Total de reps en DATA_REP incluye a todos los reps activos del FARM_PANEL
- [ ] Los reps que salieron ya no aparecen en la lista
- [ ] Los reps que entraron tienen conteos coherentes
- [ ] No hay reps huérfanos (nombres viejos que ya no existen)

---

Última actualización de esta guía: 2026-06-28
