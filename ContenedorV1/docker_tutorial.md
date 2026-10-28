# 🐳 Docker desde cero

**Guía para quien nunca ha usado Docker.**
Teoría de la Computación · Revisión 2 · octubre de 2026 · gabrielhuav

---

## Cómo está organizada esta guía

Las partes **1 a 8** se leen en orden y no se saltan. Cada una usa sólo lo que ya se
explicó antes. Al terminar la parte 8 se puede trabajar el semestre entero sin saber
nada más.

El **anexo** del final tiene todo lo demás: `run`, `--build`, `EXPOSE`, puertos, varios
servicios, perfiles, limpieza. **No hace falta para empezar.** Está ahí para cuando
aparezca la duda, no para leerlo de corrido.

| Parte | Qué se aprende |
|---|---|
| 1 | Qué problema resuelve Docker |
| 2 | Instalarlo y probar que funciona |
| 3 | Tu primer contenedor, a mano |
| 4 | Prender, entrar y apagar con un archivo |
| 5 | Que tus archivos no se pierdan |
| 6 | Construir tu propia imagen |
| 7 | El entorno de tu curso, pieza por pieza |
| 8 | Qué hacer cuando algo falla |

---
---

# Parte 1 · Qué problema resuelve

## «En mi computadora sí funciona» 😅

```
👨‍💻 Tú:        «Mi programa funciona perfecto.»
👩‍💻 Tu equipo: «En la mía no arranca.»
👨‍🏫 Profesor:  «A mí tampoco.»
```

Nadie miente. Lo que pasa es que las tres computadoras son distintas:

- Tú tienes Python 3.12; tu compañera, Python 3.9.
- Tú instalaste una biblioteca hace meses y ya no te acuerdas.
- El profesor tiene Linux y tú Windows.
- Una ruta que en tu máquina existe, en la otra no.

El resultado es que **el trabajo no se puede repetir**, y un resultado que no se puede
repetir no sirve para entregarlo.

## La analogía del barco 🚢

Antes de los contenedores, un barco se cargaba con los bultos sueltos:

```
🍎 Manzanas sueltas     → se mezclan con lo demás
🐟 Pescado fresco       → contamina todo
📱 Electrónicos         → se mojan
```

Con contenedores, cada mercancía va en su caja metálica:

```
📦 Contenedor A: sólo manzanas
📦 Contenedor B: sólo pescado, refrigerado
📦 Contenedor C: sólo electrónicos, secos
```

La caja siempre mide lo mismo, así que **cualquier barco, camión o grúa del mundo la
puede mover sin saber qué lleva dentro**.

Docker hace eso con el software. Tu programa viaja dentro de una caja que además lleva
**todo lo que necesita**: el sistema, el intérprete, las bibliotecas y la configuración.
Quien reciba la caja la abre y funciona, sin instalar nada.

## Las dos palabras que hay que aprender ahora

Por ahora **sólo dos**. Las demás vienen después, cuando hagan falta.

### 📸 Imagen — la receta

Es una plantilla congelada: «Python 3.12 con Flet y pytest ya instalados». No se
ejecuta y no cambia. Es como una receta de cocina escrita en papel.

### 📦 Contenedor — el plato ya hecho

Es una imagen **puesta a funcionar**. De la misma receta puedes hacer tres pizzas; de la
misma imagen puedes tener tres contenedores corriendo a la vez.

```
📄 Receta   ──────►  🍕 Pizza
📸 Imagen   ──────►  📦 Contenedor
```

Lo importante de esta distinción: **la imagen se construye una vez y no cambia; el
contenedor se crea, se usa y se tira.** Eso último asusta al principio, y la parte 5 lo
resuelve.

---
---

# Parte 2 · Instalarlo y probar que funciona

## Instalar

### Windows 🪟

1. Necesitas Windows 10 u 11 de 64 bits, con la virtualización habilitada en el BIOS y
   al menos 4 GB de RAM.
2. Descarga **Docker Desktop** de <https://www.docker.com/products/docker-desktop>.
3. Ejecuta el instalador como administrador. Deja marcada la opción **«Use WSL 2»**.
4. Reinicia la computadora cuando termine.

> ⚠️ **Docker Desktop tiene que estar abierto** para que los comandos funcionen. Si lo
> cierras, nada de lo que sigue responde. Es el tropiezo más común de la primera semana.

### macOS 🍎

1. Descarga Docker Desktop del mismo sitio. **Ojo con elegir bien**: hay una versión
   para Intel y otra para Apple Silicon, y no son intercambiables.
2. Arrastra `Docker.app` a Aplicaciones y ábrelo.

### Linux 🐧 (Ubuntu / Debian)

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
                    docker-buildx-plugin docker-compose-plugin
