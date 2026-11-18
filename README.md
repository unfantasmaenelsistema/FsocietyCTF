<p align="center">
  <a href="https://www.unfantasmaenelsistema.com/"><img src="docs/logo.png" alt="Un Fantasma en el Sistema" width="110" /></a>
</p>

<h1 align="center">Fsociety CTF · Operación 5/9</h1>

<p align="center">
  Juego <em>Capture The Flag</em> que se juega entero en el navegador: seis fases de intrusión encadenadas desde una terminal Kali simulada.
</p>

<p align="center">
  <a href="https://unfantasmaenelsistema.github.io/FsocietyCTF/"><strong>▶ Jugar ahora</strong></a>
  ·
  <a href="https://www.unfantasmaenelsistema.com/">unfantasmaenelsistema.com</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black" alt="React 19" />
  <img src="https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/Vite-646CFF?logo=vite&logoColor=white" alt="Vite" />
  <img src="https://img.shields.io/badge/D3.js-F9A03C?logo=d3dotjs&logoColor=black" alt="D3.js" />
  <img src="https://img.shields.io/badge/licencia-Apache_2.0-00ff41" alt="Licencia Apache 2.0" />
</p>

![Pantalla de inicio](docs/capturas/01-inicio.png)

## Qué es

Encarnas a un operador de fsociety que debe llegar hasta la cámara acorazada de ECorp. No hay servidores ni máquinas reales: todo (red, servicios, credenciales y respuestas de las herramientas) está simulado en el cliente, así que puede usarse en clase sin instalar nada y sin riesgo.

La partida se guarda en el navegador, puntúa cada acción y termina con un rango (S+, A, B o C), un ranking local y un certificado en PDF con código QR.

> **Simulador educativo.** Todos los sistemas, direcciones y credenciales son ficticios. Es un proyecto de fans sin relación con los titulares de *Mr. Robot*.

## Las seis fases

| Fase | Objetivo | Técnica que se practica |
| :-- | :-- | :-- |
| 1 · Brecha perimetral | Portal web de ECorp | Reconocimiento con `nmap` e inyección SQL con `sqlmap` |
| 2 · Red interna | Impresora de la subred | Descubrimiento de hosts y FTP anónimo |
| 3 · Servidor de seguridad | Control de accesos | Enumeración SMB y clonado de tarjeta RFID |
| 4 · Despacho del CEO | Estación de trabajo | Fuerza bruta SSH y decodificación Base64 |
| 5 · Perímetro exterior | Alarma y cámara IP | Portal cautivo Wi-Fi, API sin proteger y credenciales por defecto en IoT |
| 6 · Cámara acorazada | Controlador de la bóveda | Acceso por puerto serie |

Cada fase deja en el sistema de archivos del nodo notas e historiales (`ls -la`, `cat`) con las pistas necesarias para la siguiente.

## Cómo se juega

![Terminal, mapa de red e IRC](docs/capturas/02-terminal-mapa-irc.png)

- **Terminal**: escribe `help` para ver los comandos de la fase. Tiene autocompletado con `Tab`, historial con `↑` / `↓`, `Ctrl+L` para limpiar y `Ctrl+C` para abortar.
- **Alerta IDS**: los escaneos y los ataques mal dirigidos hacen ruido. Al 100 % la terminal se bloquea hasta que rotes tu identidad con `macchanger -r wlan0mon`. Un comando mal tecleado no penaliza.
- **Pistas**: `hint` pide ayuda a Darlene a cambio de puntos y tiempo.
- **Mapa de red**: muestra los cinco nodos y cuáles están ya comprometidos.
- **IRC**: el equipo comenta tus avances y responde si le escribes.
- **Sniffer**: `tcpdump` abre un analizador de paquetes con las tramas capturadas.
- **Monitor**: `top` abre un monitor de procesos en vivo.
- **Dificultad**: fácil, normal o difícil; cambia la sensibilidad del IDS, la penalización por fallo y los tiempos para cada rango.

![Portal cautivo y analizador de paquetes](docs/capturas/03-evil-portal-sniffer.png)

Hay extras para quien explore: borrar huellas antes de salir mejora el rango, y el propio código de la página esconde una puerta trasera.

![Pantalla de victoria](docs/capturas/04-victoria.png)

## Pistas y lecciones por fase

Este juego no tiene *writeup*: la gracia está en encontrar el camino. Si te atascas, abre la fase y lee las pistas **de una en una**: cada una orienta un poco más que la anterior, pero ninguna da el comando ni la respuesta. Debajo está la lección defensiva: qué falló en ECorp y cómo se evita en un sistema real.

<details>
<summary><strong>Fase 1 · Brecha perimetral</strong> — el portal web de ECorp</summary>

**Pistas**
1. Antes de atacar nada, averigua qué ofrece el objetivo: qué puertos responden y qué servicio hay detrás de cada uno.
2. La dirección del portal lleva un parámetro. Piensa qué hace el servidor con ese valor cuando lo recibe.
3. Una herramienta automática puede comprobar si ese parámetro llega tal cual a la base de datos. Después, ve de lo general (qué bases hay) a lo concreto (qué tabla te interesa).

**Lección defensiva**
- Consultas **parametrizadas** o procedimientos almacenados: el valor del usuario nunca se concatena en la sentencia SQL.
- La cuenta con la que la web accede a la base de datos tiene solo los permisos que necesita (mínimo privilegio).
- Contraseñas guardadas con un hash lento y con sal (bcrypt, Argon2), nunca en claro ni con MD5.
- Un WAF y la monitorización de errores SQL detectan los intentos a tiempo.

</details>

<details>
<summary><strong>Fase 2 · Red interna</strong> — la impresora de la subred</summary>

