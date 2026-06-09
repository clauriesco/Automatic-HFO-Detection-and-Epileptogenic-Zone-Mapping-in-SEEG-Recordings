# SEEG-HFO: Detector Automático de High-Frecuency Oscillations (HFO) en estéreo-EEG

En el archivo FILTRO_HFO aparece el Jupyter notebook para la detección automática y mapeo espacial de High-Frequency Oscillations (HFOs) en registros SEEG de pacientes con epilepsia refractaria. Diseñado para ser ejecutado por personal sanitario sin conocimientos de programación con una configuración mínima.

El pipeline toma un registro SEEG (EDF) y un archivo de localización de electrodos (TSV de Brainstorm) y genera un ranking de los contactos más epileptogénicos, acompañado de un mapa cerebral 3D interactivo.

---

## Pipeline

**1. Preprocesado**
Carga el EDF con MNE y elimina los canales no SEEG (ECG, trigger, SpO2…).

**2. Detección de HFOs**
Filtro Butterworth paso banda (65–250 Hz, banda ripple) → envolvente RMS deslizante de 3 ms → umbral en 3× el baseline por canal. Los eventos deben durar entre 10 y 250 ms y contener al menos 4 ciclos oscilatorios.

**3. Rechazo de artefactos**
Descarta eventos que ocurren simultáneamente en más de 6 canales (artefacto global) o con amplitud superior a 2000 µV.

**4. Normalización por región cerebral**
La tasa de HFOs de cada canal se divide entre la referencia fisiológica (p95) de su región cerebral, extraída de Parasuram et al. (2026) usando el atlas Neuromorphometrics (con fallback a Desikan-Killiany para contactos en materia blanca). Los canales con ratio > 5.8 se clasifican como patológicos.

**5. Detección de HFO on spike**
Para cada HFO detectado, comprueba si en los 220 ms previos existe una onda lenta (1–30 Hz) que supere 2× el baseline lento del canal. Este patrón de acoplamiento tiene una especificidad muy alta para tejido epileptógeno.

**6. Ranking combinado**
Tres métricas en paralelo: solo ratio / solo HFO on spike / score combinado (ratio × % HFO on spike). Resultados exportados como PNG y Excel.

**7. Mapa cerebral 3D**
Los 25 contactos con mayor ratio se proyectan sobre un cerebro plantilla MNI152 (o sobre la RM del propio paciente convertida desde DICOM) y se exportan como un archivo HTML interactivo autocontenido.

---

## Archivos de entrada

| Archivo | Formato | Origen |
|---|---|---|
| Registro SEEG | `.edf` | Sistema de adquisición clínica |
| Localización de electrodos | `.tsv` | Exportación desde Brainstorm |
| RM del paciente (opcional) | Carpeta DICOM | PACS hospitalario |

---

## Uso

El clínico solo necesita rellenar 4 variables al inicio del notebook:

```python
PACIENTE  = "Paciente_1"
EDF_PATH  = r"C:\...\SEEG.edf"
TSV_PATH  = r"C:\...\localizacion.tsv"
DICOM_DIR = r"C:\...\DICOM"   # opcional
```

Todas las rutas de salida se generan automáticamente. La carpeta de resultados se crea dentro de la carpeta del EDF.

---
## Referencia

Este proyecto implementa la Tabla 2 descrita en:

> Parasuram H, Pillai A, Gopinath S, Rajeshkannan R, Ravindran S, Raman A, Anandakuttan A.
> *Mapping physiological high-frequency oscillation rates of the cerebral cortex for improved epileptogenic zone delineation in stereoelectroencephalography.*
> J Neurosurg. Published online February 27, 2026.
> DOI: [10.3171/2025.9.JNS25602](https://thejns.org/doi/abs/10.3171/2025.9.JNS25602)

```

Ejecuta la primera celda del notebook para instalar todo en el kernel activo.
