# 🌦️ P1: Que tempo fixo? (2021–2025)
> **Sistemas de Big Data (UD1: O primeiro pipeline)**  
> **Estación meteorolóxica:** A Coruña / Alvedro (Aeroporto) · **Indicativo oficial OMM / AEMET:** `08002`

Repositorio oficial e memoria técnica reproducible para o desenvolvemento dun pipeline completo de Enxeñaría de Datos: adquisición automatizada dende código, auditoría forense en bruto, diagnóstico visual, limpeza e imputación de series temporais con Pandas, análise exploratoria climática, visualización profesional, avaliación de rendemento computacional (%timeit) e persistencia relacional en PostgreSQL orquestrado mediante **Docker Compose**.

---

## 📋 1. Ficha Técnica e Fonte dos Datos
* **Estación:** A Coruña / Alvedro (Aeroporto) — **Indicativo:** `08002`
* **Coordenadas xeográficas:** Latitude 43,3000° N | Lonxitude -8,3833° W | Altitude: 97 m
* **Período temporal:** 1 de xaneiro de 2021 ao 31 de decembro de 2025 (5 anos completos = 1.826 días continuos)
* **Fonte orixinal:** Meteostat (`meteostat.net`), simulando a exportación dun portal público de datos meteorolóxicos con anomalías e incidencias de calidade inducidas con fins didácticos.
* **URL de descarga:** `https://raw.githubusercontent.com/bigdatawirtz/datasets/main/datasets/meteo_alvedro_2021_2025.csv`
* **Data de descarga:** 29 de outubro de 2026
* **Condicións de uso e licenza:** Datos abertos de Meteostat / AEMET con fins estritamente educativos.

---

## 📁 2. Estrutura do Repositorio
O proxecto cumpre estritamente a arquitectura de cartafoles requirida na UD1:

```text
que-tempo-fixo/
├── datos/
│   ├── raw/                 # Ficheiro CSV orixinal en bruto (.gitkeep)
│   └── clean/               # Datasets limpos exportados en CSV (UTF-8) e Parquet (.gitkeep)
├── notebooks/
│   └── meteo.ipynb          # Caderno Jupyter co pipeline e memoria técnica executada
├── output/                  # Gráficas de alta resolución xeradas (.png)
├── .gitignore               # Exclusión de venv, lixo de sistema e ficheiros de datos pesados
├── docker-compose.yml       # Orquestración do servizo PostgreSQL con Docker Compose
├── requirements.txt         # Librarías de Python coas versións fixadas
└── README.md                # Memoria, táboa de limpeza, instrucións e declaración de IA
```

---

## 🛠️ 3. Guía Paso a Paso de Instalación e Execución

O caderno foi deseñado para executarse **de principio a fin dende un cartafol `datos/` completamente baleiro**, xa que a primeira celda descarga automaticamente o ficheiro orixinal en bruto dende GitHub.

### Paso 1: Prerrequisitos do Sistema
Na terminal de Ubuntu / WSL2:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-venv python3-pip
```
Verifica que o servizo de Docker está dispoñible:
```bash
docker --version
docker compose version
```

### Paso 2: Contorna Virtual de Python
Dentro do cartafol raíz do proxecto (`que-tempo-fixo`):
```bash
# 1. Crear a contorna virtual illada
python3 -m venv venv

# 2. Activar a contorna
source venv/bin/activate

# 3. Actualizar pip e instalar dependencias
pip install --upgrade pip
pip install -r requirements.txt

# 4. Rexistrar o kernel de Jupyter para VS Code
python -m ipykernel install --user --name=p1-meteo --display-name="Python (p1-meteo venv)"
```

### Paso 3: Arrincar a Base de Datos con Docker Compose
```bash
# Arrincar PostgreSQL 16 en segundo plano (-d)
docker compose up -d

