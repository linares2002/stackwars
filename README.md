# Stack Wars Episodio I: La amenaza en la red

Juego de mesa educativo para la asignatura de **Redes basadas en IP** (Máster en Ingeniería de Telecomunicación). Disponible en dos formatos: mazo físico imprimible y partida multijugador en tiempo real desde el móvil.

---

## ¿Qué es Stack Wars Episodio I?

Stack Wars Episodio I aplica la mecánica de **construcción de pila de protocolos** al estudio de redes basadas en IP. Cada jugador recibe una mano de 6 cartas de protocolo y debe construir la pila más eficiente para resolver una misión de red, sorteando al mismo tiempo una restricción inesperada (carta de crisis).

A diferencia del Episodio original (Top Trumps), aquí la pregunta no es "¿qué protocolo tiene el atributo más alto?" sino "¿qué combinación de protocolos, capa a capa, resuelve mejor este escenario real?".

Los cinco atributos de cada carta son:

| Atributo | Descripción |
|---|---|
| **Fiabilidad** | ¿Garantiza entrega correcta y ordenada? |
| **Velocidad** | Latencia y rendimiento. Más alto = más rápido. |
| **Overhead bajo** | Eficiencia de cabeceras. 10 = muy ligero. |
| **Seguridad** | Cifrado, autenticación e integridad nativos. |
| **Adopción** | Grado de uso real en Internet hoy. |

El mazo tiene **20 cartas de protocolo**:

| Capa | Protocolos | Color |
|---|---|---|
| Capa 7 — Aplicación | HTTP, HTTPS, DNS, DHCP, SMTP, FTP, SFTP, SNMP, NTP, SIP, TFTP, IMAP | Azul |
| Capa 4 — Transporte | TCP, UDP, RTP | Verde |
| Capa 3 — Red | ICMP, OSPF, BGP, MPLS | Morado |
| Capa 2 — Enlace | ARP | Naranja |

A estos se añaden dos mazos exclusivos del Episodio I:

| Mazo | Cartas | Descripción |
|---|---|---|
| Misiones | 15 cartas | Escenarios de red reales con requisitos por capa y criterio de puntuación |
| Crisis | 10 cartas | Restricciones inesperadas que modifican las condiciones de la ronda |

### Objetivo pedagógico

El momento de aprendizaje clave es la **justificación oral de la pila**: antes de revelar, cada jugador debe explicar por qué cada protocolo elegido cumple los requisitos de la misión y resiste la crisis activa. Eso obliga a razonar sobre trade-offs reales: ¿por qué HTTPS en L7 si la misión exige Seguridad ≥ 9? ¿Por qué no UDP en L4 si el firewall lo bloquea? ¿Cuándo vale la pena el overhead de TCP frente a la ligereza de UDP? Ese razonamiento de diseño de red es exactamente el que se trabaja en la asignatura.

---

## Contenido del repositorio

```
stack-wars-ep1/
├── index.html                 # Juego multijugador en tiempo real (Firebase)
├── stack_wars_ep1_prompt.md   # Prompt completo para regenerar o ampliar el juego con IA
├── qr_sw.png                  # Código QR — https://edurag.org/sw
└── README.md
```

---

## Cómo jugar

### Componentes

- 20 cartas de protocolo (reutilizadas del Episodio original)
- 15 cartas de misión
- 10 cartas de crisis
- Tablero circular con nodos de capa (L2 / L3 / L4 / L7)
- Marcador de rondas (1–6)

### El tablero circular

El tablero representa el modelo OSI como **anillos concéntricos**: L2 en el centro, L3 en el primer anillo, L4 en el segundo y L7 en el exterior. Cada misión activa solo los nodos de las capas que exige, de modo que el jugador ve exactamente qué capas necesita cubrir. La estructura visual refleja directamente la jerarquía real del modelo de red: las capas más bajas (más técnicas) están en el núcleo, y las más cercanas al usuario en el exterior.

