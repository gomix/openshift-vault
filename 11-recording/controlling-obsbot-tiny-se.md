## Estado actual

La webcam OBSBOT Tiny SE está funcionando correctamente en Fedora usando el stack estándar Linux UVC/V4L2.

---

# Lo que ya funciona

## Video UVC estándar

La cámara es detectada como dispositivo UVC normal (`/dev/video*`), por lo que funciona con:

- OBS Studio
- Browsers
- WebRTC
- ffplay
- mpv
- cheese
- v4l2-ctl

---

## PTZ (Pan/Tilt/Zoom)

Se validaron controles PTZ vía V4L2:

```bash
v4l2-ctl -d /dev/video0 --list-ctrls
```

Controles detectados:

```text
pan_absolute
tilt_absolute
focus_absolute
exposure_time_absolute
```

Esto confirma:

- Pan funcional
- Tilt funcional
- Focus manual funcional
- Exposure manual funcional

---

## Gesture Control

También se validó:

- Gestos funcionando
- Activación de tracking mediante gestos

Esto indica que parte de la lógica AI está implementada directamente en firmware.

---

# Lo que NO está disponible actualmente

## Aplicación Linux oficial

No existe soporte Linux oficial completo para:

- GUI de configuración
- SDK Linux completo
- Configuración avanzada de tracking
- Presets desde aplicación oficial

El soporte oficial de OBSBOT está enfocado principalmente en:

- Windows
- macOS

---

# Hallazgo importante

El tracking AI sí funciona bajo Linux.

Eso significa que:

- La cámara puede seguir al usuario
- Reacciona a gestos
- No depende completamente del software propietario

Esto es especialmente valioso en Fedora + Wayland.

---

# Gestos identificados

## Activar tracking

Normalmente usando gesto tipo “L” con la mano.

---

## Repetir gesto

Dependiendo del firmware:

- pausa tracking
- reactiva tracking
- lock/unlock tracking

---

# Herramientas útiles en Linux

## Ver controles

```bash
v4l2-ctl -d /dev/video0 --list-ctrls
```

---

## Exposure manual

```bash
v4l2-ctl -d /dev/video0 \
  --set-ctrl=auto_exposure=1
```

---

## Ajustar exposure

```bash
v4l2-ctl -d /dev/video0 \
  --set-ctrl=exposure_time_absolute=150
```

---

## Mover cámara

### Pan

```bash
v4l2-ctl -d /dev/video0 \
  --set-ctrl=pan_absolute=0
```

### Tilt

```bash
v4l2-ctl -d /dev/video0 \
  --set-ctrl=tilt_absolute=0
```

---

# Stack recomendado para Fedora

## Cámara

- OBSBOT Tiny SE

## Grabación / streaming

- OBS Studio

## Control bajo nivel

- v4l2-ctl

## Audio/video backend

- PipeWire

---

# Conclusión técnica

La OBSBOT Tiny SE presenta muy buena compatibilidad con Linux:

- PTZ funcional
- Tracking funcional
- Compatible con Wayland
- Compatible con Fedora
- Compatible con OBS Studio

Esto la convierte en una excelente webcam para:

- workshops técnicos
- demos OpenShift
- creación de contenido
- streaming
- home office técnico

