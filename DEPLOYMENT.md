# Deployment — PocketBase Backend

Este documento explica cómo levantar el backend de PocketBase, tanto en local (Windows) como en un entorno de nube (GitHub Codespaces / Linux).

## Requisitos previos

- No se necesita instalar nada globalmente: PocketBase es un binario único, sin dependencias.
- Ve a [github.com/pocketbase/pocketbase/releases](https://github.com/pocketbase/pocketbase/releases) para confirmar cuál es la versión más reciente y ajusta el número en los comandos de abajo si hace falta.

## Windows (local)

1. Descarga el binario para Windows:

   ```
   curl -L -o pocketbase.zip https://github.com/pocketbase/pocketbase/releases/download/v0.28.4/pocketbase_0.28.4_windows_amd64.zip
   ```

2. Descomprime el `.zip` — esto extrae `pocketbase.exe`.

3. Corre el servidor:

   ```
   .\pocketbase.exe serve
   ```

   > Nota: `.\` es la sintaxis correcta en PowerShell/cmd. No uses `./` aquí — eso es para Linux/bash.

4. El dashboard queda disponible en `http://127.0.0.1:8090/_/`.

## Linux / GitHub Codespaces (nube)

1. Descarga el binario para Linux (no uses el `.exe` de Windows, no correrá aquí):

   ```
   curl -L -o pocketbase.zip https://github.com/pocketbase/pocketbase/releases/download/v0.28.4/pocketbase_0.28.4_linux_amd64.zip
   ```

2. Descomprime:

   ```
   unzip pocketbase.zip -d .
   ```

   Salida esperada:

   ```
   Archive:  pocketbase.zip
     inflating: ./CHANGELOG.md
     inflating: ./LICENSE.md
     inflating: ./pocketbase
   ```

3. Dale permisos de ejecución (los binarios descargados en Linux no son ejecutables por defecto):

   ```
   chmod +x pocketbase
   ```

4. Levanta el servidor:

   ```
   ./pocketbase serve
   ```

### Después de `serve`

- **Primera vez / resetear acceso:** crea o actualiza el superusuario desde otra terminal (sin detener el servidor):

  ```
  ./pocketbase superuser upsert tuemail@ejemplo.com tupassword123
  ```

- **Hacer accesible el puerto desde fuera del Codespace:** en VS Code, pestaña **PORTS** → click derecho sobre el puerto `8090` → **Port Visibility** → **Public**. Sin esto, cualquier fetch externo (por ejemplo desde un frontend en tu máquina local) recibe una página de login de GitHub en vez de la respuesta de la API.

- La URL pública tiene esta forma:

  ```
  https://<nombre-del-codespace>-8090.app.github.dev
  ```

## Conectar un frontend externo (ej. Astro local)

En el proyecto del frontend, define la URL de PocketBase como variable de entorno — nunca hardcodeada en el código:

```
# .env
PUBLIC_POCKETBASE_URL=https://<nombre-del-codespace>-8090.app.github.dev
```

```ts
// src/lib/pocketbase.ts
export const pb = new PocketBase(import.meta.env.PUBLIC_POCKETBASE_URL);
```

Así, en producción solo hay que cambiar el valor de esa variable en el panel del hosting (Vercel, Netlify, etc.) — sin tocar código ni crear ramas separadas para "local" vs "online".

## Qué NO se sube al repositorio

Estos archivos deben estar en `.gitignore`:

| Archivo/carpeta   | Por qué se ignora                                                |
|--------------------|-------------------------------------------------------------------|
| `pocketbase`       | Binario compilado para Linux — pesado y específico del SO         |
| `pocketbase.exe`   | Binario compilado para Windows — mismo motivo                     |
| `pocketbase.zip`   | Archivo de descarga temporal, no es código fuente                 |
| `pb_data/`         | Base de datos en tiempo de ejecución — contiene datos, no esquema |

Lo que sí vale la pena versionar:

- `pb_hooks/` — lógica personalizada en JavaScript
- `pb_migrations/` — definición del esquema de las colecciones
- Este mismo `DEPLOYMENT.md`

## Checklist rápido

- [ ] Binario correcto descargado (Linux en Codespaces, Windows en local)
- [ ] `chmod +x pocketbase` (solo Linux)
- [ ] `./pocketbase serve` corriendo
- [ ] Superusuario creado o actualizado
- [ ] Puerto 8090 en modo **Public** (solo Codespaces)
- [ ] `.env` del frontend apuntando a la URL correcta
- [ ] `.gitignore` cubre binarios y `pb_data/`
y listo te deberia funcionar