```

Y para no tener que escribir `sudo` cada vez:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

## Probar que quedó bien

Abre una terminal —en Windows, PowerShell o el Símbolo del sistema— y escribe:

```bash
docker --version
docker compose version
```

Debe responder algo como `Docker version 27.x` y `Docker Compose version v2.x`.

> **Es `docker compose`, con espacio.** El `docker-compose` con guion es la versión
> antigua. En instalaciones nuevas sigue funcionando como atajo, pero está
> descontinuada: acostúmbrate al espacio.

## Tu primer comando de verdad

```bash
docker run hello-world
```

**Qué acaba de pasar, paso a paso:**

1. Docker buscó la imagen `hello-world` en tu computadora. No la encontró.
2. La descargó de Docker Hub, que es el catálogo público de imágenes.
3. Creó un contenedor a partir de esa imagen.
4. Lo ejecutó. El contenedor imprimió un mensaje.
5. El programa terminó, así que el contenedor se detuvo.

Si ves `Hello from Docker!`, está instalado y funcionando. **Ese mensaje es todo lo que
ese contenedor sabía hacer.**

---
---

# Parte 3 · Tu primer contenedor de verdad

Ahora uno con el que se puede jugar:

```bash
docker run -it debian:trixie-slim bash
```

La terminal cambia y aparece algo así:

```
root@3f2a1b9c4d5e:/#
```

**Estás dentro de un Debian recién hecho**, que no existía hace tres segundos. Prueba
estas cosas:

```bash
ls /                 # el sistema de archivos es otro, no el tuyo
cat /etc/os-release  # es Debian, aunque tú estés en Windows
whoami               # aquí dentro eres root
python3 --version    # error: no está instalado
docker ps            # error: Docker no existe aquí dentro
```

Esos dos últimos errores son **la demostración del aislamiento**:

- No hay Python porque esa imagen no lo trae. Lo que no está en la receta, no está.
- **No hay Docker** porque el contenedor no sabe que está dentro de un contenedor. No ve
  tu Windows, ni tu disco, ni tus programas.

Ahora sal:

```bash
exit
```

Vuelves a tu terminal de siempre.

## ¿Y dónde quedó ese contenedor?

No desapareció:

```bash
docker ps        # los que están CORRIENDO ahora mismo
docker ps -a     # TODOS, incluso los apagados
```

El segundo lo muestra con el estado `Exited`. Quedó ahí, apagado, ocupando espacio. Para
borrarlo:

```bash
docker rm <el-nombre-o-el-id-que-salió>
```

**Primera lección incómoda:** cada `docker run` crea un contenedor **nuevo**. Si lo
ejecutas diez veces, tienes diez contenedores apagados acumulándose. En la parte 4
dejamos de hacerlo así.

## Lo que se escriba dentro, se pierde

Una prueba más, que importa. Entra otra vez:

```bash
docker run -it debian:trixie-slim bash
```

Dentro:

```bash
echo "trabajo importante" > mi_archivo.txt
ls
exit
```

Y entra de nuevo:

```bash
docker run -it debian:trixie-slim bash
ls
```

**`mi_archivo.txt` no está.** El segundo `run` creó un contenedor nuevo y limpio a partir
de la misma imagen; el anterior —y tu archivo— se quedaron aparte.

Esto asusta, y es normal. La parte 5 lo resuelve.

---
---

# Parte 4 · Prender, entrar y apagar

## Por qué un archivo en vez de comandos

El comando de la parte 3 era corto. Uno real no lo es:

```bash
docker run -d -it --name mi_entorno -v "C:\proyecto":/trabajo \
  -w /trabajo -p 8550:8550 mi-imagen:1.0
```

Nadie escribe eso cada mañana, ni se acuerda de lo que puso ayer. Por eso existe **Docker
Compose**: se escribe una vez en un archivo de texto, se guarda junto al proyecto, y a
partir de ahí los comandos son de tres palabras.

## El archivo más simple que existe

Crea una carpeta para tu proyecto y dentro un archivo llamado `compose.yaml`:

```yaml
services:
  entorno:
    image: debian:trixie-slim
    container_name: mi_entorno
    stdin_open: true
    tty: true
```

Línea por línea:

| Línea | Qué dice |
|---|---|
| `services:` | «aquí empieza la lista de contenedores» |
| `entorno:` | el nombre que le doy yo a este servicio. Podría ser cualquiera |
| `image:` | de qué imagen sale |
| `container_name:` | nombre legible. Sin esto, Docker inventa uno feo |
| `stdin_open` y `tty` | **lo mantienen encendido**, esperando órdenes |

> Sobre `stdin_open` y `tty`: sin ellas el contenedor arrancaría, vería que no tiene nada
> que hacer y se apagaría solo. Una base de datos no las necesita, porque la base de
> datos ya se queda corriendo por su cuenta. Un entorno de trabajo sí.

## ¿`compose.yaml` o `docker-compose.yml`?

Verás las dos formas por internet. **Hacen exactamente lo mismo**; la diferencia es sólo
histórica:

| Nombre | De cuándo es |
|---|---|
| `docker-compose.yml` | El clásico. Era obligatorio cuando Compose era un programa aparte, escrito en Python, que se ejecutaba con guion: `docker-compose`. |
| `compose.yaml` | **El actual.** Es el nombre que recomienda la especificación abierta *Compose Specification*, que es la que sigue Docker desde Compose V2. |

Al escribir `docker compose up`, Docker busca el archivo en este orden y se queda con el
primero que encuentre:

```
1. compose.yaml          ← el que usamos
2. compose.yml
3. docker-compose.yaml
4. docker-compose.yml
```

Con cualquier otro nombre hay que decírselo con `-f`:

```bash
docker compose -f mi-archivo.yaml up -d
```

> **No pongas los dos en la misma carpeta.** Docker se quedaría con `compose.yaml` y el
> otro no se usaría nunca, lo que produce el desconcierto de «edité el archivo y no pasa
> nada».

## Los tres comandos

Desde la carpeta donde está el `compose.yaml`:

```bash
docker compose up -d                  # 1 · PRENDER
docker compose exec entorno bash      # 2 · ENTRAR
docker compose stop                   # 3 · APAGAR
```

Eso es todo. **Con esos tres se trabaja un semestre entero.**

- `up -d` lo prende y lo deja corriendo en segundo plano. La `-d` es de *detached*: te
  devuelve el prompt en vez de quedarse enganchada.
- `exec entorno bash` te mete dentro. `entorno` es el nombre del servicio del archivo.
- `stop` lo apaga. Sigue existiendo, apagado, listo para volver.

Para volver a prender el mismo:

```bash
docker compose start
```

Y para ver cómo está:

```bash
docker compose ps        # ¿está encendido?
docker compose ps -a     # ¿existe, aunque esté apagado?
```

## Si escribiste `up` sin la `-d`

Es lo primero que le pasa a todo el mundo. La terminal muestra esto y parece colgada:

```
 ✔ Container teoria_computacion  Created