**Pistas**
1. Ya estás dentro: lo primero es saber qué más hay en la subred.
2. Entre los equipos descubiertos hay uno que nadie vigila mucho. Mira qué servicios tiene abiertos.
3. Algunos servicios de transferencia de ficheros dejan entrar a cualquiera sin cuenta propia. Lo que encuentres ahí puede servir más adelante.

**Lección defensiva**
- Desactivar el acceso anónimo y los servicios que no se usan, también en impresoras y equipos «tontos».
- Segmentar la red: los periféricos en su propia VLAN, sin alcance a los servidores.
- Inventario de activos: no se puede proteger lo que no se sabe que existe.
- Usar protocolos cifrados (SFTP, FTPS) en lugar de FTP en claro.

</details>

<details>
<summary><strong>Fase 3 · Servidor de seguridad</strong> — el control de accesos</summary>

**Pistas**
1. Los recursos compartidos de Windows también se pueden listar. Pregunta al servidor qué comparte y con quién.
2. Entre lo compartido hay datos que no deberían estar ahí: algo relacionado con las tarjetas de acceso del personal.
3. Una tarjeta de proximidad sencilla solo transmite un identificador. Si lo conoces, piensa qué te impide fabricar otra igual.

**Lección defensiva**
- Revisar los permisos de los recursos compartidos: nada accesible para invitados o para «Todos».
- Las copias de seguridad y los volcados de configuración, cifrados y fuera de los recursos compartidos.
- Tarjetas con autenticación criptográfica (por ejemplo, MIFARE DESFire) en lugar de las de solo identificador, que se clonan en segundos.
- Doble factor en las puertas críticas: tarjeta y PIN, o biometría.

</details>

<details>
<summary><strong>Fase 4 · Despacho del CEO</strong> — la estación de trabajo</summary>

**Pistas**
1. El objetivo de la fase te dice qué usuario buscas. Falta su contraseña, y las personas eligen contraseñas muy predecibles.
2. Hay listas públicas con millones de contraseñas reales filtradas. Pruébalas contra el servicio de acceso remoto, pero recuerda que cada intento hace ruido.
3. Una vez dentro, hay un texto que parece aleatorio pero solo está **codificado**, no cifrado. Los caracteres que usa te dicen qué codificación es.

**Lección defensiva**
- Bloqueo o retardo tras varios intentos fallidos (fail2ban, políticas de bloqueo) y alertas al SOC.
- Acceso remoto con **claves** en lugar de contraseñas, o con doble factor; nunca expuesto sin VPN.
- Contraseñas largas comprobadas contra listas de contraseñas filtradas.
- Base64 no protege nada: los secretos se cifran o se guardan en un gestor de secretos.

</details>

<details>
<summary><strong>Fase 5 · Perímetro exterior</strong> — la alarma y la cámara IP</summary>

**Pistas**
1. Para entrar en la red del vigilante, primero necesitas sus credenciales. Piensa cómo lograr que te las escriba él mismo en una página que parezca legítima.
2. El sistema de alarma tiene una interfaz pensada para máquinas, no para personas. Comprueba si exige algo para aceptar órdenes.
3. Los dispositivos IoT salen de fábrica con usuario y contraseña conocidos. Pregúntate si alguien los cambió al instalar la cámara.

**Lección defensiva**
- Wi-Fi corporativa con WPA2/WPA3-Enterprise (802.1X) y formación para no introducir credenciales en portales inesperados.
- Detección de puntos de acceso falsos (WIPS).
- Toda API exige autenticación y autorización, también las «internas».
- Cambiar las credenciales por defecto al instalar cualquier dispositivo, actualizar su firmware y aislarlo en su propia red.

</details>

<details>
<summary><strong>Fase 6 · Cámara acorazada</strong> — el controlador de la bóveda</summary>

**Pistas**
1. El controlador tiene un puerto físico de mantenimiento. Averigua cómo se presenta en el sistema.
2. Para hablar con él necesitas un programa de terminal serie y los parámetros correctos de la conexión.
3. El código maestro no está en esta fase: lo conseguiste antes. Repasa lo que has ido encontrando.

**Lección defensiva**
- Los puertos de depuración y mantenimiento (serie, JTAG, USB) se deshabilitan o se protegen en producción.
- Seguridad física: el acceso a la sala y al hardware también es un control de seguridad.
- Un secreto que abre la bóveda no puede reutilizarse ni quedar guardado en otro sistema.
- Registro y alertas de cualquier acceso al controlador.

</details>

> **Para el profesorado:** las seis fases se pueden plantear como un ejercicio de *defensa*. Tras jugar, cada grupo redacta para una fase qué control habría cortado la cadena y cómo se habría detectado el ataque en los registros.

## Ejecutarlo en local

Requiere [Bun](https://bun.sh/) (o Node.js 20+ con npm).

```bash
git clone https://github.com/unfantasmaenelsistema/FsocietyCTF.git
cd FsocietyCTF
bun install
bun run dev
```

Abre `http://localhost:3000`. Otros comandos: `bun run build` (genera `dist/`) y `bun run lint` (comprueba los tipos).

Añadir `?dev` a la URL muestra atajos para saltar de fase, útiles al desarrollar.

## Estructura

```text
├── public/assets/      Fondos de cada fase, emblema, logo y música ambiente
├── src/App.tsx         Máquina de estados del juego, intérprete de comandos e interfaz
├── src/index.css       Estética (manga oscuro, CRT, temas de terminal)
├── src/layout.css      Distribución: barra superior y ventanas
└── .github/workflows/  Publicación en GitHub Pages
```

## Licencia

[Apache 2.0](LICENSE). Un proyecto de [Un Fantasma en el Sistema](https://www.unfantasmaenelsistema.com/).
