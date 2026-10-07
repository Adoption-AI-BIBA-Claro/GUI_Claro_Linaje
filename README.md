<div align="center">

# 🔴 Claro Linaje Setup

### Instalador de un clic para el entorno de **linaje de datos ODI** de Claro Colombia

`ClaroLinajeSetup.exe` · Windows 10/11 · sin Python previo · sin cuenta de Docker

![Windows](https://img.shields.io/badge/Windows-10%2F11-0078D6?logo=windows&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.12%20auto--instalado-3776AB?logo=python&logoColor=white)
![Docker](https://img.shields.io/badge/Docker%20Desktop-sin%20login-2496ED?logo=docker&logoColor=white)
![Neo4j](https://img.shields.io/badge/Neo4j-grafo%20t%C3%A9cnico-008CC1?logo=neo4j&logoColor=white)
![Obsidian](https://img.shields.io/badge/Obsidian-grafo%20funcional-7C3AED?logo=obsidian&logoColor=white)
![Claude](https://img.shields.io/badge/Claude%20Desktop-MCP-D97757?logo=anthropic&logoColor=white)

</div>

---

## 📌 ¿Qué es esto?

Un **único ejecutable** que deja listo, en cualquier PC Windows de la compañía, el ecosistema completo
para preguntarle a **Claude Desktop** por el linaje de datos de los procesos ODI de Claro:

```
  Logs ODI (.txt)  ─┐                          ┌─▶  Neo4j   ──┐
  Mapas F→D (.xlsx) ─┼─▶  parsers del repo  ──┤               ├─▶  MCP  ─▶  Claude Desktop
  Arquitectura (.docx)┘                        └─▶  Obsidian ──┘
```

El usuario final **solo descarga el `.exe`, lo abre y pulsa «Iniciar todo»**. El instalador detecta qué falta en el equipo,
lo instala, clona el proyecto, levanta los contenedores, carga el grafo, publica las notas en Obsidian,
registra los MCPs en Claude y abre Claude listo para consultar.

---

## 🗂️ Contenido de esta carpeta

| Archivo | Para qué sirve |
|---|---|
| `setup_gui.py` | Código fuente de la aplicación (CustomTkinter). ~4.800 líneas. |
| `ClaroLinajeSetup.spec` | Receta de PyInstaller: un solo archivo, sin consola, con icono. |
| `ClaroLinajeSetup.ico` | Icono «CL» de la aplicación. |
| `build_exe.bat` | Doble clic → recompila el ejecutable en `dist\`. |
| `dist\ClaroLinajeSetup.exe` | **Ejecutable final** (≈15 MB). Es lo único que se distribuye. |
| `dist_actualizado\` | Copia del último build entregado. |
| `logs\` | Logs de los MCPs copiados de Claude para diagnóstico. |

---

## 🚀 Uso (usuario final)

1. Copia `ClaroLinajeSetup.exe` a cualquier carpeta del PC (p. ej. Escritorio).
2. Ábrelo. La **primera vez** aparece un panel *"Primera ejecución: analizando el equipo…"* con barra de progreso
   mientras detecta Python, Claude, CodeMie, Docker/WSL2 y Obsidian. Desaparece solo al terminar.
3. Pulsa **«Iniciar todo»** en el Dashboard y acepta los avisos de Windows (UAC) cuando aparezcan.
4. Si el equipo no tenía WSL2, la app pedirá **reiniciar Windows**. Al volver, abre la app y pulsa «Iniciar todo» otra vez.
5. Si el proyecto no está en el equipo, aparece el diálogo **«Acceso al repositorio»**: introduce la contraseña de despliegue que te entregaron.
6. Al finalizar se abre **Claude Desktop**. El prompt del proyecto queda en el portapapeles: pégalo en
   *Claude → Settings → Custom Instructions* (solo la primera vez).

> ⏱️ En un PC limpio el proceso completo tarda entre 10 y 25 minutos (descarga de Docker Desktop, imágenes y Python).

---

## ⚙️ Lo que hace «Iniciar todo», paso a paso

| # | Etapa | Detalle |
|---|---|---|
| 1 | **Python 3.10+** | Si falta → `winget install Python.Python.3.12` e instala `neo4j`, `openpyxl`, `mcp<2`, `httpx`, `python-docx`… en ese Python. Detecta el intérprete **real** (ignora el alias de la Microsoft Store). |
| 2 | **Claude Desktop** | Si falta → `winget install Anthropic.Claude`; sin winget, descarga el instalador oficial y lo ejecuta en silencio. |
| 3 | **CodeMie Connect** | Verifica que esté instalado y con sesión; si la sesión caducó hace `proxy stop → profile login → proxy start → proxy connect desktop`. |
| 4 | **Repositorio** | Busca `claro_linaje` en `Documents\Claro Linaje`, junto al exe y carpetas vecinas. Si no existe, instala Git y clona con el deploy token (diálogo de credenciales). Nunca abre el diálogo de credenciales de Windows. |
| 5 | **Claude MCP** | Escribe `claro-odi-lineage` y `obsidian` en **todos** los `claude_desktop_config.json` (`%LOCALAPPDATA%\Claude-3p`, `%APPDATA%\Claude`, Store), conservando lo que ya hubiera. Reinicia Claude si estaba abierto. Esto ocurre **antes** de Docker: Claude queda configurado aunque Docker falle. |
| 6 | **WSL2** | `wsl --status`. Si ya está operativo, **no hace nada** (verificado: `WSL2 detectado: WSL2 operativo`). Si falta → `wsl --install --no-distribution` **elevado** (UAC) y pide reiniciar solo si Windows lo exige. Si está en v1 → `--set-default-version 2`. Si falta el kernel → `wsl --update`. |
| 7 | **Docker Desktop** | Si falta → `winget install Docker.DockerDesktop`. Lo arranca y espera al daemon. **No requiere cuenta ni login.** |
| 8 | **Contenedores** | `docker compose up -d` (Neo4j `odi-lineage-neo4j`, MCP server, Inspector). Recupera contenedores viejos y conflictos de puertos. Contraseña fija de Neo4j: `change_me_1234`. |
| 9 | **Grafo Neo4j** | Si el grafo está vacío ejecuta `import.cypher` (logs ODI) y `excel_import.cypher` (mapas fuente-destino). Si ya hay datos, lo omite. La carpeta `import` va montada en el contenedor: no copia nada. |
| 10 | **Obsidian** | Instala Obsidian si falta, abre el vault `Documents\Obsidian Vault`, instala los plugins `obsidian-local-rest-api` y `mcp-tools-istefox` en `<vault>\.obsidian\plugins`, lee la API key del plugin, espera a que la REST API responda (hasta 45 s, probando `http://27123` y `https://27124`) y corre `docs_to_obsidian.py` para publicar `Linaje/Servicios`, `Tablas`, `Sistemas` e índice. |
| 11 | **Claude listo** | Copia el prompt de `prompt_claro_linaje.md` al portapapeles y a `Documents\Claro Linaje\prompt_claude_desktop.txt`; relanza Claude Desktop. |

Cada bloque final es independiente: si uno falla, se registra en el log y los demás continúan.

---

## 🧠 ¿Qué se puede preguntar a Claude después?

| Objetivo | Pregunta de prueba |
|---|---|
| Neo4j responde | *¿Cuántos nodos y relaciones tiene el grafo de linaje? Usa get_service_status.* |
| Obsidian responde | *Lee la nota Linaje/00_Indice.md y dime qué servicios están documentados.* |
| Cruce de ambas fuentes | *¿Qué hace el proceso 0480 y qué tablas lee y escribe?* |
| Linaje de tablas | *¿Qué procesos producen la tabla CONTRACT_ALL y quién la consume después?* |
| Linaje campo a campo | *¿De qué campo fuente viene MONTO y qué regla de negocio lo transforma? ¿Es mapping certificado o heurístico?* |
| Impacto | *Si cambia CUSTOMER_ID en la tabla fuente, ¿qué tablas y campos destino se ven afectados?* |

Estado verificado en el último despliegue: **17.064 nodos / 11.692 campos** en Neo4j y notas publicadas en el vault; ambos MCPs atendiendo peticiones en los logs de Claude.

---

## ⚠️ Fallas y limitaciones conocidas

### 🔴 Importantes

| Problema | Detalle | Mitigación |
|---|---|---|
| **El MCP de Neo4j falla si Docker no está arriba** | `mcp_server` del repositorio se conecta a Neo4j al arrancar; sin contenedor muere con `ServiceUnavailable` y Claude lo marca como *failed*. | Pulsar «Iniciar todo» (o arrancar Docker) y reiniciar Claude. Es comportamiento del repo, no de la app. |
| **El proxy de CodeMie no sobrevive al reinicio** | Tras reiniciar Windows, Claude muestra `ERR_CONNECTION_REFUSED 127.0.0.1:4001` hasta que algo relance el proxy. | Pulsar «Iniciar todo» lo relanza. Pendiente: arrancar el proxy al abrir la app. |
| **CodeMie Connect no se instala solo** | Existe `install_codemie_connect_if_missing()` pero `preflight` todavía devuelve el bloqueo `MSG_NO_CODEMIE` sin llamarlo. | Instalar CodeMie Connect a mano. Pendiente de conectar en `preflight`. |
| **Custom Instructions son manuales** | Claude Desktop no expone ningún archivo para escribirlas. | La app deja el prompt en el portapapeles y avisa. |

### 🟡 Menores

| Problema | Detalle |
|---|---|
| La URL de clone con token queda en `.git/config` del repo clonado (comportamiento estándar de `git clone https://user:token@…`). | Solo lectura y de un único repo; rotar el token si se filtra. |
| El icono «CL» puede verse genérico tras recompilar muchas veces. | Caché de iconos de Windows: `ie4uinit -show` o copiar el exe con otro nombre. |
| El aviso naranja de Claude *"Cowork requires the modern installer"*. | Función ajena (Cowork); no afecta a los MCPs. Cerrar con ✕. |
| El `README.md` del repositorio `claro_linaje` sugiere `npx -y mcp-obsidian` para Obsidian. | Ese paquete npm ignora la API key; la app usa `mcp_server/obsidian_bridge.py`, que sí funciona. Conviene corregir el README del repo. |
| `build_exe.bat` depende de que `python` en PATH tenga `pyinstaller`, `customtkinter`, `tkinterdnd2` y `neo4j`. | Instalarlos con `python -m pip install …` antes de compilar. |
| Primer arranque de WSL2 **exige reiniciar** Windows y aceptar UAC. | Inevitable a nivel de Windows; la app lo guía con un diálogo y puede programar el reinicio en 60 s. |

### 🔒 Seguridad

- El deploy token **ya no va incrustado** en el ejecutable: se pide en un diálogo (campo *Usuario* decorativo, *Contraseña* = token) y se valida con `git ls-remote` antes de clonar. No se guarda ni se muestra en logs.
- La API key de Obsidian y la contraseña de Neo4j se guardan en `.setup_gui_settings.json` / `obsidian.env` junto al exe, y se enmascaran en el log.
- Las conexiones a `127.0.0.1` (Obsidian HTTPS autofirmado) no verifican TLS; las descargas de internet sí.

---

## 🛠️ Compilar el ejecutable

```bat
:: requisitos (una sola vez)
python -m pip install pyinstaller customtkinter tkinterdnd2 neo4j

:: compilar → dist\ClaroLinajeSetup.exe
build_exe.bat
```

o directamente:

```bat
python -m PyInstaller --clean --noconfirm ClaroLinajeSetup.spec
```

---

## 📍 Rutas que usa la aplicación

| Qué | Dónde |
|---|---|
| Proyecto clonado | `%USERPROFILE%\Documents\Claro Linaje\claro_linaje` |
| Vault de Obsidian | `%USERPROFILE%\Documents\Obsidian Vault` |
| Plugins de Obsidian | `<vault>\.obsidian\plugins\{obsidian-local-rest-api, mcp-tools-istefox}` |
| Config de Claude (CodeMie) | `%LOCALAPPDATA%\Claude-3p\claude_desktop_config.json` |
| Config de Claude (clásico) | `%APPDATA%\Claude\claude_desktop_config.json` |
| Logs de los MCPs en Claude | `%LOCALAPPDATA%\Claude-3p\logs\mcp-server-*.log` |
| Prompt para Custom Instructions | `%USERPROFILE%\Documents\Claro Linaje\prompt_claude_desktop.txt` |
| Neo4j Browser / Bolt | `http://localhost:7475` · `bolt://localhost:7688` |
| Obsidian REST API | `https://127.0.0.1:27124` (o `http://127.0.0.1:27123`) |
| Proxy CodeMie | `http://127.0.0.1:4001` |

---

<div align="center">

Hecho para el equipo **BIBA · Claro** — cualquier incidencia, adjuntar el log de la app y `%LOCALAPPDATA%\Claude-3p\logs\mcp.log`.

</div>
