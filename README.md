# Stack Wars — Episodio I: La amenaza en la red

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

El mazo comparte las mismas **20 cartas de protocolo** del Episodio original:

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
├── stack_wars_ep1.html        # Juego multijugador en tiempo real (Firebase)
├── stack_wars_ep1_prompt.md   # Prompt completo para regenerar o ampliar el juego con IA
└── README.md
```

---

## Cómo jugar

### Componentes

- 20 cartas de protocolo (reutilizadas del Episodio original)
- 15 cartas de misión
- 10 cartas de crisis
- Tablero con 4 ranuras de capa (L2 / L3 / L4 / L7)
- Marcador de rondas

### Preparación

1. Barajar por separado los tres mazos: protocolos, misiones y crisis.
2. Repartir 6 cartas de protocolo a cada jugador (mano oculta).
3. Colocar 1 carta de misión boca arriba en el centro de la mesa.
4. Revelar 1 carta de crisis que afectará a toda la ronda.

### Reglas básicas

1. Cada jugador lee en silencio la misión y la crisis activa.
2. En 60 segundos, cada jugador coloca cartas de su mano en las ranuras del tablero para cubrir las capas requeridas por la misión.
3. Antes de revelar, el jugador **justifica en voz alta** por qué cada protocolo elegido cumple la misión y supera la restricción de crisis. Sin justificación = pierde la ronda.
4. Ambos jugadores revelan su pila simultáneamente.
5. Se calcula la puntuación de cada pila (suma de atributos relevantes indicados en la misión, menos penalizaciones de crisis).
6. Gana la ronda quien tenga mayor puntuación. En caso de empate, nadie puntúa.
7. El ganador de la ronda roba 2 cartas nuevas; el perdedor roba 1.

> Las cartas que violan la crisis quedan descalificadas o penalizadas. Una pila técnicamente perfecta puede perder si ignora la restricción activa.

### Fin de la partida

La partida termina al completar 6 rondas o cuando un equipo consiga 4 victorias. Gana quien haya resuelto más misiones. En caso de empate, se juega una ronda extra con la misión de mayor dificultad disponible.

### Variantes para el aula

- **Debate obligatorio** — sin justificación oral, la pila se invalida automáticamente. Fomenta el razonamiento técnico explícito.
- **Modo experto** — sin límite de tiempo. Se valora la calidad de la argumentación técnica, no la velocidad.
- **Capa prohibida** — se sortea al inicio una capa que no puede usarse en toda la partida. Obliga a soluciones creativas.
- **Torneo** — eliminatorias entre parejas. Ideal para sesiones de 45 min con 8–16 estudiantes.

---

## Versión digital multijugador (móvil)

`stack_wars_ep1.html` es un juego web que funciona en el navegador del móvil sin instalar nada. Dos jugadores se conectan en tiempo real a través de Firebase Realtime Database. Comparte la misma infraestructura Firebase que el Episodio original, usando el nodo `ep1rooms/` para que ambos juegos coexistan sin interferirse.

### Requisitos

- La misma cuenta y proyecto Firebase del Episodio original (plan gratuito suficiente).
- Un servidor web para servir el archivo HTML.

### Configuración de Firebase

Pega el mismo bloque `firebaseConfig` de Stack Wars en `stack_wars_ep1.html` (líneas ~230-238):

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
4. Cada jugador toca una carta de su mano y luego la ranura de capa donde quiere colocarla — los móviles son independientes.
5. Al completar la pila, pulsa **Enviar pila** → se abre un modal para escribir la justificación.
6. Cuando los dos han enviado, ambas pilas se revelan simultáneamente en los dos dispositivos con la puntuación calculada.
7. Jugador 1 pulsa **Siguiente ronda** para avanzar.
8. Tras 6 rondas aparece el marcador final.

### Despliegue

Para replicarlo en otro servidor, copia `stack_wars_ep1.html` a cualquier hosting estático — no requiere backend propio, toda la sincronización va a través de Firebase.

---

## Ampliar el juego con IA

El archivo `stack_wars_ep1_prompt.md` contiene el prompt completo para regenerar o ampliar el juego usando Claude (u otro LLM). Al ejecutarlo se obtiene el código del juego web, las cartas de misión y crisis, y el diseño visual de los componentes.

Reglas de calibración para nuevas cartas de misión:
- Un protocolo sin cifrado nativo nunca debe satisfacer un requisito de Seguridad ≥ 7.
- Los requisitos de capa deben reflejar el modelo OSI real: no se puede exigir TCP en L7.
- Las misiones básicas deben resolverse con cualquier combinación razonable; las avanzadas deben tener al menos un protocolo obligatorio (`forced`).

Reglas de calibración para nuevas cartas de crisis:
- Una crisis no debe hacer imposible resolver todas las misiones del mazo.
- Las crisis duras (que descalifican cartas) deben dejar al menos 2 opciones válidas por capa.
- Las crisis de penalización (−pts) son más suaves y admiten más estrategias.

Para cambiar el dominio temático (de protocolos de red a otro campo), adapta los nombres de las capas y los atributos al nuevo contexto manteniendo la mecánica de construcción de pila con misión + crisis.

---

## Relación con el Episodio original

| Elemento | Stack Wars (original) | Stack Wars Episodio I |
|---|---|---|
| Mecánica base | Top Trumps — comparación de atributos | Construcción de pila por capas OSI |
| Decisión clave | ¿Qué atributo elige el jugador activo? | ¿Qué protocolo encaja en cada capa para esta misión? |
| Restricción dinámica | Ninguna | Carta de crisis por ronda |
| Justificación | Variante opcional | Obligatoria (invalida la pila si falta) |
| Rondas | 10 | 6 |
| Cartas en mano | 10 (todo el mazo) | 6 (mano parcial aleatoria) |
| Mazos adicionales | — | 15 misiones + 10 crisis |

Los dos juegos son complementarios y pueden jugarse en la misma sesión: el Episodio original como introducción a los protocolos y sus atributos, y el Episodio I como aplicación a escenarios de diseño de red.

---

## Contexto académico

| Campo | Valor |
|---|---|
| Asignatura | Redes basadas en IP |
| Titulación | Máster en Ingeniería de Telecomunicación |
| Marco pedagógico | Aprendizaje Basado en Juegos (ABJ) |
| Mecánica base | Construcción de pila + deducción colaborativa |
| Jugadores | 2 (versión digital) · 2–4 (versión física) |
| Duración | 25–35 min por partida |

---

## Licencia

CC BY-NC-SA 4.0
