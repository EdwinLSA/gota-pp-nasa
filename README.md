# gota-pp-nasa

Pipeline de ingeniería de datos para la extracción y preprocesamiento de datos meteorológicos y solares de la **NASA POWER API**. Este repositorio convierte los datos crudos de la NASA en datasets limpios, estructurados y listos para análisis, alimentando los modelos de aprendizaje automático y de riego del proyecto **GOTA**.

A partir de las variables climáticas por municipio, el pipeline construye el ciclo fenológico del cultivo de **trigo**, calcula la evapotranspiración (ET0 / ETc) y estima la humedad del suelo para generar recomendaciones de riego.

---

## 📦 Datos de entrada

- **Fuente:** [NASA POWER API](https://power.larc.nasa.gov/) (Prediction Of Worldwide Energy Resources).
- **Cobertura:** 18 municipios del estado de Sonora, México.
- **Periodo:** 2023 – 2026.
- **Formato:** un archivo `.csv` por municipio. El nombre del municipio se extrae automáticamente del nombre del archivo.

### Variables NASA POWER utilizadas

| Variable            | Descripción                                            | Unidad    |
|---------------------|--------------------------------------------------------|-----------|
| `YEAR`, `DOY`       | Año y día del año (day of year)                        | —         |
| `MO`, `DY`, `HR`    | Mes, día y hora (en el dataset horario)                | —         |
| `ALLSKY_SFC_SW_DWN` | Radiación solar incidente en superficie                | MJ/m²/día |
| `T2M`               | Temperatura del aire a 2 m                             | °C        |
| `QV2M`              | Humedad específica a 2 m                              | g/kg      |
| `RH2M`              | Humedad relativa a 2 m                                | %         |
| `PRECTOTCORR`       | Precipitación corregida                                | mm        |
| `WS2M`              | Velocidad del viento a 2 m                            | m/s       |
| `PS`                | Presión superficial                                    | kPa       |
| `GWETTOP`/`GWETROOT`/`GWETPROF` | Humedad del suelo (superficie / raíz / perfil) | fracción  |

---

## 🔄 Flujo del pipeline

El procesamiento se realiza en el notebook [`dataset_pp_6to_start_nov.ipynb`](dataset_pp_6to_start_nov.ipynb):

1. **Detección y concatenación.** Se recorren todos los `.csv` de la carpeta de datos climatológicos, se añade la columna `Municipio` (extraída del nombre del archivo) y se unen en un solo DataFrame.
2. **Construcción de fechas.** Se convierten `YEAR` y `DOY` a tipo numérico y se genera la columna `fecha` (y opcionalmente `MO`, `DY`, `fecha_hora`).
3. **Limpieza.** Se normalizan tipos de datos y se reemplazan los valores faltantes de NASA (`-999`) por `NaN`.
4. **Ciclo fenológico del trigo.** Se asigna a cada día un `dia_global`, un `dia_ciclo` (ciclo recurrente de 150 días), su **etapa fenológica** y el coeficiente de cultivo **Kc** correspondiente.
5. **Evapotranspiración.** Se agregan los datos a nivel diario y se calcula:
   - **ET0** mediante la ecuación de **Penman-Monteith (FAO-56)**.
   - **ETc** = ET0 × Kc.
6. **Balance hídrico.** Un modelo prototipo estima la `humedad_suelo_estimada` día a día por municipio a partir de la lluvia infiltrada, la demanda del cultivo (ETc) y el drenaje.
7. **Recomendación de riego.** A partir de la humedad estimada se clasifica el `nivel_humedad` (Baja / Media / Alta), se calcula el `agua_recomendada_mm` y el indicador binario `regar_hoy`.

### Etapas fenológicas y Kc del trigo

| Día del ciclo | Etapa                              | Kc   |
|---------------|------------------------------------|------|
| 1 – 15        | Germinación / Emergencia           | 0.30 |
| 16 – 55       | Amacollamiento                     | 0.55 |
| 56 – 75       | Encañe                             | 0.85 |
| 76 – 80       | Embuche / Espigamiento             | 1.10 |
| 81 – 85       | Floración                          | 1.15 |
| 86 – 110      | Grano acuoso-lechoso               | 1.10 |
| 111 – 130     | Grano masoso                       | 0.85 |
| 131 – 150     | Madurez fisiológica / Cosecha      | 0.35 |

---

## 📤 Datos de salida

| Archivo                                              | Descripción                                                                 |
|------------------------------------------------------|-----------------------------------------------------------------------------|
| `dataset con variable objetivo_final2.csv`           | Dataset preprocesado con la variable objetivo.                              |
| `dataset_trigo_municipios_con_ciclos_final (1).xlsx` | Dataset final con ciclos fenológicos, ET0/ETc, humedad estimada y riego.   |

### Columnas principales del dataset final

`YEAR`, `MO`, `DY`, `HR`, variables climáticas NASA, `Municipio`, `fecha`, `fecha_hora`,
`dia_global`, `dia_ciclo`, `etapa_fenologica`, `kc`, `et0`, `etc`,
`humedad_suelo_estimada`, `nivel_humedad`, `agua_recomendada_mm`, `regar_hoy`.

---

## 🛠️ Requisitos

```bash
pip install pandas numpy openpyxl jupyter
```

- Python 3.9+
- pandas
- numpy
- openpyxl (para exportar a `.xlsx`)

---

## ▶️ Uso

1. Coloca los `.csv` descargados de NASA POWER (uno por municipio) en la carpeta de datos climatológicos.
2. Ajusta la variable `ruta` en el notebook para que apunte a esa carpeta.
3. Ejecuta las celdas de [`dataset_pp_6to_start_nov.ipynb`](dataset_pp_6to_start_nov.ipynb) en orden.
4. Los datasets procesados se generarán en la ruta de salida configurada.

---

## 📁 Estructura del repositorio

```
gota-pp-nasa/
├── dataset_pp_6to_start_nov.ipynb                  # Notebook del pipeline de preprocesamiento
├── dataset con variable objetivo_final2.csv        # Dataset con variable objetivo
├── dataset_trigo_municipios_con_ciclos_final (1).xlsx  # Dataset final con ciclos y riego
└── README.md
```

---

> Parte del proyecto **GOTA** — sistema de apoyo a la decisión de riego para el cultivo de trigo.
