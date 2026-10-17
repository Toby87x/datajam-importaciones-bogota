# Dependencia y concentración de las importaciones de Bogotá

**Bogotá DataJam 2026 · Equipo ETL Importaciones Bogotá** (Fundación Universitaria Compensar)
Integrantes: Samuel Alejandro Rodríguez Vargas (líder), Diego Mauricio Molano Castillo, Samuel Andrés Barrios Alfonso.

## Problema abordado
Bogotá depende de las importaciones para abastecer su industria y su consumo, pero esas compras están concentradas en pocos países y en bienes de tecnología media y alta. Además, la información pública se publica sin estandarizar y solo en dólares, lo que dificulta medir el riesgo de dependencia y el efecto del tipo de cambio sobre el costo real para las empresas de la ciudad.

**Pregunta:** ¿Qué tan concentradas están las importaciones con destino a Bogotá por país, región del mundo y nivel tecnológico, y cómo cambió su valor entre el primer bimestre de 2025 y el de 2026 en dólares y en pesos?

**Hipótesis:** la dependencia se concentra en China y Estados Unidos y se desplaza hacia Asia; el crecimiento en dólares se compensa en pesos por la apreciación del peso.

## Fuentes de datos públicas
| # | Conjunto | Entidad | Acceso | Archivo en el repo |
|---|---|---|---|---|
| 1 | Importaciones Bogotá | Secretaría Distrital de Desarrollo Económico | https://datosabiertos.bogota.gov.co/dataset/importaciones-bogota (descargado el 20/08/2026) | `data/raw/importaciones_bogota_original.csv.gz` (copia sin modificar, hoja original) |
| 2 | Tasa de Cambio Representativa del Mercado (TRM) | Superintendencia Financiera de Colombia | API https://www.datos.gov.co/resource/32sa-8pi3.json | `data/external/trm_mensual_2025_2026.csv` |
| 3 | ISO 3166 con regiones ONU M49 | ONU / ISO | https://github.com/lukes/ISO-3166-Countries-with-Regional-Codes | `data/external/iso3166_regiones_onu_m49.csv` |
| 3b | Nombres de países en español (tabla puente) | ISO 3166 – repositorio *world_countries* | https://github.com/stefangabos/world_countries | `data/external/paises_nombres_es_iso.csv` |
| — | Equivalencias manuales de países (26 casos) | Elaboración propia | — | `data/external/equivalencias_paises_manual.csv` |

El notebook descarga las fuentes 2 y 3 en cada ejecución y usa las copias locales si no hay internet.

## Metodología
1. **Extracción:** lectura del archivo original y descarga de la TRM y de las tablas ISO/ONU.
2. **Limpieza y transformación:** réplica en Python del ETL hecho en Power Query y OpenRefine durante el curso (ver `docs/Informe_proyecto_ETL_primer_corte.pdf`):
   - códigos de país, partida, CIIU y capítulo como texto, con ceros a la izquierda;
   - eliminación de 9 duplicados exactos;
   - 65 valores FOB y 4 pesos en cero convertidos a nulo;
   - unificación de 7 códigos CIIU con descripciones inconsistentes;
   - «Sin información» tratado como dato faltante;
   - separación de las zonas francas de los países;
   - creación de una columna de fecha.
3. **Integración:**
   - importaciones + TRM por año y mes, para obtener el valor CIF en pesos;
   - importaciones + regiones ONU por código ISO alfa-3, obtenido a partir del nombre normalizado más la tabla de equivalencias. Cobertura: 96,5 % del valor por cruce automático.
4. **Indicadores:**
   - participación por país, región y subregión;
   - Índice Herfindahl-Hirschman (IHH) de concentración por país;
   - variación interanual en USD y en COP;
   - TRM ponderada;
   - USD por kilogramo;
   - composición por nivel tecnológico y uso económico.
5. **Visualización:**
   - tablero de Power BI (modelo estrella: 1 tabla de hechos y 6 dimensiones, 20 medidas DAX, 3 páginas);
   - dashboard HTML interactivo generado por el notebook.

## Principales resultados (Ene–Feb 2025 vs Ene–Feb 2026)
- China (29,2 %) y Estados Unidos (23,0 %) concentran el 52 % del valor CIF. El IHH por país subió de **1.505 a 1.556** (concentración moderada).
- Asia creció **+21,3 %** y superó a América como primera región de origen (43,4 % frente a 40,1 %). China creció +22,0 % y EE. UU. cayó −4,7 %.
- El valor CIF creció **+8,8 % en USD** pero cayó **−4,8 % en COP**, porque la TRM promedio bajó 12,5 % (4.213 → 3.686).
- Los bienes de consumo crecieron +27,7 % y los de capital +16,1 %, mientras que las materias primas cayeron −4,3 %.

Las tablas completas están en `outputs/tablas/` y las figuras en `outputs/figuras/`.

## Cómo ejecutar el análisis
Requisitos: Python 3.10 o superior.
```bash
git clone https://github.com/Toby87x/datajam-importaciones-bogota.git
cd datajam-importaciones-bogota
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace notebooks/01_etl_integracion_importaciones.ipynb
```
También se puede abrir con `jupyter notebook notebooks/01_etl_integracion_importaciones.ipynb` y ejecutar todas las celdas.

El notebook regenera:
- `data/processed/importaciones_integrado.csv.gz`: datos limpios e integrados, insumo de Power BI;
- `outputs/tablas/*.csv`;
- `outputs/figuras/*.png`;
- `outputs/dashboard_importaciones_bogota.html`.

## Cómo abrir los tableros (despliegue)
**Dashboard HTML:** abra `outputs/dashboard_importaciones_bogota.html` en cualquier navegador. No requiere instalación y funciona sin internet.

**Power BI Desktop** (versión 2025 o posterior):
1. Abra `dashboard/powerbi/Importaciones_Bogota.pbip`. Si hay un `.pbix` en `dashboard/`, también puede abrir ese archivo directamente.
2. Vaya a *Inicio → Transformar datos → Editar parámetros → `CarpetaRepo`* y escriba la ruta de la carpeta del repositorio terminada en `\`. Por ejemplo: `C:\Users\usuario\Desktop\datajam-importaciones-bogota\`.
3. Haga clic en *Aplicar cambios* o en *Inicio → Actualizar*. La carga lee `data/processed/importaciones_integrado.csv.gz`.

Si el `.pbip` no abre, active en *Opciones → Características en versión preliminar* las opciones «Proyecto de Power BI (.pbip)», «TMDL» y «PBIR».

## Estructura del repositorio
```
├── data/
│   ├── raw/          # Fuente 1 original (sin modificar)
│   ├── external/     # Fuentes 2 y 3 (TRM, ISO/ONU) y equivalencias manuales
│   └── processed/    # Salida integrada del notebook (insumo de Power BI)
├── notebooks/        # 01_etl_integracion_importaciones.ipynb (código documentado)
├── outputs/
│   ├── figuras/      # Gráficos PNG
│   ├── tablas/       # Indicadores en CSV
│   └── dashboard_importaciones_bogota.html
├── dashboard/powerbi/  # Proyecto Power BI (.pbip: modelo TMDL + reporte PBIR)
├── docs/             # Nota técnica, formulario, modelo relacional, informe del primer corte
├── requirements.txt
└── README.md
```

## Licencia de los datos
Los datos de la SDDE y la Superintendencia Financiera se publican como datos abiertos (Ley 1712 de 2014). Las tablas ISO/ONU son de dominio público.
