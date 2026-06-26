# Stack Wars Episodio I — Prompt de generación completa

---

## CONTEXTO Y OBJETIVO

Eres un asistente experto en diseño de juegos educativos y en redes de telecomunicaciones. Tu tarea es generar un juego web multijugador completo llamado **Stack Wars Episodio I: La amenaza en la red**, para la asignatura **Redes basadas en IP** (Máster en Ingeniería de Telecomunicación). El juego se enmarca en una metodología de **Aprendizaje Basado en Juegos (ABJ)** aplicada al aula universitaria.

El juego es la secuela de Stack Wars (Top Trumps), y cambia la mecánica de comparación de atributos por una mecánica de **construcción de pila de protocolos por capas OSI** para resolver misiones de red bajo restricciones dinámicas (cartas de crisis).

El resultado debe incluir:
1. Un archivo HTML único con el juego web completo (multijugador en tiempo real vía Firebase)
2. El sistema completo de cartas de misión (15) y crisis (10)
3. Un README.md al estilo del Episodio original
4. Este prompt completo para regenerar o ampliar el juego

---

## ARQUITECTURA DEL JUEGO

### Stack tecnológico
- HTML + CSS + JS en un único archivo `stack_wars_ep1.html`
- Firebase Realtime Database para sincronización en tiempo real
- Sin frameworks ni dependencias adicionales
- Optimizado para móvil (viewport, touch events, 100dvh)

### Nodo Firebase
El juego escribe en `ep1rooms/{código}` para coexistir con el Episodio original (`rooms/{código}`) en el mismo proyecto Firebase.

### Estructura de sala en Firebase

```json
{
  "code": "ABCD",
  "status": "waiting | playing | finished",
  "p1": {
    "name": "Jugador 1",
    "hand": ["01","04","07","09","12","16"],
    "wins": 0,
    "stack": { "7": "04", "4": "01", "3": "07" },
    "justify": "Elijo HTTPS porque Seguridad=10...",
    "ready": true
  },
  "p2": { "...": "igual que p1" },
  "round": 0,
  "totalRounds": 6,
  "missions": ["M01","M04","M07","M09","M12","M15"],
  "crisis":   ["C02","C05","C01","C08","C03","C10"],
  "phase": "build | reveal"
}
```

### Flujo de ronda

```
build phase:
  Jugador 1 construye pila → envía + justificación → ready=true
  Jugador 2 construye pila → envía + justificación → ready=true
  Cuando p1.ready && p2.ready → phase='reveal'

reveal phase:
  Ambos ven las dos pilas reveladas con puntuaciones
  Solo P1 pulsa "Siguiente ronda" → avanza el round, resetea stacks, phase='build'

Si round >= totalRounds → status='finished'
```

---

## MECÁNICA DE JUEGO

### Formato de jugadores
El juego digital enfrenta a **2 jugadores individuales** (1 vs 1). En la versión física puede jugarse con 2–4 jugadores individuales o en 2 equipos de 2 (óptimo para el aula). Toda la lógica de Firebase y puntuación asume participantes individuales; la modalidad de equipos se gestiona offline.

### Construcción de pila
- Cada jugador recibe 6 cartas de protocolo al inicio (mano aleatoria del mazo de 20)
- La misión activa indica qué capas OSI son obligatorias y qué atributos mínimos se requieren
- El jugador coloca una carta por nodo del tablero circular, seleccionando primero la carta de su mano y luego tocando el nodo de la capa correcta
- **Excepción M10**: esta misión activa dos ranuras en L7 (envío + acceso); el jugador debe colocar una carta distinta en cada ranura
- Las cartas que violan la crisis activa se atenúan y no pueden colocarse
- El jugador tiene 60 segundos (honor system) para construir y justificar su pila

### Puntuación
```
Para cada capa requerida por la misión:
  puntuación += suma de atributos relevantes (scoreAttrs de la misión)
  puntuación -= penalización de crisis (si aplica)
  si la carta viola un requisito obligatorio → esa capa no puntúa

Gana la ronda quien tenga mayor puntuación total.
Empate → nadie puntúa, ambos roban 1 carta.
```

