# Bacheta — Registrador Autónomo de Rugosidad Vial

Dispositivo autónomo embarcado para medir el estado de pavimentos (baches, deformaciones e índice de rugosidad), georreferenciarlos y generar mapas de evidencia técnica.

## Estructura del Proyecto (Crear carpetas bajo demanda)

- `docs/` -> Diagramas de pines, datasheets y fórmulas matemáticas.
- `firmware/` -> Proyecto PlatformIO (C++) para ESP32.
  - `firmware/src/` -> Código fuente C++ (IMU, GPS, botones, SD, Wi-Fi).
  - `firmware/data/` -> Interfaz web estática (HTML/JS/Leaflet) servida por el ESP32.
  - `firmware/wokwi/` -> `diagram.json` para emular el circuito en Wokwi Web.
- `simulation/` -> Scripts en Python para generar recorridos sintéticos con baches y visualizar mapas.
- `enclosure-3d/` -> Modelos paramétricos en OpenSCAD (`.scad`) y exportables (`.stl`).

## Etapas del Proyecto y Detalles Técnicos

- [ ] **Etapa 1 — Arquitectura y Simulación Virtual:**
  - Crear estructura de carpetas inicial según necesidad.
  - Generar script en Python que simule un recorrido GPS real con vibración base de asfalto, baches artificiales (picos en eje Z) y eventos de botones manuales, exportando un mapa HTML interactivo.
  - Configurar `firmware/wokwi/diagram.json` para probar el circuito en Wokwi Web gratuito.
- [ ] **Etapa 2 — Firmware Core (ESP32):**
  - Lectura de IMU 3D (MPU6050/ICM-42688) a alta frecuencia (100-200 Hz).
  - Orientación por cuaterniones para proyectar la aceleración al eje vertical real de la Tierra (descontando inclinación del montaje y pendientes).
  - Fusión con GPS (10 Hz) y registro continuo en tarjeta MicroSD (CSV) sin perder muestras (uso de buffer/tareas FreeRTOS).
  - Lectura con antirrebote (_debounce_) de botones físicos para marcar eventos manualmente durante el manejo.
- [ ] **Etapa 3 — Portal Web Móvil y API de Sincronización:**
  - Configurar ESP32 en modo **Wi-Fi Cliente (STA)** para conectarse al Hotspot del teléfono (manteniendo internet en el móvil para cargar mapas reales) con acceso por `http://bacheta.local` (mDNS).
  - Implementar apagado/encendido de Wi-Fi por botón físico para máximo ahorro energético mientras graba autónomamente en MicroSD.
  - Crear API REST / WebSockets en el ESP32 que entregue **eventos en tiempo real** (hacia adelante) y permita **paginación histórica hacia atrás** desde el archivo CSV de la SD para reconstruir recorridos previos sin perder muestras.
  - Servir interfaz web liviana (`firmware/data/`) con telemetría en vivo, mapa de calor/puntos (Leaflet) y descarga de archivos CSV desde el navegador del teléfono.
- [ ] **Etapa 4 — Diseño Mecánico 3D:**
  - Modelar caja paramétrica en OpenSCAD (`enclosure-3d/bacheta_case.scad`) con soportes internos para ESP32, IMU (rígida al chasis), GPS, módulo SD, batería y pulsadores superiores ergonómicos.
- [ ] **Etapa 5 — Pruebas de Campo y Calibración:**
  - Calibrar filtro pasa-banda / RMS para distinguir vibración normal del motor frente a baches reales y lomos de burro.
- [ ] **Etapa 6 — Escalabilidad Futura:**
  - Arquitectura preparada para integrar GPS RTK (precisión centimétrica) y cámara/láser transversal en una SBC (Raspberry Pi/Jetson) para medir perfil a lo ancho del carril ignorando objetos sobre el plano de la calzada.