### Preparación

1. Barajar por separado los tres mazos: protocolos, misiones y crisis.
2. Repartir 6 cartas de protocolo a cada jugador (mano oculta).
3. Colocar **1 carta de misión** boca arriba en el centro de la mesa.
4. Revelar 1 carta de crisis que afectará a toda la ronda.
5. Decidir quién empieza: el jugador que nombre más protocolos en 10 segundos.

### Reglas básicas

1. Cada jugador lee en silencio la misión y la crisis activa.
2. En 60 segundos, cada jugador coloca cartas de su mano en los nodos del tablero para cubrir las capas requeridas por la misión.
3. Antes de revelar, el jugador **justifica en voz alta** por qué cada protocolo elegido cumple la misión y supera la restricción de crisis. Sin justificación = pierde la ronda.
4. Ambos jugadores revelan su pila simultáneamente.
5. Se calcula la puntuación de cada pila (suma de atributos relevantes indicados en la misión, menos penalizaciones de crisis).
6. Gana la ronda quien tenga mayor puntuación. En caso de empate, nadie puntúa y ambos roban 1 carta.
7. El ganador de la ronda roba 2 cartas nuevas; el perdedor roba 1.

> Las cartas que violan la crisis quedan descalificadas o penalizadas. Una pila técnicamente perfecta puede perder si ignora la restricción activa.

### Nota sobre la misión M10 (Correo empresarial cifrado)

Esta misión es la única que exige **dos protocolos distintos en Capa 7**: uno de envío (p. ej. SMTP) y otro de acceso con sincronización (p. ej. IMAP). El tablero activa dos ranuras en L7 para esta misión. La puntuación se calcula sobre ambas cartas de forma independiente.

### Fin de la partida

La partida termina al completar 6 rondas o cuando un jugador consiga 4 victorias. Gana quien haya resuelto más misiones. En caso de empate, se juega una ronda extra con la misión de mayor dificultad disponible.

### Variantes para el aula

- **Debate obligatorio** — sin justificación oral, la pila se invalida automáticamente. Fomenta el razonamiento técnico explícito.
- **Modo experto** — sin límite de tiempo. Se valora la calidad de la argumentación técnica, no la velocidad.
- **Capa prohibida** — se sortea al inicio una capa que no puede usarse en toda la partida. Obliga a soluciones creativas.
- **Torneo** — eliminatorias entre parejas. Ideal para sesiones de 45 min con 8–16 estudiantes.

---

## Versión digital multijugador (móvil)

`index.html` es un juego web que funciona en el navegador del móvil sin instalar nada. Dos jugadores se conectan en tiempo real a través de Firebase Realtime Database. Comparte la misma infraestructura Firebase que el Episodio original, usando el nodo `ep1rooms/` para que ambos juegos coexistan sin interferirse.

### Requisitos

- La misma cuenta y proyecto Firebase del Episodio original (plan gratuito suficiente).
- Un servidor web para servir el archivo HTML.

### Configuración de Firebase

Pega el mismo bloque `firebaseConfig` de Stack Wars en `index.html` (líneas ~230-238):