Attaching to teoria_computacion
```

**No está colgada, y no falló nada.** `up` sin `-d` se queda *enganchado* a la salida del
contenedor, para mostrarte lo que imprima. Como el contenedor lo único que hace es
esperar órdenes en silencio, no imprime nada y no ves más.

Tienes dos salidas:

**Opción A — dejarla ahí.** Abre **otra** ventana de terminal, ve a la misma carpeta y
entra con `exec`:

```bash
cd "C:\Users\gabri\...\Contenedor"
docker compose exec teoria bash
```

**Opción B — hacerlo bien desde el principio** (recomendado):

```
Ctrl + C                      detiene el proceso enganchado
docker compose up -d          prende y te devuelve el prompt
docker compose exec teoria bash
```

La `-d` es de *detached*: «déjalo corriendo por detrás y devuélveme la terminal».

## Salir sin apagar

Dentro del contenedor, `exit` te devuelve a Windows **pero no lo apaga**. Puedes volver a
entrar con `exec` las veces que quieras. Sólo `stop` lo apaga.

> ⚠️ **Los comandos de Docker se escriben FUERA del contenedor.** Si escribes
> `docker compose stop` estando dentro, sale `bash: docker: command not found`. Está bien
> que así sea: es la prueba del aislamiento de la parte 3. Primero `exit`, después el
> comando.

## `stop` y `down` no son lo mismo

Ésta confunde a todo el mundo:

| Comando | Qué le pasa al contenedor | ¿Sigue en Docker Desktop? |
|---|---|---|
| `docker compose stop` | Se **apaga**. Como el interruptor de la luz. | Sí, apagado |
| `docker compose start` | Se vuelve a prender, el mismo. | Sí, encendido |
| `docker compose down` | Se apaga **y se borra**. | **No, desaparece** |

Si le das `down` y el contenedor desaparece de la pestaña *Containers* de Docker Desktop,
**no hiciste nada malo**: eso es lo que hace. Y no perdiste nada: la imagen sigue
guardada y `up -d` vuelve a crear el contenedor idéntico en dos segundos.

**Para el día a día, usa `stop`.** Deja `down` para cuando quieras dejar limpio.

---
---

# Parte 5 · Que tus archivos no se pierdan

Al final de la parte 3 escribimos un archivo dentro del contenedor y desapareció. Vamos a
arreglarlo.

## La idea: una ventana entre las dos carpetas

Un **volumen** conecta una carpeta de tu computadora con una carpeta de dentro del
contenedor. Lo que escribe uno lo ve el otro, al instante, en los dos sentidos.

```
Tu computadora                     Dentro del contenedor
────────────────────               ────────────────────
C:\proyecto\          ═══════════►  /trabajo/
  programa.py                         programa.py
  datos.csv                           datos.csv
```

### La analogía

```
📦 Contenedor = la cocina donde preparas la comida
💾 Volumen    = el refrigerador, que está en otra habitación

Puedes desmontar la cocina entera. El refrigerador sigue ahí.
```

## Cómo se escribe

Se añade `volumes:` al `compose.yaml`:

```yaml
services:
  entorno:
    image: debian:trixie-slim
    container_name: mi_entorno
    volumes:
      - ./:/trabajo
    working_dir: /trabajo
    stdin_open: true
    tty: true
```

La línea clave se lee **de izquierda a derecha**, separada por dos puntos:

```
- ./:/trabajo
   │      └──── dónde aparece DENTRO del contenedor
   └─────────── qué carpeta TUYA se comparte ( ./ = ésta misma )
```

Y `working_dir: /trabajo` hace que al entrar ya estés ahí, sin tener que hacer `cd`.

## La comprobación que lo deja claro

```bash
docker compose up -d
docker compose exec entorno bash
```

Dentro:

```bash
ls
```

**Debe mostrar exactamente lo mismo que el Explorador de Windows en esa carpeta**,
incluido el propio `compose.yaml`. Si muestra otra cosa, la terminal se abrió en otra
carpeta.

Ahora crea un archivo desde dentro:

```bash
echo "hola desde el contenedor" > prueba.txt
exit
```

Mira tu carpeta en Windows: **`prueba.txt` está ahí**. Y sobrevive a `stop`, a `down` y a
borrar la imagen, porque nunca estuvo dentro del contenedor: estuvo siempre en tu disco.

## ¿Entonces el contenedor ve todo mi disco?

No. **Sólo esa carpeta.** El resto de tu computadora sigue siendo invisible desde dentro.
El volumen es una ventana que abres tú, a propósito, y sólo donde tú la pones.

## Dos tipos de volumen, y cuál borra `-v`

Aquí hay una trampa que conviene conocer desde ya. Hay dos formas de escribir un volumen:

```yaml
volumes:
  - ./datos:/var/lib/datos      # ① carpeta tuya  (bind mount)
  - mis_datos:/var/lib/datos    # ② volumen de Docker (con nombre)
