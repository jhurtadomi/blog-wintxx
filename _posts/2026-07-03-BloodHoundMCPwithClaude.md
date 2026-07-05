---
title: "BloodHound MCP: Integrando Claude con BloodHound"
date: 2026-07-03 12:00:00 -0500
categories: [Red Teaming, Active Directory]
tags: [BloodHound, MCP, Claude, AI, Active Directory, Red Team]
image: /assets/img/BloodHoundMCP/portada_mcp.png
---

## Introducción

Hace unas 2 semanas me enteré de la existencia de un servidor MCP (Model Context Protocol) para BloodHound, el cual nos permite integrarlo con Claude. A continuación, comparto las publicaciones que leí al respecto:

> **Artículo de SpecterOps:**
> [BloodHound MCP: One Year Later — What I Learned About MCPs, Models, and Context](https://specterops.io/blog/2026/06/18/bloodhound-mcp-one-year-later-what-i-learned-about-mcps-models-and-context/#h-introduction-the-first-version-worked-but-needed-improvement)
{: .prompt-info }

> **Repositorio de GitHub:**
> [BloodHound MCP: Repositorio de GitHub](https://github.com/mwnickerson/bloodhound_mcp)
{: .prompt-info }

### ¿Qué es esto?

El MCP de BloodHound permite consultar tus datos de Active Directory en lenguaje natural desde Claude Desktop. En lugar de escribir queries Cypher manualmente, le preguntas directamente a Claude cosas como "muéstrame todos los Domain Admins" o "encuentra usuarios kerberoasteables".

### Prerrequisitos

- Claude Desktop (plan Pro)
- BloodHound Community Edition corriendo localmente
- Datos ya importados en BH CE (output de RustHound o bloodhound-python)
- Git o acceso a GitHub para descargar el repo

> Cabe mencionar que también he visto a colegas utilizarlo con otros LLMs, así que estaré investigando un poco más y les compartiré si encuentro algo interesante.

## Implementación

## Paso 1: Clonar el repositorio

```powershell
git clone https://github.com/mwnickerson/bloodhound_mcp.git
cd bloodhound_mcp
```

## Paso 2: Generar el API Token en BH CE

En la UI de BloodHound:

```
My Profile → Authentication : Api Key Management → Create Token
```
![api_key](/assets/img/BloodHoundMCP/api_key.png)

![api_key2](/assets/img/BloodHoundMCP/api_key2.png)

![api_key3](/assets/img/BloodHoundMCP/api_key3.png)

Guarda los valores KEY & ID.

## Paso 3: Crear el archivo .env

Dentro de la carpeta del repositorio que clonamos, necesitamos crear un **.env**

```powershell
@"
BLOODHOUND_DOMAIN=localhost
BLOODHOUND_TOKEN_KEY=tu-token-key-aqui
BLOODHOUND_TOKEN_ID=tu-token-id-aqui
BLOODHOUND_PORT=8080
BLOODHOUND_SCHEME=http
"@ | Out-File -FilePath .env -Encoding utf8
```
> Coloca los valores que generaste en el **Paso 2**

![env](/assets/img/BloodHoundMCP/env.png)

## Paso 4: Instalar **uv**

En Powershell 

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

En Linux

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Con `uv` instalado, ahora necesitamos realizar la integración del MCP de BloodHound con Claude Desktop:

## Paso 5: Configurar Claude Desktop

Abrimos el Claude Desktop:

```
Settings → Developer → Edit Config
```

![claude](/assets/img/BloodHoundMCP/claude_desktop.png)

Esto nos abrirá los archivos de configuración y el que tocaremos especificamente será `claude_desktop_config.json` y agregamos el siguiente código:

```json
{
  "mcpServers": {
    "bloodhound_mcp": {
      "command": "uv",
      "args": [
        "--directory",
        "D:\\bloodhound_mcp",
        "run",
        "main.py"
      ]
    }
  }
}
```

![configjson](/assets/img/BloodHoundMCP/configjson.png)

> Asegurarse de en los `args` estar apuntando al directorio correcto donde tienen clonado el repositorio.


## Paso 6: Reiniciar Claude Desktop

Cierra Claude Desktop completamente desde la bandeja del sistema (click derecho → Quit, no solo la X) y vuelve a abrirlo.

![test](/assets/img/BloodHoundMCP/test.png)


## Pruebas Funcionales

Ahora es momento de probar su funcionamiento, lo primero que debemos hacer es subir un `.zip` generado por cualquier colector (RustHound, SharpHound, etc)

![upload_test](/assets/img/BloodHoundMCP/upload_test.png)

Ahora, desde Claude Desktop:

![test1](/assets/img/BloodHoundMCP/test1.png)
![test2](/assets/img/BloodHoundMCP/test2.png)
![test3](/assets/img/BloodHoundMCP/test3.png)
![test4](/assets/img/BloodHoundMCP/test4.png)
![test5](/assets/img/BloodHoundMCP/test5.png)


## Conclusiones

La integración de **BloodHound MCP** con Claude Desktop marca una gran diferencia en la velocidad con la que podemos interrogar a Neo4j durante una auditoría de Active Directory:

- **Abstracción de Cypher:** Elimina la necesidad de memorizar consultas complejas en Cypher para búsquedas rápidas.
- **Velocidad y Comodidad:** Consultar rutas de ataque o cuentas críticas mediante lenguaje natural acelera drásticamente la fase de análisis.
- **Criterio Profesional:** Recuerda que la IA es un copiloto. Siempre debes verificar manualmente las rutas sugeridas por Claude para descartar alucinaciones o interpretaciones erróneas.

## Referencias

- [BloodHound MCP - GitHub Repository](https://github.com/mwnickerson/bloodhound_mcp)
- [SpecterOps - BloodHound MCP: One Year Later](https://specterops.io/blog/2026/06/18/bloodhound-mcp-one-year-later-what-i-learned-about-mcps-models-and-context/)

---

Gracias por leer, espero que te sirva. Si tienes alguna duda, sugerencia, corrección o simplemente quieres compartir algo al respecto, tienes mis redes para contactarme.

Frase para finalizar:

> "Solo podemos ver un poco del futuro, pero lo suficiente para darnos cuenta de que hay mucho que hacer"
>
> — Alan Turing
