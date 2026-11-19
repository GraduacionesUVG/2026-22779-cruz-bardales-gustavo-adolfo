# Transcripción Automática de Música Polifónica a Musicografía Braille mediante Inteligencia Artificial y Algoritmos Deterministas

## Descripción

Sistema de código abierto que recibe una grabación de piano y entrega la partitura en musicografía Braille, en un archivo BRF listo para leerse en una línea Braille o imprimirse en relieve.

Una persona ciega que estudia piano necesita la partitura en Braille, y hoy la escribe a mano un especialista. Existen modelos que reconocen las notas de una grabación y traductores que convierten una partitura digital a Braille, pero ninguna herramienta une los dos pasos. Este sistema los une en una cadena de cuatro partes:

1. **Modelo acústico.** El modelo de Kong et al. (2021), ajustado con audio degradado, detecta cada nota con su inicio, fin, altura y velocidad, más el pedal.
2. **Reconstrucción de la partitura.** Limpieza de armónicos, seguimiento del pulso, reparto de manos y cuantización a figuras y compases.
3. **Traductor determinista.** Un árbol de sintaxis abstracta y una máquina de estados aplican 28 reglas del *Manual simplificado de musicografía braille* (Aller Pérez, 2001).
4. **Renderizador y exportador.** Formato compás sobre compás y archivo BRF de 40 celdas por línea.

La persona lo usa desde una aplicación web progresiva con tres modos: tutor, baja visión y lector de pantalla. La aplicación está publicada en https://g2309.github.io/amt-braille-translator/.

Resultados principales:

| Objetivo | Resultado |
|---|---|
| Elegir el modelo | Kong, con 95.9 % de F1 de inicio y 83.7 % a nivel de nota en MAESTRO |
| Ajustarlo | +5.80 puntos en audio degradado y −1.00 en limpio |
| Traducir | 28 reglas trazadas al Manual y 249 pruebas automáticas |
| Integrar | Siempre por debajo de 1.5 veces la duración del audio |
| Medir la fidelidad | Exactitud de Símbolo Braille de 33.2 a 46.5 % desde audio de estudio, y 78.8 % con la partitura correcta |
| Acceso autónomo | 226 de 226 elementos etiquetados y el flujo completo con TalkBack |

## Tecnologías Utilizadas

- Python 3.10 a 3.13
- PyTorch y `piano_transcription_inference` (modelo de Kong et al., 2021)
- librosa y NumPy
- FastAPI y Uvicorn
- Flutter (aplicación web progresiva)
- Docker y Railway (despliegue de la API)
- GitHub Actions y GitHub Pages
- Kaggle (entrenamiento y evaluación con GPU)
- axe-core, Lighthouse y TalkBack (auditoría de accesibilidad)

## Requisitos Previos

- Python 3.10 o superior
- ffmpeg, para convertir grabaciones M4A u OGG de teléfono
- 2 GB de RAM libres. No hace falta GPU.
- Flutter del canal stable con Dart 3.13 o superior, solo si se quiere compilar la aplicación web
- Docker, opcional, para correr la API en un contenedor

## Instalación

1. Clonar el repositorio:
   ```bash
   git clone https://github.com/G2309/2026-22779-cruz-bardales-gustavo-adolfo.git
   cd 2026-22779-cruz-bardales-gustavo-adolfo/src
   ```
2. Crear un entorno virtual e instalar las dependencias. PyTorch para CPU se instala primero para no descargar las bibliotecas de CUDA:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install torch --index-url https://download.pytorch.org/whl/cpu
   pip install -r requirements.txt
   ```
3. Configurar las variables de entorno. Ninguna es obligatoria y todas tienen valor por omisión:
   ```bash
   cp .env.example .env
   ```
   El modelo ajustado se descarga solo la primera vez que se convierte un audio y se verifica con su hash.

## Ejecución

Todos los comandos se corren dentro de `src/`.

**Desde la terminal:**
```bash
python convertir.py grabacion.mp3 salida.brf --tempo 100 --compas 3/4 --armadura 1
```
`--armadura` recibe el número de sostenidos en positivo o de bemoles en negativo.

**API y aplicación web en la misma computadora:**
```bash
# API en el puerto 8000
uvicorn api.app:app --app-dir src --port 8000

# En otra terminal, la aplicación apuntando a esa API
cd cliente
flutter build web --release --dart-define=API_URL=http://localhost:8000
cd build/web && python -m http.server 8765
```
Después se abre `http://localhost:8765` en el navegador.

**Con Docker:**
```bash
docker build -t amt-braille-api .
docker run --rm -p 8000:8000 -e OMP_NUM_THREADS=2 amt-braille-api
```

**Pruebas:**
```bash
python -m unittest discover -s tests -v
```

La documentación técnica completa está en [`src/docs/`](src/docs/). Ahí están la arquitectura, los resultados, la instalación y el despliegue, el presupuesto, la escalabilidad, la operación y la seguridad.

## Capturas

Las capturas de pantalla del sistema se encuentran en [`/capturas`](capturas/), en el orden en que se usa el sistema. La aplicación tiene un solo flujo de tres pasos (grabación, datos de la obra y conversión), así que las capturas 01 a 05 muestran ese flujo en sus modos de accesibilidad, y las 06 a 11 muestran la API, la salida Braille y la verificación del sistema.

| # | Pantalla |
|---|---|
| [01](capturas/captura-01.png) | Aplicación web: elección del modo de uso, en teléfono |
| [02](capturas/captura-02.png) | Aplicación web: modo tutor con los tres pasos del flujo, en computadora |
| [03](capturas/captura-03.png) | Aplicación web: conversión terminada con la vista previa del Braille y la descarga del BRF, en teléfono |
| [04](capturas/captura-04.png) | Aplicación web: modo baja visión, con alto contraste y texto grande |
| [05](capturas/captura-05.png) | Aplicación web: modo lector de pantalla |
| [06](capturas/captura-06.png) | API publicada en Railway: documentación interactiva con sus cuatro rutas |
| [07](capturas/captura-07.png) | Conversión de 30 s de la Variación 7 de las Goldberg con la API publicada: el trabajo pasa a `done` en 23 s, con BRF válido |
| [08](capturas/captura-08.png) | El BRF de esa conversión en Braille Unicode, compás sobre compás con las dos manos |
| [09](capturas/captura-09.png) | Traducción determinista de una partitura MusicXML de ejemplo a Braille y a BRF desde la terminal |
| [10](capturas/captura-10.png) | Suite de 249 pruebas automáticas pasando |
| [11](capturas/captura-11.png) | Integración continua en GitHub Actions con las pruebas de cada cambio |

## Demo

El video demostrativo se encuentra en [`/demo/demo.mp4`](demo/demo.mp4).

## Documentación

- El informe final del proyecto está disponible en [`/docs/informe_final.pdf`](docs/informe_final.pdf).
- El consentimiento de uso de foto y video firmado se encuentra en [`/docs/consentimiento_firmado.pdf`](docs/consentimiento_firmado.pdf).

## Autores

- Gustavo Adolfo Cruz Bardales - 22779

Asesor: Ing. Luis Alberto Suriano Saravia. Universidad del Valle de Guatemala, Facultad de Ingeniería.

## Licencia

GNU General Public License v3.0. Ver [`src/LICENSE`](src/LICENSE).
