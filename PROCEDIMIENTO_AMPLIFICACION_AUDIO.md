# Procedimiento de Amplificación de Audio desde Video MP4

## Problema
Amplificar el audio de un archivo de video MP4 sin acceso a FFmpeg en el sistema.

## Solución Implementada

### 1. **Instalación de librerías Python**
```bash
pip install av numpy scipy
```

- `av` (PyAV): Decodifica y codifica archivos multimedia sin necesidad de FFmpeg externo
- `numpy`: Manipulación de arrays de audio
- `scipy`: Procesamiento de señales de audio

### 2. **Extracción de Audio del Video**
```python
import av
import numpy as np

container = av.open("SOS 1.mp4")
audio_stream = container.streams.audio[0]

# Decodificar todos los frames de audio
audio_frames = []
for frame in container.decode(audio_stream):
    audio_frames.append(frame.to_ndarray())

# Combinar en un solo array
audio_data = np.concatenate(audio_frames, axis=1)
```

**Resultado**: Array NumPy con forma (1, 7548928) = 1 canal mono, 7.5M muestras

### 3. **Amplificación del Audio**
```python
# Amplificar 3x
amplified = audio_data.astype(np.float32) * 3.0

# Limitar rango [-1, 1] para evitar clipping
amplified = np.clip(amplified, -1.0, 1.0)

# Convertir a int16 (formato estándar de audio)
audio_int16 = (amplified * 32767).astype(np.int16)
```

**Amplificación**: 3x = aproximadamente +9.5 dB
**Clipping**: Se aplica para evitar distorsión por saturación

### 4. **Guardado como WAV**
```python
from scipy.io import wavfile

wavfile.write(output_wav, int(audio_stream.sample_rate), audio_int16.T)
```

**Por qué WAV:**
- Formato sin compresión (audio limpio)
- Compatible universalmente
- Sin dependencias externas
- scipy.io.wavfile no requiere FFmpeg

**Parámetros:**
- Frecuencia de muestreo: 44100 Hz (del original)
- Canales: Mono a estéreo (transpose con `.T`)
- Profundidad: 16 bits (int16)

### 5. **Subida a GitHub**
```bash
cd /projects/sandbox/Grabaciones
git add SOS_1_amplified.wav
git commit -m "Audio amplificado WAV"
git push origin main
```

**Resultado**: Archivo disponible en GitHub para descargar

---

## Código Completo

```python
import av
import numpy as np
from scipy.io import wavfile

# Archivos
video_file = "/projects/sandbox/Grabaciones/SOS 1.mp4"
output_wav = "/projects/sandbox/Grabaciones/SOS_1_amplified.wav"

# 1. Extrar audio del video
print("🎵 Extrayendo audio...")
container = av.open(video_file)
audio_stream = container.streams.audio[0]

audio_frames = []
for frame in container.decode(audio_stream):
    audio_frames.append(frame.to_ndarray())

audio_data = np.concatenate(audio_frames, axis=1)
print(f"✓ Audio extraído: {audio_data.shape}")

# 2. Amplificar
print("📈 Amplificando 3x...")
amplified = audio_data.astype(np.float32) * 3.0
amplified = np.clip(amplified, -1.0, 1.0)
print(f"✓ Rango: [{amplified.min():.3f}, {amplified.max():.3f}]")

# 3. Convertir a int16
audio_int16 = (amplified * 32767).astype(np.int16)

# 4. Guardar como WAV
print("💾 Guardando WAV...")
wavfile.write(output_wav, int(audio_stream.sample_rate), audio_int16.T)
print(f"✅ Archivo guardado: {output_wav}")
```

---

## Por qué funcionó

| Intento | Problema | Por qué falló |
|---------|----------|--------------|
| 1 | MP4 con audio AAC comprimido | Necesitaba decodificador AAC (FFmpeg) |
| 2 | pydub + MoviePy | Ambas requieren FFmpeg externo |
| 3 | PyAV con codificación AAC | Problema al muxear audio + video |
| 4 | **WAV con scipy.io** | ✅ **Sin dependencias externas, formato simple** |

## Requisitos Finales

- **Python 3.9+**
- **av** (PyAV)
- **numpy**
- **scipy**

## Resultado Final

- **Archivo original**: SOS 1.mp4 (18 MB, video + audio)
- **Archivo amplificado**: SOS_1_amplified.wav (15 MB, solo audio)
- **Amplificación**: 3x (+9.5 dB)
- **Formato**: WAV 44.1 kHz, 16-bit, mono
- **Estado**: ✅ Reproduce en cualquier reproductor multimedia

---

## Descargar

```
https://github.com/HenrySali/Grabaciones/raw/main/SOS_1_amplified.wav
```
