# Automatización para Actualizar ESPHome Automáticamente

## Descripción

Esta automatización de Home Assistant detecta automáticamente cuando existe una actualización disponible para uno o más dispositivos ESPHome y ejecuta la instalación de esas actualizaciones sin intervención manual.

La automatización revisa las entidades update asociadas a la integración de ESPHome. Si alguna de ellas está en estado on, significa que hay una actualización disponible y se procede a instalarla.

---

## Código de la Automatización

yaml alias: Actualizar ESPHome automáticamente description: "" trigger:   - platform: template     value_template: >-       {{ integration_entities('esphome') | select('match','^update') |       select('is_state', 'on') | list | count > 0 }} condition: [] action:   - repeat:       for_each: >-         {{ states.update | selectattr('state', 'eq', 'on') |          map(attribute='entity_id') |         select('in',integration_entities('esphome')) | list }}       sequence:         - service: update.install           data: {}           target:             entity_id: "{{ repeat.item }}"         - wait_template: "{{ is_state(repeat.item, 'off') }}"           continue_on_timeout: true mode: single 

---

## ¿Qué hace esta automatización?

La automatización realiza los siguientes pasos:

1. Detecta si existe alguna entidad de actualización de ESPHome en estado on.
2. Obtiene la lista de dispositivos ESPHome pendientes de actualización.
3. Recorre cada entidad encontrada.
4. Ejecuta el servicio update.install.
5. Espera a que la actualización termine.
6. Continúa con el siguiente dispositivo, si existe.

---

## Requisitos

- Home Assistant instalado y funcionando.
- Integración ESPHome configurada.
- Dispositivos ESPHome agregados a Home Assistant.
- Entidades update disponibles para los dispositivos ESPHome.
- Acceso OTA funcional en cada dispositivo ESPHome.

---

## Funcionamiento del Trigger

yaml trigger:   - platform: template     value_template: >-       {{ integration_entities('esphome') | select('match','^update') |       select('is_state', 'on') | list | count > 0 }} 

Este disparador se activa cuando al menos una entidad de actualización de ESPHome está en estado on.

En Home Assistant, una entidad update normalmente está en estado on cuando hay una actualización disponible.

---

## Funcionamiento del Repeat

yaml repeat:   for_each: >-     {{ states.update | selectattr('state', 'eq', 'on') |      map(attribute='entity_id') |     select('in',integration_entities('esphome')) | list }} 

Esta sección obtiene todas las entidades de tipo update que cumplen dos condiciones:

- Están en estado on.
- Pertenecen a la integración ESPHome.

Luego ejecuta la secuencia de actualización para cada una.

---

## Instalación de la Actualización

yaml - service: update.install   data: {}   target:     entity_id: "{{ repeat.item }}" 

Este bloque ejecuta la instalación de la actualización disponible para el dispositivo ESPHome correspondiente.

La variable repeat.item contiene la entidad update que se está procesando en ese momento.

---

## Espera de Finalización

yaml - wait_template: "{{ is_state(repeat.item, 'off') }}"   continue_on_timeout: true 

Después de iniciar la actualización, la automatización espera a que la entidad vuelva al estado off.

El estado off indica que ya no hay actualización pendiente.

La opción:

yaml continue_on_timeout: true 

permite que la automatización continúe aunque la espera llegue a un tiempo máximo definido por Home Assistant.

---

## Modo de Ejecución

yaml mode: single 

Esto evita que la automatización se ejecute varias veces al mismo tiempo.

Si ya hay una actualización en proceso, Home Assistant no iniciará otra ejecución paralela de la misma automatización.

---

## Ventajas

- Mantiene los dispositivos ESPHome actualizados automáticamente.
- Evita revisar manualmente cada dispositivo.
- Actualiza solo dispositivos con actualización disponible.
- Procesa los dispositivos uno por uno.
- Reduce el riesgo de ejecuciones simultáneas.

---

## Consideraciones Importantes

Antes de usar esta automatización, tome en cuenta lo siguiente:

- Si un dispositivo ESPHome falla durante la actualización, podría requerir intervención manual.
- Si un dispositivo está desconectado, la actualización podría fallar.
- Es recomendable probar primero con pocos dispositivos.
- No se recomienda usar esta automatización en dispositivos críticos sin supervisión.
- Las actualizaciones OTA dependen de que el dispositivo esté conectado correctamente a la red WiFi.

---

## Recomendaciones

- Mantener respaldos de las configuraciones ESPHome.
- Verificar que las actualizaciones OTA funcionen antes de automatizar el proceso.
- Usar esta automatización inicialmente en dispositivos no críticos.
- Revisar los logs de Home Assistant si alguna actualización falla.
- Evitar apagar Home Assistant durante una actualización.

---

## Flujo General

text Home Assistant detecta actualización ESPHome                  │                  ▼ Entidad update pasa a estado on                  │                  ▼ Se ejecuta la automatización                  │                  ▼ Se obtiene la lista de dispositivos pendientes                  │                  ▼ Se instala la actualización en cada dispositivo                  │                  ▼ La entidad update vuelve a off                  │                  ▼ Automatización finalizada 

---

## Posibles Mejoras Futuras

- Enviar una notificación antes de iniciar las actualizaciones.
- Enviar una notificación cuando finalicen las actualizaciones.
- Excluir dispositivos críticos.
- Ejecutar actualizaciones solo en horarios específicos.
- Agregar un tiempo máximo de espera por dispositivo.
- Registrar errores en una entidad o helper.
- Crear una lista manual de dispositivos permitidos.

---

## Licencia

Uso libre para proyectos personales, educativos y de automatización residencial.

Puede modificarse y adaptarse según las necesidades de cada instalación.