```

La diferencia importa por un comando: `docker compose down -v`.

| | ① carpeta tuya `./datos` | ② volumen con nombre `mis_datos` |
|---|---|---|
| **Dónde está** | En tu proyecto, la ves | La administra Docker, no la ves |
| **¿La borra `down -v`?** | **No.** Sigue ahí | **Sí.** Desaparece |
| **Hay que ponerla en `.gitignore`** | Sí | No hace falta |

**La regla corta:**

> Si la ruta empieza por `.` o por `/` → es una carpeta tuya, y `-v` **no la toca**.
> Si es un nombre a secas → es un volumen de Docker, y `-v` **sí lo borra**.

Mucha gente cree que `down -v` le va a borrar su carpeta de datos. Si es un bind mount,
no. Y al revés: si usas un volumen con nombre y le das `-v`, los datos se van de verdad.

---
---

# Parte 6 · Construir tu propia imagen

Hasta aquí hemos usado `debian:trixie-slim`, que alguien más construyó. Pero esa imagen
no trae Python, ni Flet, ni pytest. Para eso hay que construir la propia, y eso se hace
con un **Dockerfile**.

## Cuándo hace falta y cuándo no

**No hace falta** si existe una imagen pública con lo que necesitas. Para una base de
datos, `postgres:17` ya está lista y no hay nada que construir.

**Sí hace falta** cuando necesitas varias herramientas juntas que nadie ha empaquetado,
que es justo el caso de un entorno de curso.

## El Dockerfile mínimo

Crea un archivo llamado `Dockerfile` —sin extensión, con esa D mayúscula— junto al
`compose.yaml`:

```dockerfile
FROM python:3.12-slim-trixie

RUN apt-get update && apt-get install -y --no-install-recommends git nano \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir "flet[web]==0.86.5" pytest==8.3.4

WORKDIR /trabajo
CMD ["bash"]
```

Son cuatro tipos de instrucción y cada una hace una cosa:

| Instrucción | Qué hace |
|---|---|
| **`FROM`** | De qué imagen se parte. Siempre la primera línea. |
| **`RUN`** | Ejecuta un comando **mientras se construye**. Aquí, instalar programas. |
| **`WORKDIR`** | La carpeta en la que se entra al arrancar. |
| **`CMD`** | Qué se ejecuta al arrancar el contenedor. |

Hay otras dos que verás a menudo:

| Instrucción | Qué hace |
|---|---|
| **`COPY`** | Mete un archivo tuyo dentro de la imagen. |
| **`ENV`** | Fija una variable de entorno dentro del contenedor. |

## ⚠️ La regla más importante de todo Docker: nunca `latest`

```dockerfile
FROM python:latest               # ❌
FROM python:3.12-slim-trixie     # ✅
```

`latest` **no significa «estable»**. Significa «la última que subieron». Lo que pasa:

```
Septiembre: construyes el proyecto   → funciona
Diciembre:  tu compañero lo clona    → la imagen cambió → ya no levanta
Enero:      el profesor lo califica  → tampoco
```

Poner la versión —`python:3.12-slim-trixie`, `postgres:17`, `debian:trixie-slim`— es lo que
hace que tu trabajo **siga funcionando dentro de un año**. En los cursos donde se usa esta
guía, es requisito de entrega.

## Si el Dockerfile ya lo construye todo, ¿qué es el compose.yaml?

Es la duda más común al ver los dos archivos juntos, y se responde con una analogía:

| | Qué es | Qué dice |
|---|---|---|
| **`Dockerfile`** | El **plano de fabricación** | **QUÉ** lleva dentro: qué sistema, qué programas se instalan, qué carpetas se crean. Su resultado es una imagen «congelada». |
| **`compose.yaml`** | El **manual de arranque** | **CÓMO** se pone a correr esa imagen en tu computadora: qué carpeta se comparte, qué variables se fijan, cómo se llama el contenedor. |

El `Dockerfile` se usa **una vez**, al construir. El `compose.yaml` se usa **cada día**,
al prender.

Sin el `compose.yaml` tendrías que escribir esto cada mañana:

```bash
docker run -it --name teoria_computacion -v "C:\Users\...\practica2-tc:/trabajo" \
  -w /trabajo -p 8550:8550 teoria-computacion:2027-1
```

El `compose.yaml` guarda todo eso para que sean tres palabras. Y en proyectos reales hace
algo más: levanta varios contenedores a la vez —una base de datos, una página web y un
servidor— con un solo comando.

## Enganchar el Dockerfile al compose

En el `compose.yaml` se cambia **una sola clave**: `image:` por `build:`.

```yaml
services:
  entorno:
    build: .                    # ← en vez de image:
    image: mi-entorno:1.0       # el nombre que tendrá la imagen construida
    container_name: mi_entorno
    volumes:
      - ./:/trabajo
    working_dir: /trabajo
    stdin_open: true
    tty: true
```

`build: .` quiere decir «construye con el Dockerfile de esta carpeta».

## ¿`build:` o `image:`? Las dos formas de tener una imagen

Aquí hay una duda muy razonable: *si las imágenes se descargan de Docker Hub, ¿qué hace
`image:` en un compose que además construye?*

Hay **dos maneras** de que Docker consiga una imagen:

| En el compose | Qué hace Docker |
|---|---|
| Sólo `image: postgres:17` | **La descarga** de Docker Hub, ya hecha por otra persona. |
| `build: .` **e** `image: mi-entorno:1.0` | **La construye** con tu Dockerfile y, al terminar, **la guarda en tu computadora con ese nombre**. |

Es decir: cuando aparecen las dos, `image:` **no es de dónde se baja, es cómo se va a
llamar la tuya**.

Y ojo: tu imagen tampoco sale de la nada. El `FROM python:3.12-slim-trixie` del Dockerfile
sí descarga una imagen de Docker Hub; lo que tú construyes es esa base **más** lo que le
instalaste encima.

```
Docker Hub                    Tu computadora
──────────                    ──────────────
python:3.12-slim-trixie  ──►  + git, Flet, pytest       ──►  teoria-computacion:2027-1
   (la descarga)                 (lo del Dockerfile)           (queda guardada aquí)