# Comprobar que o servizo escoita no porto 5432
docker compose ps
```
> Credenciais do servizo:
> * **Host/Porto:** `localhost:5432` | **Database:** `meteo` | **Usuario:** `postgres` | **Contrasinal:** `meteo123`
> * *Para apagar o contedor ao rematar:* `docker compose down`.

### Paso 4: Execución do Notebook
1. Abre o proxecto en VS Code: `code .`
2. Abre `notebooks/meteo.ipynb`.
3. Selecciona o kernel **`Python (p1-meteo venv)`** arriba á dereita.
4. Preme en **Run All** (*Executar todo*). O caderno executarase de principio a fin sen erros.

---

## 🧹 4. Táboa de Decisións de Limpeza (Problemas de Calidade)

En cumprimento do apartado 4.8 da práctica, recóllense as decisións adoptadas para resolver as **13 incidencias de calidade** identificadas no conxunto de datos:

| Nº | Variable / Aspecto | Problema detectado | Como se detectou? | Decisión e Tratamento aplicado |
| :---: | :--- | :--- | :--- | :--- |
| **1** | **Metadatos iniciais** | As 6 primeiras liñas conteñen texto descritivo da estación, non datos tabulares. | Inspección de liñas iniciais do ficheiro raw. | Parámetro `skiprows=6` en `pd.read_csv()` para comezar na liña da cabeceira. |
| **2** | **Codificación** | O ficheiro está codificado en estándar hispano (`latin-1` / `ISO-8859-1`) por mor de caracteres como `º` ou `ñ`. | Inspección de bytes iniciais. | Parámetro `encoding='latin-1'` en `pd.read_csv()`. |
| **3** | **Separador de campos** | Emprégase punto e coma `;` en troques da coma estándar `,`. | Inspección do texto en bruto. | Parámetro `sep=';'` en `pd.read_csv()`. |
| **4** | **Separador decimal** | Decimais representados con coma `,` (ex: `12,2`). | Inspección das columnas numéricas. | Parámetro `decimal=','` en `pd.read_csv()`. |
| **5** | **Separador de milleiros** | A presión atmosférica contén puntos de milleiros (ex: `1.015,7 hPa`). Se non se trata, lese como texto. | `.info()` e visualización de presión. | Parámetro `thousands='.'` en `pd.read_csv()` para interpretalo directamente como float. |
| **6** | **Centinelas de nulos** | Valores ausentes codificados como `-9999` ou cadeas baleiras. | Inspección de liñas e cabeceiras. | Parámetro `na_values=[-9999, "-9999", "NA", ""]` para convertelos a `NaN`. |
| **7** | **Precipitación 'Ip'** | Código meteorolóxico `'Ip'` (*Inapreciable*, $<0.1\text{ mm}$), provocando que a columna se lea como texto. | `df['precipitacion'].dtype` devolve `object`. | Substitución de `'Ip'` por `0.0` mm e conversión mediante `pd.to_numeric()`. |
| **8** | **Datas mixtas** | Mestura de datas en formato estándar ISO (`YYYY-MM-DD`) e europeo (`DD/MM/YYYY`). | Fallo de parseo estrito nun único formato. | Parseo en cascada con `pd.to_datetime()` sobre formato ISO e europeo con `.fillna()`. |
| **9** | **Filas duplicadas** | Rexistros con datas repetidas procedentes de erros de inserción. | `.duplicated(subset=['data']).sum()`. | Eliminación de duplicados con `drop_duplicates(subset=['data'])`. |
| **10** | **Días que faltan** | Faltan 14 días na serie orixinal (1.812 filas fronte ás 1.826 días reais entre 2021 e 2025). | Comparación con `pd.date_range()`. | Reindexación completa sobre o rango diario continuo de 1.826 días (`2021-01-01` a `2025-12-31`). |
| **11** | **Outlier térmico** | Temperatura media de 32.5 °C o 13 de febreiro de 2025 (erro de sensor invernal). | Gráfica de diagnóstico previo á limpeza. | Substitución do valor por `NaN` e interpolación lineal entre os días adxacentes. |
| **12** | **Inconsistencia física** | Incoherencias puntuais onde $T_{min} > T_{max}$ ou $T_{med}$ cae fóra do rango. | Validación lóxica de desigualdades. | Intercambio de valores e axuste dentro do límite $[T_{min}, T_{max}]$. |
| **13** | **Imputación de baleiros** | Valores nulos tras reindexar calendario completo. | Reconto de `.isnull().sum()`. | `fillna(0.0)` en precipitación (se non hai dato non choveu) e `.interpolate(method='linear')` en variables continuas. |

O resultado final gárdase en **`datos/clean/meteo_alvedro_2021_2025.csv`** en codificación limpa **`UTF-8`** e tamén en formato eficiente **`Parquet`**.

---

## 📈 5. Resumo do Pipeline e Resultados

Os cálculos detallados, figuras e análises atópanse completamente documentados e executados no caderno **`notebooks/meteo.ipynb`**:

* **Análise climática (4.4):**
  * *Temperatura:* Estabilidade térmica anual arredor de 14.5 °C – 15.0 °C (2023 o máis cálido con 15.15 °C).
  * *Precipitación:* Réxime atlántico estacional. O mes máis chuvioso é **Novembro** (152.1 mm acumulados de media) e os máis secos **Xullo e Agosto** (13.4 mm). A maior frecuencia de días con choiva efectiva ($\ge 1.0\text{ mm}$, OMM) rexístrase en Xaneiro e Novembro (~13 días/mes).
  * *Extremos:* Máxima absoluta de 38.2 °C (14/07/2022) e mínima de -2.1 °C (14/01/2022).
* **Visualizacións (4.5):** Gráficas exportadas en alta resolución en `output/`: evolución temporal con media móbil a 30 días, diagrama boxplot mensual e barras de precipitación acumulada mensual.
* **Eficiencia computacional (4.6):** A operación vectorizada con Pandas/NumPy resulta **entre 700x e 2.650x máis rápida** que o bucle `iterrows()`, grazas ao procesamento en memoria contigua C con instrucións SIMD nativas e ausencia de overhead de Python.
* **Persistencia PostgreSQL (4.7):** Táboa `meteo_alvedro` cargada vía `to_sql()`. As 3 consultas analíticas en SQL reproducen exactamente os valores calculados con Pandas.

---

## 🤖 6. Declaración do Uso de Intelixencia Artificial (4.8 / 6)
En cumprimento da normativa da práctica sobre o uso responsable da IA:
* **Ferramenta empregada:** Gemini
* **Tarefas nas que se utilizou:**
  * Axuda na redacción da memoria técnica.
  * Optimización da sintaxe nas consultas SQL complexas con cláusulas CTE.
  * Deseño da grella e formato estético das visualizacións de `matplotlib` e `seaborn`.
