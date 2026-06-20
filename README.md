# HidroSed Maestra Integrada v6.3 — Auditoría de Informe

Aplicación maestra Streamlit para análisis hidrológico, hidráulico, sedimentológico y de socavación en cauces naturales, integrada con DEM COP30, curvas de nivel, secciones v13, HidroSed Cauces, granulometría por defecto/usuario, IDF y revisión de informes externos.

## Main file path

```text
app.py
```

## Novedades v6.3

Esta versión incorpora las observaciones detectadas al revisar el informe **"Extracción Mecanizada de áridos Estero Punitaqui"**:

1. **Corrección de preset DGA-AC Zona Jp Limarí**
   - Se agrega preset `Zona Jp pluvial Limarí - envolvente máxima informe Punitaqui (editable)`.
   - Usa `alpha_inst = 2.14`.
   - Usa curva `Máx.` de frecuencia regional: T2=0.30, T5=0.66, T10=1.00, T20=1.61, T25=1.85, T50=2.76, T75=3.42, T100=3.94.
   - Conserva preset `Zona Jp pluvial Limarí - media regional (editable)` para contraste.

2. **Auditoría de informe**
   - Nuevo módulo de menú: `4 · Auditoría de informe`.
   - Compara datos declarados en informe vs cálculo interno.
   - Genera matriz de diferencias y Excel de auditoría.

3. **Control de P24 inconsistente**
   - Permite registrar P24 textual y P24 usado en cálculo.
   - Advierte si el texto declara 120 mm pero el cálculo usa 80,7 mm.

4. **Control de comuna / trazabilidad territorial**
   - Advierte si la portada/objetivo indica Punitaqui y otra sección indica Salamanca.

5. **Sedimentos: evento vs anual**
   - Advierte cuando se denomina “anual” a un cálculo basado en evento de 48 h.

6. **Geometría digital obligatoria para verificación hidráulica completa**
   - Si las secciones solo están en PDF/imagen, la app bloquea dictamen definitivo.
   - Requiere KMZ/DXF/Excel de secciones o `04_Puntos_Seccion`.

## Flujo recomendado

```text
1. DEM y curvas
2. Hidrología KMZ
3. HidroSed Maestra Integrada
4. Auditoría de informe
5. Guía técnica
```

## Validaciones ejecutadas

```bash
python -m compileall .
python smoke_test_integracion_v13.py
python smoke_test_3d.py
python smoke_test_modelacion_v6.py
python smoke_test_v6_1_inputs_idf_grain.py
python smoke_test_v6_2_default_grain.py
python smoke_test_v6_3_auditoria.py
```

## Configuración Streamlit Cloud

En `Secrets`:

```toml
OPENTOPO_API_KEY = "TU_API_KEY_DE_OPENTOPOGRAPHY"
```


## Novedad v6.4
- Se agregó una **Galería de resultados** con lista desplegable para visualizar imágenes y gráficos en una ventana dedicada.
- Se incorporaron tres vistas de referencia: **Resultados de socavación**, **Transporte de sedimentos** y **Modelo 3D del cauce**.

## Novedad v6.5 - corrección crítica
- Se corrigió el error `ValueError: The truth value of a DataFrame is ambiguous` en `status_badge()`.
- Se robusteció `contour_generator.py` para generar curvas de nivel desde DEM en Streamlit Cloud.
- Se agregó filtrado de bordes NoData para evitar curvas falsas.
- Se agregó fallback con Matplotlib si falla `skimage.measure.find_contours`.
- La vista previa ahora muestra DEM + curvas superpuestas.


## Novedad v6.6 - Delimitación de cuenca corregida
- Se agregó el módulo `modules_hidrosed/watershed_delineation.py`.
- La app ahora delimita cuenca desde DEM + punto de control mediante:
  - relleno de depresiones Priority-Flood;
  - dirección de flujo D8;
  - acumulación de flujo;
  - ajuste automático del punto de salida al cauce de mayor acumulación;
  - vectorización del polígono de cuenca;
  - exportación KML/KMZ de cuenca delimitada.
- La pestaña `1 · DEM COP30 → Cuenca → Curvas → KMZ combinado` ahora incluye:
  - A. Descargar DEM;
  - B. Delimitar cuenca;
  - C. Generar curvas;
  - D. Armar KMZ combinado.
- El KMZ combinado puede usar la cuenca automática o un KMZ/KML validado manualmente.


## Novedad v6.7 - Integración total
- La delimitación automática de cuenca queda integrada como etapa nativa del flujo maestro.
- La app crea automáticamente el `KMZ combinado` cuando ya existen:
  - cuenca delimitada automáticamente;
  - curvas de nivel generadas desde DEM.
- El módulo hidrológico carga automáticamente el KMZ combinado desde `st.session_state["hidro_workspace_kmz_path"]`.
- Si todavía no existen curvas, el módulo hidrológico puede usar directamente la cuenca delimitada desde `st.session_state["basin_auto_kmz_path"]`.
- No se requiere descargar y volver a subir la cuenca ni las curvas entre módulos.


## Novedad v6.8 - Aplicación integral con API Key desde interfaz
- La aplicación queda autosuficiente.
- No es obligatorio configurar `Secrets` en Streamlit Cloud.
- La API Key de OpenTopography puede:
  - pegarse directamente en la barra lateral;
  - cargarse como archivo `.txt`;
  - cargarse como archivo `.toml`;
  - o leerse desde `st.secrets` si el usuario decide usarlo.
- La clave queda solo en `st.session_state` durante la sesión y no se escribe en el repositorio.
- Flujo integral:
  `Punto KMZ → API Key en interfaz → DEM → cuenca → curvas → KMZ combinado → hidrología → HidroSed`.


## Novedad v6.9 - Módulo OpenTopo original integrado sin reescritura
- Se integró la aplicación `opentopo_dem_streamlit` adjunta por el usuario como módulo nativo.
- Se conserva su lógica de ejecución: carga KMZ/KML, API Key manual, cálculo bbox, URL OpenTopography, descarga GeoTIFF.
- Se agregó únicamente el bloque de **curvas de nivel desde el DEM descargado**, con equidistancia mínima de **1 metro**.
- El módulo se ejecuta desde:
  `modules_external/opentopo_dem_streamlit_original/app.py`
- También queda disponible como página Streamlit:
  `pages/01_DEM_OpenTopo_original_curvas_1m.py`
- Main file path de la aplicación maestra: `app.py`.


## Novedad v6.10 - Curvas de nivel con distancia mínima explícita de 1 m
- El módulo OpenTopo original integrado ahora muestra explícitamente:
  `Distancia entre curvas de nivel [m]`.
- La lista de equidistancias parte en:
  `1 m`.
- No se permite ingresar valores menores a 1 m.
- Opciones disponibles:
  `1, 2, 5, 10, 20, 25, 50, 100, 200 m`.
- La metadata de curvas registra:
  `distancia_minima_permitida_m = 1.0`.