```

Para verla:

```bash
docker image ls
```

### Cómo se lee ese nombre

Las imágenes se nombran `nombre:etiqueta`. En `teoria-computacion:2027-1`:

| Parte | Qué significa |
|---|---|
| `teoria-computacion` | La unidad de aprendizaje: Teoría de la Computación |
| `:2027-1` | La **etiqueta**, que aquí es el semestre: 2027-1 |

El nombre lo eliges tú. Sirve para que tu imagen quede identificada y no se confunda con
las demás que tengas en la máquina. Poner la etiqueta del semestre además permite tener
la del semestre pasado y la de éste sin pisarse.

## Construir y usar

```bash
docker compose up -d --build      # construye la imagen y prende
docker compose exec entorno bash  # entra
```

### Qué dice la salida de la construcción

La primera vez salen muchas líneas. Vale la pena saber leer tres cosas:

```
 => CACHED [2/4] RUN apt-get update && apt-get install -y ... git     0.0s
 => [3/4] RUN pip install --no-cache-dir "flet[web]==0.86.5" ...   12.4s
 => exporting to image                                              7.7s
 => => naming to docker.io/library/teoria-computacion:2027-1        0.0s
```

- **`[2/4]`, `[3/4]`** — en qué paso del Dockerfile va, de cuántos.
- **`CACHED`** — «esta instrucción no cambió desde la última vez, me la salto». Por eso
  la primera construcción tarda unos minutos y la siguiente dos segundos. Si cambias
  una línea del Dockerfile, esa y **todas las de abajo** se vuelven a ejecutar; las de
  arriba siguen en caché. Por eso conviene poner lo que menos cambia al principio.
- **`naming to ... teoria-computacion:2027-1`** — la imagen terminada quedó guardada con ese
  nombre, el que pusiste en `image:`.

La primera vez tarda varios minutos, porque descarga e instala todo. A partir de ahí **no
vuelve a hacerlo**: la imagen queda guardada y prender tarda dos segundos.

Sólo hay que volver a poner `--build` cuando **cambies el Dockerfile**. El resto del
semestre, `docker compose up -d` a secas.

---
---

# Parte 7 · El entorno de tu curso, pieza por pieza

Aquí está un entorno real y completo: el de Teoría de la Computación, con Python, Flet
para las interfaces gráficas y pytest para las pruebas. Los autómatas se programan en
Python puro.

## ✅ El mínimo: dos archivos

**Con dos archivos basta para levantar el entorno.** Está comprobado: se construye y
funciona. Todo lo demás es opcional.

```
mi-proyecto/
  Dockerfile        ← qué lleva dentro
  compose.yaml      ← cómo se usa
```

### `Dockerfile` — qué lleva dentro

```dockerfile
FROM python:3.12-slim-trixie

