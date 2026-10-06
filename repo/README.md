# Observatorio de Conectividad — Antioquia

Repositorio del tablero público de conectividad de la Dirección de Gestión
Territorial TIC.

```
.github/workflows/publicar.yml   construcción y publicación automática
observatorio/                    el tablero de mapa
ficha/                           la ficha municipal detallada
```

El sitio se arma solo: **todos los días a las 6:00 a. m.** baja el Excel de
SharePoint, recalcula los cuatro ejes y vuelve a publicar.

---

## Paso a paso para dejarlo andando

### 1. Crear el repositorio

En GitHub: **New repository**. Nombre sugerido `observatorio-conectividad`.
Sin README, sin .gitignore (este paquete ya los trae).

### 2. Subir el contenido

```bash
cd carpeta-donde-descomprimió
git init
git add .
git commit -m "Observatorio de conectividad"
git branch -M main
git remote add origin https://github.com/<usuario>/<repositorio>.git
git push -u origin main
```

Si prefiere no usar la terminal: **Add file → Upload files** y arrastre las
carpetas. Funciona igual.

### 3. Activar GitHub Pages

**Settings → Pages → Source: `GitHub Actions`**. No elija «Deploy from a
branch»; el flujo de este repositorio publica por la otra vía.

### 4. Primera construcción

**Actions → Publicar observatorio → Run workflow**.

Tarda unos tres minutos. Al terminar, el sitio queda en
`https://<usuario>.github.io/<repositorio>/`.

En esta primera corrida todavía usa el Excel que viene en el repositorio: la
conexión con SharePoint se configura en el paso siguiente.

### 5. Conectar SharePoint

Dos caminos. Empiece por el primero; si la organización lo bloquea, vaya al
segundo.

#### Camino A — Enlace directo (cinco minutos, sin pedir permisos)

1. En SharePoint, sobre el Excel: **Compartir → Cualquier persona con el
   vínculo → Puede ver → Copiar vínculo**.
2. A la URL copiada, cámbiele el final `?web=1` por `?download=1`.
   Si no termina en `?web=1`, agréguele `&download=1`.
3. En GitHub: **Settings → Secrets and variables → Actions**
   - Pestaña **Variables** → New: `SP_MODO` = `enlace`
   - Pestaña **Secrets** → New: `SP_ENLACE` = la URL completa
4. **Actions → Run workflow** y revise el log. Debe decir
   `Fuente: descargado de SharePoint (NNN KB)`.

Si el log dice *«Lo descargado no es un Excel»*, la organización no permite
enlaces anónimos. Pase al camino B.

#### Camino B — Aplicación registrada (lo correcto para producción)

Escríbale a TI. Puede copiar esto:

> Buen día. Necesitamos registrar una aplicación en Entra ID, de **solo
> lectura**, para que un proceso automatizado descargue un archivo de una
> biblioteca de SharePoint una vez al día.
>
> - Permiso de aplicación: **`Sites.Selected`**, concedido únicamente sobre el
>   sitio `<nombre del sitio>`
> - No requiere acceso a correo, calendario ni a otros sitios
> - Necesitamos: **Tenant ID**, **Client ID** y un **Client Secret**
>
> El proceso corre en GitHub Actions y las credenciales quedan guardadas como
> secretos cifrados del repositorio.

Pedir `Sites.Selected` y no `Files.Read.All` importa: limita el acceso a un
solo sitio en vez de a toda la organización, y por eso es mucho más fácil que
lo aprueben.

Cuando le respondan, configure:

| Pestaña | Nombre | Valor de ejemplo |
|---|---|---|
| Variables | `SP_MODO` | `graph` |
| Variables | `SP_HOST` | `antioquia.sharepoint.com` |
| Variables | `SP_SITIO` | `/sites/PlaneacionTIC` |
| Variables | `SP_RUTA` | `/Documentos compartidos/General/sabana_unica_visor.xlsx` |
| Secrets | `SP_TENANT_ID` | |
| Secrets | `SP_CLIENT_ID` | |
| Secrets | `SP_CLIENT_SECRET` | |

`SP_RUTA` es la ruta dentro de la biblioteca, no la URL del navegador.

### 6. Comprobar que se actualiza

Cambie algo en el Excel de SharePoint, lance el flujo a mano y vea que la cifra
cambió en el sitio. De ahí en adelante corre solo cada madrugada.

---

## Cómo trabajar de aquí en adelante

Usted **sigue actualizando el Excel en SharePoint como siempre**. No toca el
repositorio para nada.

Lo único que recordar: el tablero muestra el dato de la última construcción,
no el de este segundo. Si necesita que un cambio salga ya, entre a **Actions →
Run workflow** y en tres minutos está publicado.

---

## Advertencias

**El sitio es público.** GitHub Pages en un repositorio gratuito no tiene
control de acceso: cualquiera con el enlace ve el tablero. Si la Dirección
considera que estos datos no deben ser abiertos, resuélvalo antes de publicar.
Alternativa gratuita con control de acceso: Cloudflare Pages con Cloudflare
Access, que sirve la misma carpeta `sitio/`.

**Si falla la descarga, el sitio no se cae.** El build usa la copia del Excel
que está en el repositorio y lo advierte en el log. Un tablero con el dato de
ayer sirve; uno que no compila, no. Conviene revisar los logs de vez en cuando
para no quedarse con datos viejos sin darse cuenta.

**El componente de inversión está apagado** hasta que lleguen los datos. Está
oculto por completo, no en ceros: un cero se lee como «no se invirtió nada»,
que es distinto de «no tenemos el dato».

---

## Probar en el computador antes de publicar

```bash
cd observatorio
pip install -r requirements.txt
python publicar.py --ficha ../ficha
python -m http.server 8000 --directory sitio
```

Abra `http://localhost:8000`.

Documentación detallada: [observatorio/DESPLIEGUE.md](observatorio/DESPLIEGUE.md)