### Penalizaciones de crisis
| Crisis | Penalización |
|---|---|
| C01 UDP bloqueado | L4 con Overhead bajo ≥ 9 sin conexión descalificada (UDP #02, TFTP #19) |
| C02 Ataque DDoS | L7 con Seguridad < 7 descalificada |
| C03 Fallo enlace troncal | L3 con Fiabilidad < 8 descalificada |
| C04 Ancho de banda 10% | Cada carta con Overhead bajo < 6 → −3 pts |
| C05 Auditoría GDPR | L7 con Seguridad < 8 descalificada |
| C06 Dispositivo legacy | Cada carta con Adopción < 8 descalificada |
| C07 L2 no disponible | Ranura L2 bloqueada, misiones con L2 → −5 pts |
| C08 Latencia disparada | L4 con Velocidad < 8 descalificada |
| C09 Filtración de datos | Cada carta con Seguridad ≤ 3 descalificada |
| C10 Congestión en núcleo | L3/L4 con Overhead bajo < 7 → −4 pts |

> **Nota C01**: UDP (#02, OVH=10) y TFTP (#19, OVH=10) quedan descalificados. RTP (#18, OVH=7) **no** queda descalificado por este criterio; su uso en L4 es válido bajo C01.

### Violaciones duras vs. penalizaciones suaves
- **Duras** (C01, C02, C03, C05, C06, C08, C09): la carta queda descalificada, esa capa no puntúa. Si hay múltiples violaciones, la puntuación total se multiplica por 0.6 (reducción del 40%).
- **Suaves** (C04, C10): la carta sigue siendo válida pero acumula una penalización de puntos sobre el total.
- **Especial** (C07): bloquea físicamente la ranura L2. Las misiones que la exigen se resuelven sin ella (penalización fija −5 pts sobre el total).

---

## CARTAS DE PROTOCOLO (mazo compartido con Episodio original)

Las mismas 20 cartas con los mismos valores. 5 atributos valorados del 1 al 10:

| # | Nombre | Capa | Fiabilidad | Velocidad | Overhead bajo | Seguridad | Adopción |
|---|---|---|---|---|---|---|---|
| 01 | TCP   | 4 | 10 | 5  | 3  | 6  | 9  |
| 02 | UDP   | 4 | 3  | 10 | 10 | 3  | 9  |
| 03 | HTTP  | 7 | 7  | 7  | 5  | 2  | 10 |
| 04 | HTTPS | 7 | 8  | 6  | 4  | 10 | 10 |
| 05 | DNS   | 7 | 6  | 9  | 9  | 4  | 10 |
| 06 | ICMP  | 3 | 5  | 9  | 9  | 3  | 8  |
| 07 | OSPF  | 3 | 9  | 6  | 4  | 6  | 7  |
| 08 | BGP   | 3 | 9  | 4  | 3  | 7  | 9  |
| 09 | ARP   | 2 | 7  | 9  | 8  | 2  | 6  |
| 10 | DHCP  | 7 | 7  | 8  | 7  | 3  | 9  |
| 11 | SMTP  | 7 | 8  | 5  | 5  | 5  | 8  |
| 12 | MPLS  | 3 | 9  | 9  | 7  | 7  | 6  |
| 13 | FTP   | 7 | 8  | 7  | 4  | 2  | 7  |
| 14 | SFTP  | 7 | 8  | 6  | 3  | 9  | 7  |
| 15 | SNMP  | 7 | 6  | 8  | 8  | 4  | 8  |
| 16 | NTP   | 7 | 7  | 9  | 9  | 5  | 9  |
| 17 | SIP   | 7 | 7  | 8  | 6  | 6  | 8  |
| 18 | RTP   | 4 | 4  | 10 | 7  | 4  | 8  |
| 19 | TFTP  | 7 | 3  | 9  | 10 | 1  | 5  |
| 20 | IMAP  | 7 | 8  | 6  | 5  | 6  | 8  |

---

## CARTAS DE MISIÓN (15 cartas)

Cada misión tiene: ID, nombre, dificultad, requisitos por capa (`reqs`), atributos que puntúan (`scoreAttrs` como índices 0-4), y opcionalmente un protocolo forzado (`forced`).

### Estructura de un requisito
```json
{
  "layer": 7,
  "attr": 3,
  "min": 9,
  "label": "L7: Seguridad ≥ 9",
  "forced": null
}
```
Si `forced` tiene valor (ID de carta), solo esa carta satisface el requisito independientemente de sus atributos.

### Las 15 misiones

**M01 — Navegación web segura** (Básica)
- L7: Seguridad ≥ 9 | L4: Fiabilidad ≥ 7 | L3: cualquier protocolo
- Puntuación: Seguridad + Fiabilidad de la pila

**M02 — Videoconferencia corporativa** (Básica)
- L7: SIP obligatorio (#17) | L4: Velocidad ≥ 8 | L3: cualquier protocolo
- Puntuación: Velocidad + Adopción de la pila

**M03 — Transferencia masiva de ficheros** (Básica)
- L7: Seguridad ≥ 8 | L4: Fiabilidad ≥ 7 | L3: cualquier protocolo
- Puntuación: Fiabilidad + Seguridad de la pila

**M04 — Telemetría IoT en tiempo real** (Intermedia)
- L7: Overhead bajo ≥ 7 | L4: Overhead bajo ≥ 8 | L3: cualquier protocolo
- Puntuación: Velocidad + Overhead bajo de la pila

**M05 — Resolución de nombres distribuida** (Intermedia)
- L7: DNS obligatorio (#05) | L4: Velocidad ≥ 8 | L3: Adopción ≥ 7
- Puntuación: Velocidad + Overhead bajo de la pila

**M06 — Red de distribución CDN** (Intermedia)
- L7: Adopción = 10 | L4: Velocidad ≥ 8 | L3: Adopción ≥ 8
- Puntuación: Velocidad + Adopción de la pila

**M07 — Acceso remoto seguro VPN** (Intermedia)
- L7: Seguridad ≥ 9 | L4: Fiabilidad ≥ 8 | L3: Seguridad ≥ 6
- Puntuación: Seguridad de todas las capas (scoreAttrs: [3,3,3])

**M08 — Monitorización de infraestructura** (Intermedia)
- L7: SNMP obligatorio (#15) | L4: cualquier protocolo | L3: Overhead bajo ≥ 8
- Puntuación: Adopción + Overhead bajo de la pila

**M09 — Arranque PXE laboratorio diskless** (Avanzada)
- L7: TFTP obligatorio (#19) | L4: Overhead bajo ≥ 9 | L2: ARP obligatorio (#09)
- Puntuación: Velocidad + Overhead bajo de la pila

**M10 — Correo empresarial cifrado** (Avanzada)
- L7a (envío): Fiabilidad ≥ 7 — p. ej. SMTP (#11) | L7b (acceso): protocolo de acceso con sincronización — IMAP recomendado (#20) | L4: Fiabilidad ≥ 8
- Puntuación: Seguridad + Fiabilidad de la pila
- **Nota de implementación**: esta es la única misión con doble ranura en L7. El tablero activa dos nodos en el anillo exterior. El jugador debe colocar dos cartas distintas de L7 en las ranuras L7a y L7b respectivamente.

**M11 — Sincronización de relojes financiera** (Avanzada)
- L7: NTP obligatorio (#16) | L4: Overhead bajo ≥ 8 | L3: Adopción ≥ 8
- Puntuación: Velocidad + Overhead bajo de la pila

**M12 — Backbone inter-AS de ISP** (Avanzada)
- L3: BGP obligatorio (#08) | L4: Fiabilidad ≥ 8 | L7: cualquier protocolo
- Puntuación: Fiabilidad + Adopción de la pila

**M13 — Streaming de audio en directo** (Avanzada)
- L7: SIP obligatorio (#17, señalización de sesión) | L4: RTP obligatorio (#18) | L3: Adopción ≥ 7
- Puntuación: Velocidad + Overhead bajo de la pila
- **Nota**: SIP gestiona la señalización en L7; RTP transporta el flujo de audio en L4. Cada protocolo ocupa su capa propia.

**M14 — Asignación dinámica de IPs** (Básica)
- L7: DHCP obligatorio (#10) | L4: cualquier protocolo | L2: ARP obligatorio (#09)
- Puntuación: Adopción + Velocidad de la pila

**M15 — Red MPLS con SLA garantizado** (Avanzada)
- L3: MPLS obligatorio (#12) | L4: Fiabilidad ≥ 7 | L7: cualquier protocolo
- Puntuación: Velocidad + Fiabilidad de la pila

---

## CARTAS DE CRISIS (10 cartas)

**C01 — UDP bloqueado en el firewall**
Seguridad aplica regla que bloquea todo el tráfico UDP esta ronda.
→ L4 no puede usar carta sin conexión con Overhead bajo ≥ 9: UDP (#02) y TFTP (#19) descalificados. RTP (#18, OVH=7) no queda descalificado por este criterio.

**C02 — Ataque DDoS en curso**
La red sufre un ataque de denegación de servicio.
→ L7 debe tener Seguridad ≥ 7. HTTP (#03), FTP (#13), TFTP (#19) y SNMP (#15) descalificados.

**C03 — Fallo del enlace troncal**
El enlace principal datacenter–ISP ha caído. Solo rutas de respaldo.
→ L3 debe tener Fiabilidad ≥ 8. ICMP (#06) descalificado en L3.

**C04 — Ancho de banda reducido al 10%**
Mantenimiento reduce la capacidad de red disponible al 10%.
→ Cartas con Overhead bajo < 6 penalizan −3 pts sobre la puntuación total.

**C05 — Auditoría GDPR obligatoria**
Regulación exige que todas las comunicaciones sean auditables y cifradas.
→ L7 debe tener Seguridad ≥ 8. HTTP (#03), FTP (#13) y TFTP (#19) prohibidos.

**C06 — Dispositivo legacy crítico**
Un equipo crítico solo soporta protocolos con amplio despliegue en la industria.
→ Todas las cartas de la pila deben tener Adopción ≥ 8.

**C07 — Capa de enlace no disponible**
El switch de la LAN ha fallado. Comunicación directa en Capa 2 imposible.
→ Ranura L2 bloqueada. Misiones con L2 resuelven sin ella (penalización −5 pts sobre el total).

**C08 — Latencia de red disparada**
Cuello de botella introduce 300 ms adicionales en el núcleo.
→ L4 debe tener Velocidad ≥ 8. TCP (#01) queda descalificado en L4.

**C09 — Filtración de datos detectada**
Tráfico no cifrado detectado. Todos los protocolos en texto plano, prohibidos.
→ Ninguna carta con Seguridad ≤ 3 puede colocarse en ninguna ranura.

**C10 — Congestión extrema en el núcleo**
El núcleo de red está saturado al 98%. Gestión de tráfico de emergencia activada.
→ L3 y L4 con Overhead bajo < 7 penalizan −4 pts sobre la puntuación total.

---

## DISEÑO VISUAL

### Paleta de colores (herencia del Episodio original)

| Elemento | Color |
|---|---|
| Fondo de pantalla | `#0f1923` |
| Superficie | `#1a2535` |
| Capa 7 Aplicación | `#378ADD` / fondo `#0a1828` |
| Capa 4 Transporte | `#1D9E75` / fondo `#061a12` |
| Capa 3 Red | `#7F77DD` / fondo `#120f1e` |
| Capa 2 Enlace | `#D85A30` / fondo `#1a0e06` |
| Crisis (banner) | `#D85A30` sobre `#2a1508` |
| Misión (banner) | `#378ADD` sobre `#0a1828` |

### Tablero circular (inspirado en SMART10)

El tablero central es un SVG de 320×320 px que representa el modelo OSI como **anillos concéntricos**. Cada capa OSI requerida por la misión aparece como un nodo circular en su anillo correspondiente:

| Capa | Radio del anillo | Posición |
|---|---|---|
| L2 — Enlace | 0 (centro) | Nodo central único |
| L3 — Red | 72 px | Distribuidos uniformemente en el anillo |
| L4 — Transporte | 112 px | Distribuidos uniformemente en el anillo |
| L7 — Aplicación | 148 px | Distribuidos uniformemente en el anillo |

**Estados de nodo:**
- **Vacío interactivo**: borde punteado + símbolo `+` + animación de pulso suave — toca para colocar la carta seleccionada
- **Ocupado**: fondo de color de la capa + nombre del protocolo + atributos de puntuación + halo de brillo animado
- **Bloqueado** (crisis C07 en L2): borde rojo oscuro + candado 🔒
- **Doble ranura L7** (misión M10): dos nodos en el anillo exterior, etiquetados L7a y L7b

**Interacción en dos pasos:**
1. El jugador toca una carta de su mano → queda resaltada (borde azul)
2. El jugador toca el nodo del anillo correcto → la carta se coloca

Si la carta es de una capa diferente al nodo tocado, aparece un toast de error. Las cartas que violan la crisis activa se atenúan en la mano y no pueden seleccionarse.

### Pantallas del juego
1. **Lobby** — nombre + crear/unirse a sala
2. **Juego (build phase)** — banner de crisis + banner de misión + tablero circular SVG + mano de 6 cartas + botón enviar
3. **Juego (reveal phase)** — banner de crisis + resultado + lista de filas por capa con puntuaciones + botón siguiente
4. **Final** — trofeo + resultado + estadísticas + botón nueva partida

---

## INSTRUCCIONES DE GENERACIÓN

Al ejecutar este prompt, genera en este orden:

1. El archivo `stack_wars_ep1.html` completo con toda la lógica de juego
2. Este prompt como `stack_wars_ep1_prompt.md`
3. El `README.md` con el estilo del Episodio original

Si el usuario pide ampliar el mazo de misiones, mantén la misma estructura de `reqs` y `scoreAttrs`. Calibra los requisitos mínimos siendo fiel a los valores reales de los protocolos: si exiges Seguridad ≥ 9 en L7, solo HTTPS (#04) y SFTP (#14) lo cumplen. Si diseñas una misión con doble ranura en L7 (al estilo de M10), indícalo explícitamente en los `reqs` con `"slot": "7a"` y `"slot": "7b"`.

Si el usuario pide nuevas cartas de crisis, respeta el balance entre crisis duras (descalifican) y suaves (penalizan). No añadas crisis que hagan imposible resolver más de 3 misiones simultáneamente. Para crisis que descalifican protocolos en L4 por protocolo sin conexión, usa el criterio combinado Overhead bajo + ausencia de handshake, no solo el valor de Overhead bajo.

Si el usuario quiere cambiar el tema de la asignatura, adapta los nombres de las capas, los protocolos y los escenarios de misión al nuevo dominio, manteniendo la mecánica de construcción de pila con 3–4 ranuras y el sistema de crisis.

---

## FRAGMENTOS DE LÓGICA CRÍTICA

### Guard anti-race en nextRound (solo P1 avanza)
Solo el jugador 1 llama a la función que incrementa el round y resetea las pilas en Firebase. El jugador 2 ve un mensaje "esperando a P1". Esto evita que ambos escriban simultáneamente al pulsar "Siguiente ronda".

### Re-render local sin consultar Firebase
Al seleccionar carta o colocarla en un nodo, no se hace `get()` a Firebase — se re-renderiza solo el DOM del tablero circular (`.board-wrap`) y la mano (`.hand-grid`) usando el estado local (`S.myStack`, `S.placedCards`, `S.selectedCard`). Solo se hace `get()` para obtener `mission`/`crisis` actuales cuando es necesario (placeCard, undoStack).

### Tablero circular — función renderStackSlots
Genera un SVG de 320×320 px con nodos circulares por capa. Los radios de anillo son fijos: L2=0 (centro), L3=72, L4=112, L7=148. Los nodos vacíos tienen `onclick="placeCard(layer)"` inline en el SVG — requiere que `placeCard` esté expuesta como `window.placeCard`. Los nodos pulsantes usan `@keyframes slot-pulse` en CSS.

Para M10, el anillo L7 genera dos nodos separados angularmente: `placeCard('7a')` y `placeCard('7b')`. La validación de pila completa comprueba que `S.myStack['7a']` y `S.myStack['7b']` estén ambos rellenos antes de habilitar el botón de envío.

### Penalización por violaciones de crisis duras
Si la pila contiene cartas que violan una crisis dura (C01, C02, C03, C05, C06, C08, C09), la puntuación total se multiplica por 0.6 (reducción del 40%) en lugar de invalidar carta a carta. Esto permite pilas parcialmente válidas pero penalizadas.

### Validación de C01
```js
function violatesC01(card) {
  // Solo descalifica protocolos sin conexión con OVH >= 9
  // UDP (#02): OVH=10 → descalificado
  // TFTP (#19): OVH=10 → descalificado
  // RTP (#18): OVH=7 → NO descalificado
  return card.layer === 4 && card.overhead >= 9 && card.reliability < 5;
}
```

---

## METADATOS DEL PROYECTO

| Campo | Valor |
|---|---|
| Nombre del juego | Stack Wars Episodio I: La amenaza en la red |
| Asignatura | Redes basadas en IP |
| Titulación | Máster en Ingeniería de Telecomunicación |
| Marco pedagógico | Aprendizaje Basado en Juegos (ABJ) — Innovación Docente en el Aula Universitaria |
| Mecánica base | Construcción de pila de protocolos por capas OSI |
| Mecánica complementaria | Carta de crisis por ronda (restricción dinámica) |
| Tablero | Circular SVG — capas OSI como anillos concéntricos (inspirado en SMART10) |
| Jugadores | 2 (versión digital) · 2–4 (versión física) |
| Duración estimada | 25–35 minutos por partida (incluyendo preparación y debate) |
| Nivel educativo | Máster universitario |
| Objetivo pedagógico principal | Razonamiento sobre selección de protocolos por capa y trade-offs ante restricciones de red |
| URL de acceso | https://edurag.org/sw |