# Para versionar y editar sin salir del contenedor
RUN apt-get update && apt-get install -y --no-install-recommends \
        git nano \
    && rm -rf /var/lib/apt/lists/*

# Dos avisos de pip que dentro de un contenedor no aplican
ENV PIP_ROOT_USER_ACTION=ignore \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# Flet para la interfaz gráfica y pytest para las pruebas, con la versión fijada.
# No se instala ninguna biblioteca de autómatas.
RUN pip install --no-cache-dir \
        "flet[web]==0.86.5" \
        pytest==8.3.4

# Acentos, hora de la Ciudad de México, y Flet como página web
ENV LANG=C.UTF-8 \
    TZ=America/Mexico_City \
    FLET_FORCE_WEB_SERVER=true \
    FLET_SERVER_PORT=8550

WORKDIR /trabajo
CMD ["bash"]
```

La primera línea se lee así:

| Parte | Qué significa |
|---|---|
| `python` | La imagen oficial de Python |
| `3.12` | La versión del intérprete, la de referencia del curso |
| `slim` | Sin compiladores ni documentación: pesa mucho menos |
| `trixie` | El Debian que lleva debajo, Debian 13 |

Fijar también `trixie` evita que el sistema de debajo cambie cuando salga el siguiente
Debian. Es la regla de la parte 6, llevada un paso más allá.

Qué se instala y para qué:

- `git` y `nano` → para versionar o editar sin salir del contenedor.
- `flet[web]` → las interfaces gráficas de las prácticas, servidas como página web.
- `pytest` → las pruebas automatizadas.

Nada más. La imagen terminada pesa unos 510 MB.

### ¿Y las bibliotecas de autómatas?

No se instala ninguna, a propósito. Hay bibliotecas de Python que ya simulan autómatas,
convierten un AFN en AFD o dibujan diagramas, pero en este curso esos algoritmos son
justamente lo que se aprende a programar: la simulación, la cerradura λ, la construcción
de subconjuntos o la eliminación de estados se escriben en Python puro.

El ejemplo de `extras/ejemplos/` muestra que no hace falta más: lee un archivo de JFLAP
con la biblioteca estándar de Python y simula el autómata en unas cuantas líneas.

Las dos instrucciones que parecen raras tienen su motivo:

- `PIP_ROOT_USER_ACTION` y `PIP_DISABLE_PIP_VERSION_CHECK` callan dos avisos de pip: que
  se está instalando como root, y que hay una versión nueva de pip. Dentro de un
  contenedor ninguno de los dos aplica, pero salen en amarillo y se leen como si fueran
  errores.
- `FLET_FORCE_WEB_SERVER` y `FLET_SERVER_PORT` hacen que Flet sirva la interfaz como
  página web en el puerto 8550, en lugar de intentar abrir una ventana, que dentro del
  contenedor no existe.

### `compose.yaml` — cómo se usa

```yaml
name: teoria-computacion

services:
  teoria:
    build: .                        # construye con el Dockerfile de esta carpeta
    image: teoria-computacion:2027-1
    container_name: teoria_computacion
    volumes:
      - ./:/trabajo                 # esta carpeta, vista desde dentro
    working_dir: /trabajo
    ports:
      - "127.0.0.1:8550:8550"       # la interfaz de Flet: http://localhost:8550
    stdin_open: true                # las dos mantienen el contenedor
    tty: true                       #   encendido, esperando órdenes
```

Casi todo ya lo conoces de las partes 4 y 5. Hay dos líneas nuevas:

- **`name:`** es el nombre del proyecto. Sin él, Compose usa el nombre de la carpeta, que
  cambia de una computadora a otra y acaba produciendo contenedores duplicados.
- **`ports:`** abre una puerta entre tu computadora y el contenedor, y es lo que te deja
  ver la interfaz en tu navegador. El `127.0.0.1` del principio hace que esa puerta sólo
  se abra en tu computadora; sin él, cualquiera en tu misma red podría entrar. El detalle
  está en el anexo A.3.

### Cómo se usa

```bash
docker compose up -d --build         # la primera vez
docker compose exec teoria bash      # entrar a trabajar
bash extras/comprobar.sh             # ya dentro: comprobar que todo responde
docker compose stop                  # apagar
```

La comprobación revisa Python, pytest, el volumen y Flet, y simula el autómata del
ejemplo. Si todo está bien, termina con **«Entorno listo.»**

Y dentro, se trabaja normal:

```bash
python src/app.py                           # la interfaz: abre http://localhost:8550
pytest                                      # las pruebas
python extras/ejemplos/afd.py 0110 000      # simular el AFD de ejemplo
```

> **La interfaz no te devuelve el prompt.** Mientras está abierta, la terminal se queda
> ocupada sirviéndola. Para detenerla, `Ctrl + C`. Si quieres seguir trabajando con ella
> abierta, abre **otra** terminal y entra con otro `docker compose exec teoria bash`.

## JFLAP y Tkinter no van dentro del contenedor

Los dos abren una ventana, y el contenedor no tiene pantalla.

**JFLAP** es una ventana de Java. Se instala y se usa en tu computadora, como siempre. Lo
que sí se comparte son los archivos: guarda los `.jff` en la carpeta del proyecto y tus
programas de Python los leen desde dentro del contenedor, porque es la misma carpeta
(parte 5).

**Tkinter** falla dentro del contenedor por dos razones. En la imagen `slim` ni siquiera
se importa:

```
ImportError: libtk8.6.so: cannot open shared object file: No such file or directory
```

Y aunque se instale la biblioteca que falta, no hay dónde abrir la ventana:

```
_tkinter.TclError: no display name and no $DISPLAY environment variable
```

Si eliges Tkinter para tu interfaz, ejecútala en tu computadora, con tu Python. El
contenedor te sigue sirviendo para el núcleo y las pruebas, que no necesitan ventana; por
eso conviene que la lógica del autómata viva en un archivo aparte de la interfaz. Flet sí
funciona dentro, porque se sirve como página web.

## Los archivos opcionales, y para qué sirve cada uno

Ninguno hace falta para que el entorno levante:

| Archivo | Para qué sirve | ¿Se puede omitir? |
|---|---|---|
| `extras/comprobar.sh` | Comprueba en menos de un minuto que todo responde. | Sí |
| `extras/ejemplos/` | El autómata M₁ de la sección 1.2.3 guardado como lo guarda JFLAP, el programa que lo simula y sus pruebas. Los usa la comprobación. | Sí |
| `.gitattributes` | Hace que Git guarde los scripts con finales de línea de Linux. Sin él, Git en Windows puede dejar inservible `comprobar.sh`. | Sí, pero conviene tenerlo |
| `.dockerignore` | Evita mandar el repositorio entero al construir. Sin él, tarda más. | Sí |
| `README.md` | La documentación para quien reciba el proyecto. | Sí, pero conviene tenerlo |

## Dónde van los archivos

**En la raíz del repositorio de la práctica**, junto a todo lo demás:

```
practica2-tc/                  ← aquí se abre la terminal
  Dockerfile
  compose.yaml
  .gitattributes
  .gitignore
  README.md

  automatas/
  src/
  tests/
```

Así, al hacer `ls` dentro del contenedor ves **exactamente lo mismo** que en el Explorador
de Windows, autómatas y código incluidos.

## Y el Dockerfile y el compose se versionan

Van al repositorio con el resto del código. **Son parte de la entrega**, no un accesorio:
son lo que permite que otra persona reconstruya tu trabajo.

---
---

# Parte 8 · Cuando algo falla

## Los dos comandos con los que empieza todo diagnóstico

**Antes de borrar nada, lee el mensaje de error.** Casi siempre lo dice con todas sus
letras.

```bash
docker compose ps -a     # 1 · ¿está encendido, apagado, o nunca arrancó?
docker compose logs      # 2 · ¿qué dijo antes de morir?
```

En ese orden. El 90 % de los problemas se resuelven leyendo la última línea de `logs`.

## Tabla de síntomas

| Lo que sale | Casi siempre es | Qué hacer |
|---|---|---|
| `Cannot connect to the Docker daemon` | Docker Desktop no está abierto | Abrirlo y esperar a que el icono deje de animarse |
| `bash: docker: command not found` | Escribiste un comando de Docker **dentro** del contenedor | `exit` primero, y escribirlo en la terminal de Windows |
| La terminal se queda colgada, sin prompt | Usaste `up` sin `-d`, o tienes una interfaz de Flet abierta | Ctrl+C, y después `docker compose up -d` |
| `service "..." is not running` al hacer `exec` | El contenedor está apagado | `docker compose up -d` primero |
| El contenedor arranca y se apaga solo | Faltan `stdin_open` y `tty` | `docker compose logs` y leer la última línea |
| `ls` muestra archivos que no esperabas | La terminal se abrió en otra carpeta | `cd` a la carpeta del `compose.yaml` |
| Los archivos que genero no aparecen | Se guardaron fuera del volumen | Guardar dentro de la carpeta montada, no en `/tmp` |
| `localhost:8550` no abre en el navegador | La interfaz no está corriendo | Entrar y ejecutar `python src/app.py`; dejar esa terminal abierta |
| `address already in use` al correr la interfaz | Ya hay otra interfaz abierta en el 8550 | Ctrl+C en la otra terminal, o `docker compose restart` |
| `port is already allocated` al hacer `up` | Otro programa de tu computadora usa el 8550 | Cambiar el número de en medio: `"127.0.0.1:8551:8550"`, y abrir `localhost:8551` |
| `The container name "/teoria_computacion" is already in use` | Tienes el contenedor de una versión anterior del entorno | En la carpeta de la versión anterior, `docker compose down`. Si ya no la tienes: `docker rm teoria_computacion`. Tus archivos no se tocan |
| `ImportError: libtk8.6.so` o `no display name and no $DISPLAY` | Intentaste abrir una interfaz de Tkinter dentro del contenedor | Ejecutarla en tu computadora, o usar Flet (parte 7) |
| Instalé algo con `pip` y desapareció | Se instaló en el contenedor, no en la imagen, y `down` lo borró | Añadirlo al `RUN pip install` del Dockerfile y `up -d --build` |
| `no such file or directory` al montar | La ruta del volumen no existe | Crear la carpeta, o revisar la ruta |
| `permission denied` al escribir (Linux) | El contenedor escribe como root | Añadir `user: "${UID}:${GID}"` al servicio |
| La construcción tarda muchísimo | Se está mandando todo el repositorio | Crear un `.dockerignore` (anexo A.6) |
| Se acabó el espacio en disco | Imágenes y contenedores viejos | `docker system df` y después `docker system prune` |

## Empezar de cero sin miedo

Si algo se enredó y quieres reiniciar:

```bash
docker compose down        # borra el contenedor
docker compose up -d       # lo crea de nuevo, limpio
```

**No pierdes nada**: tus archivos están en la carpeta montada y la imagen sigue
construida. Tarda dos segundos.

---
---
---

# 📎 ANEXO · Cuando ya te sientas cómodo

**Nada de esto hace falta para trabajar.** Está aquí para cuando aparezca la duda.

---

## A.1 · `exec` y `run` no son lo mismo

Los dos «te meten al contenedor», y por eso se confunden:

| | Qué hace |
|---|---|
| `docker compose exec teoria bash` | Entra al contenedor **que ya prendiste** con `up -d`. Es el de siempre. |
| `docker compose run teoria bash` | **Crea un contenedor nuevo**, aparte, cada vez. No entra al que prendiste. |

Por eso `run` parece funcionar «aunque el contenedor esté apagado»: no está apagado, es
que `run` fabricó otro. Si lo usas varias veces acabas con tres o cuatro contenedores
sueltos sin darte cuenta, y `docker ps -a` los muestra todos.

`run --rm` borra el contenedor al terminar, y sirve para un comando suelto sin dejar
rastro:

```bash
docker compose run --rm teoria python --version
```

**Para trabajar, quédate con `up -d` una vez y `exec` todas las que haga falta.**

---

## A.2 · `--build`: cuándo sirve y cuándo no

Es el comando que más se copia sin entender:

| Si tu servicio dice… | `--build`… |
|---|---|
| `image: postgres:17` | **no tiene nada que construir**. Se ejecuta, no falla, y no cambia nada. |
| `build: .` | **reconstruye tu imagen**. Hace falta cuando cambiaste el Dockerfile. |

---

## A.3 · `EXPOSE` no publica el puerto

La confusión más extendida sobre los Dockerfile:

| | Qué hace |
|---|---|
| `EXPOSE 8550` en el Dockerfile | **Documenta** que el programa de dentro escucha ahí. No abre nada. Se puede borrar sin que cambie el comportamiento. |
| `ports: - "127.0.0.1:8550:8550"` en el compose | **Esto sí publica.** Sin esta línea el contenedor corre pero no puedes llegar a él. |

**Regla:** `EXPOSE` lo escribe quien *publica* la imagen; `ports` lo escribe quien la
*usa*. Por eso el Dockerfile del curso no lleva `EXPOSE` y el compose sí lleva `ports`.

Y los puertos se leen **mi máquina : contenedor**:

```
"127.0.0.1:8551:8550"
     │       │     └── el de dentro. Lo fija FLET_SERVER_PORT, no se toca
     │       └──────── el tuyo. Lo eliges tú, y se cambia si está ocupado
     └──────────────── quién puede entrar: sólo tu computadora
```

Sin el `127.0.0.1` del principio, el puerto se publica en todas las conexiones de tu
computadora y cualquiera en tu misma red puede abrir tu interfaz.

---

## A.4 · Varios servicios en el mismo archivo

Un proyecto puede necesitar dos contenedores: una base de datos y una aplicación.

```yaml
services:
  web:
    image: nginx:1.27
    ports:
      - "8080:80"
    depends_on:
      - db

  db:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: postgres
    volumes:
      - pg_datos:/var/lib/postgresql/data

volumes:
  pg_datos:
```

`docker compose up -d` levanta **los dos**. `depends_on` dice cuál arranca primero. Y se
hablan entre ellos por el nombre del servicio: desde `web`, la base de datos está en
`db:5432`.

---

## A.5 · Perfiles: servicios que no arrancan solos

Si quieres un servicio que no arranque con `up -d`, se le pone un perfil:

```yaml
  notebook:
    profiles: ["notebook"]
```

Y se levanta pidiéndolo:

```bash
docker compose --profile notebook up -d
```

Alternativa más simple: ponerlo en un archivo aparte y usar `-f`.

```bash
docker compose -f compose.extra.yaml up -d
```

---

## A.6 · El archivo `.dockerignore`

Al construir, Docker copia **toda la carpeta** a su motor antes de empezar. Sin este
archivo eso incluye el historial de Git, los binarios y las imágenes, y la construcción
tarda mucho más.

Es el equivalente del `.gitignore`, pero para Docker:

```
.git
venv/
__pycache__/
*.pyc
.pytest_cache/
*.png
*.log
```

---

## A.7 · Capas, y por qué la imagen pesa tanto

Cada instrucción del Dockerfile crea una **capa**. Tres `RUN` seguidos pesan más que uno
solo con todo junto, por eso se escriben encadenados con `&&`.

```bash
docker history mi-imagen     # qué capa es la culpable del tamaño
docker system df             # cuánto espacio ocupa todo
```

Para adelgazar una imagen: usar la variante `-slim` o `alpine` de la base, agrupar los
`RUN`, y terminar con `rm -rf /var/lib/apt/lists/*`.

---

## A.8 · ¿Y el entorno virtual de Python?

En la imagen del curso **no hace falta**. El Python de `python:3.12-slim-trixie` no es el
de Debian: está instalado aparte, en `/usr/local`, y `pip install` funciona directo. Lo
que se instala ahí tampoco lo tapa el volumen, porque el volumen se monta en `/trabajo`.

Donde sí hace falta es en imágenes que parten de `debian` o de `ubuntu`. Debian 12 en
adelante y Ubuntu 24.04 protegen su Python, y un `pip install` directo falla con
`externally-managed-environment`. La solución correcta es un entorno virtual:

```dockerfile
RUN python3 -m venv /opt/venv
ENV PATH="/opt/venv/bin:${PATH}"
RUN pip install --no-cache-dir -r requirements.txt
```

Poner `/opt/venv/bin` al principio del `PATH` equivale a activarlo. Existe el atajo
`pip install --break-system-packages`, pero hace lo que su nombre dice.

---

## A.9 · Cómo se lee una imagen en Docker Hub

Antes de usar una imagen, mira cuatro cosas en `hub.docker.com`:

1. **¿Es oficial?** Las oficiales llevan la etiqueta *Docker Official Image*.
2. **¿Qué etiquetas ofrece?** La etiqueta **es** la versión.
3. **¿Qué variables de entorno espera?** Para `postgres`, `POSTGRES_PASSWORD` es
   obligatoria; sin ella el contenedor arranca y se muere.
4. **¿Dónde guarda sus datos?** Ése es el punto donde hay que montar el volumen.

---

## A.10 · Máquina virtual y contenedor

| | Máquina virtual | Contenedor |
|---|---|---|
| **Qué trae** | Un sistema operativo completo | Sólo el programa y sus dependencias |
| **Cuánto ocupa** | Gigabytes | Megabytes |
| **Cuánto tarda en arrancar** | Minutos | Segundos |
| **Qué comparte** | Nada | El núcleo de la máquina anfitriona |

**El matiz:** en Windows y macOS, Docker usa por debajo una máquina virtual ligera. Pero
es **una sola**, compartida por todos tus contenedores. En Linux no hay ninguna.

---

## A.11 · Comandos sueltos que conviene conocer

```bash
# Imágenes
docker image ls                 # las que tienes
docker pull nombre:version      # descargar una
docker rmi nombre               # borrar una
docker image inspect nombre     # ver su ficha

# Contenedores
docker logs -f nombre           # seguir su salida en vivo
docker cp nombre:/ruta ./aqui   # sacar un archivo de dentro
docker stats                    # cuánta memoria y CPU usan

# Volúmenes
docker volume ls
docker volume rm mis_datos      # esto SÍ destruye datos

# Limpieza
docker container prune          # borrar contenedores apagados
docker image prune              # borrar imágenes sin usar
docker system prune             # borrar todo lo no utilizado
```

> ⚠️ `docker system prune -a --volumes` borra **todos** los volúmenes con nombre que no
> esté usando ningún contenedor, de todos tus proyectos. Es el comando con el que hay que
> tener más cuidado. Tus carpetas no las toca.

---

## A.12 · Podman

Hace lo mismo que Docker, sin proceso central y sin privilegios de root. Los comandos son
casi idénticos: basta sustituir `docker` por `podman`. **Elige uno de los dos** y
documenta cuál usaste.

---

## 📚 Para seguir

- **Documentación oficial**: <https://docs.docker.com/>
- **Documentación de Flet**: <https://flet.dev/docs/>
- **Documentación de Tkinter**: <https://docs.python.org/3/library/tkinter.html>
- **Referencia del Dockerfile**: <https://docs.docker.com/reference/dockerfile/>
- **Play with Docker**: laboratorios interactivos en el navegador, sin instalar nada
- **Docker Hub**: <https://hub.docker.com/>

---

## 🎓 Las cinco reglas que no se negocian

1. **Nunca `latest`.** Fija siempre la versión.
2. **Lee el error antes de borrar nada.** `ps -a`, después `logs`.
3. **Lo que no se puede perder va en un volumen.** El contenedor es desechable.
4. **`stop` apaga; `down` borra el contenedor.** Ninguno de los dos toca tus archivos.
5. **El Dockerfile y el compose van en el repositorio.** Son parte del trabajo.

---

*«El futuro del desarrollo de software está en los contenedores. Docker te da
superpoderes para crear, compartir y ejecutar aplicaciones en cualquier lugar.»*
— gabrielhuav
