# Reto ETL: dos fuentes, una decisión

**Pregunta:** ¿qué 10 libros conviene comprar primero con presupuesto limitado?
- Fuente A (HTML): https://books.toscrape.com/ (listado paginado + fichas)
- Fuente B (JSON): https://openlibrary.org/search.json (clave de unión: título normalizado)

## Contenido
| Archivo / carpeta | Descripción |
|---|---|
| `reto_etl.ipynb` | Notebook con extracción, transformación, carga, SQL, pruebas y conclusión |
| `datos/base/` | Copias locales originales (HTML y JSON) que genera la primera ejecución |
| `datos/esc_cambia_decision/` | Copia con el precio de un libro editado en el HTML (prueba 1) |
| `datos/esc_duplicado/` | Copia con un registro duplicado en el HTML (prueba 2) |
| `docker-compose.yml` | PostgreSQL 16 en Docker (usuario/clave de desarrollo `reto`/`reto`) |
| `requirements.txt` | Dependencias de Python |

## 1. Instalar dependencias
```bash
python -m venv .venv
.venv\Scripts\activate          # Windows   (Linux/macOS: source .venv/bin/activate)
pip install -r requirements.txt
```

## 2. Levantar PostgreSQL con Docker Desktop
```bash
docker compose up -d
```
Conexión por defecto del notebook: `localhost:5432`, usuario `reto`, clave `reto`, base `retoetl`.
Para otra configuración defina variables de entorno antes de abrir Jupyter:
`PGHOST`, `PGPORT`, `PGUSER`, `PGPASSWORD`, `PGDATABASE` (o `DATABASE_URL=postgresql+psycopg2://usuario:clave@host:puerto/base`).
No incluya credenciales personales en la entrega.

## 3. Ejecutar
```bash
jupyter lab reto_etl.ipynb       # Kernel > Restart & Run All
```
- **Primera vez (con internet):** el notebook descarga las páginas y el JSON y guarda las copias en `datos/base/` (unos minutos por la pausa de cortesía de 1 s entre peticiones).
- **Reproducción sin internet:** si `datos/base/` ya existe, el notebook lee solo las copias locales. Los escenarios `esc_cambia_decision` y `esc_duplicado` se recrean siempre desde `datos/base/` y se ejecutan únicamente con copias locales.

## Los tres escenarios
1. **base**: fuentes originales.
2. **esc_cambia_decision**: se sube el precio del libro de prioridad 1 en el HTML del listado; sale de la lista y entra el libro K+1.
3. **esc_duplicado**: se duplica el bloque HTML del libro de prioridad 3; el proceso detecta 1 duplicado y la decisión no cambia.

Los tres se cargan en la tabla `decision_libros` (columna `escenario`) y se comparan con la misma consulta SQL.

## Parámetros
En la celda de configuración: `N_PAGINAS`, `PARAMS = {min_estrellas, min_ediciones, top_k}`. Si en su ejecución hay menos de `top_k + 1` candidatos, baje `min_ediciones` o `min_estrellas`.

## Antes de entregar
Ejecute todo el notebook (para que quede con salidas), revise y complete la conclusión con sus propias palabras, y comprima la carpeta (con `datos/`) en un solo ZIP.