```js
const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  databaseURL: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

No es necesario crear nada nuevo en Firebase: el juego escribe en el nodo `ep1rooms/` automáticamente.

### Acceso directo

El juego está disponible públicamente en: [https://edurag.org/sw](https://edurag.org/sw)

Los alumnos solo necesitan escanear el QR o abrir la URL en el navegador de su móvil — sin instalaciones ni configuración adicional.

![QR code — https://edurag.org/sw](qr_sw.png)

### Flujo de partida en el móvil

1. **Jugador 1** escribe su nombre → pulsa **Crear sala** → obtiene un código de 4 letras.
2. **Jugador 2** escribe su nombre + ese código → pulsa **Unirse**.
3. Cada jugador ve en su pantalla la crisis activa, la misión de la ronda y su mano de 6 cartas.
4. El jugador toca una carta de su mano (queda resaltada) y luego toca el nodo del anillo correspondiente en el tablero circular para colocarla.
5. Al completar la pila, pulsa **Enviar pila** → se abre un modal para escribir la justificación.
6. Cuando los dos han enviado, ambas pilas se revelan simultáneamente en los dos dispositivos con la puntuación calculada capa a capa.
7. Jugador 1 pulsa **Siguiente ronda** para avanzar.
8. Tras 6 rondas aparece el marcador final.

### Despliegue

Para replicarlo en otro servidor, copia `index.html` a cualquier hosting estático — no requiere backend propio, toda la sincronización va a través de Firebase.

---

## El tablero circular — diseño inspirado en SMART10

El tablero digital está inspirado en la plantilla física del juego **SMART10** (SD Games), que organiza los componentes alrededor de un círculo central con nodos satélite distribuidos en anillos. En Stack Wars Episodio I esta geometría se adapta al modelo OSI:

- **Centro** → L2 (Enlace): la capa física, el núcleo de la red
- **Primer anillo** (r = 72 px) → L3 (Red): encaminamiento y direccionamiento
- **Segundo anillo** (r = 112 px) → L4 (Transporte): control de flujo y fiabilidad
- **Anillo exterior** (r = 148 px) → L7 (Aplicación): protocolos de usuario

Solo aparecen los nodos de las capas exigidas por la misión activa. Los nodos vacíos pulsan suavemente indicando que se puede colocar una carta; los nodos ocupados muestran el nombre del protocolo y sus atributos de puntuación con un halo del color de su capa.

---

## Ampliar el juego

El archivo `stack_wars_ep1_prompt.md` contiene el prompt completo para regenerar o ampliar el juego. Al ejecutarlo se obtiene el código del juego web, las cartas de misión y crisis, y el diseño visual de los componentes.

Reglas de calibración para nuevas cartas de misión:
- Un protocolo sin cifrado nativo nunca debe satisfacer un requisito de Seguridad ≥ 7.
- Los requisitos de capa deben reflejar el modelo OSI real: no se puede exigir TCP en L7.
- Las misiones básicas deben resolverse con cualquier combinación razonable; las avanzadas deben tener al menos un protocolo obligatorio (`forced`).
- La misión M10 es la única con doble ranura en L7; cualquier misión nueva con este patrón debe indicarlo explícitamente.

Reglas de calibración para nuevas cartas de crisis:
- Una crisis no debe hacer imposible resolver más de 3 misiones del mazo simultáneamente.
- Las crisis duras (que descalifican cartas) deben dejar al menos 2 opciones válidas por capa.
- Las crisis de penalización (−pts) son más suaves y admiten más estrategias.
- C01 descalifica en L4 los protocolos sin conexión con Overhead bajo ≥ 9: UDP (#02) y TFTP (#19). RTP (#18) tiene Overhead bajo = 7 y no queda descalificado por este criterio; su exclusión debe justificarse por ser un protocolo basado en UDP si se desea incluirlo.

Para cambiar el dominio temático (de protocolos de red a otro campo), adapta los nombres de las capas y los atributos al nuevo contexto manteniendo la mecánica de construcción de pila con misión + crisis y el tablero circular con el número de anillos que corresponda.

---

## Contexto académico

| Campo | Valor |
|---|---|
| Asignatura | Redes basadas en IP |
| Titulación | Máster en Ingeniería de Telecomunicación |
| Marco pedagógico | Aprendizaje Basado en Juegos (ABJ) |
| Mecánica base | Construcción de pila + deducción colaborativa |
| Tablero | Circular inspirado en SMART10 |
| Jugadores | 2 (versión digital) · 2–4 (versión física) |
| Duración | 25–35 min por partida (incluyendo preparación y debate) |

---

## Licencia

CC BY-NC-SA 4.0
