# Auditoria de migracion de Oposiciones Navaranda a Moodle App 5.2.1

Fecha: 2026-10-06. Alcance: inventario del origen y seguimiento del traslado a Moodle App 5.2.1.

## Estado del traslado

- Aplicados: identidad y sitio permitido en `moodle.config.json`; identidad nativa en `config.xml` sin volver a la version 4.5; tema, logotipos, favicon, iconos de finalizacion y actividad heredada; imagenes nativas Android/iOS y ficheros Firebase activos del origen. El splash Android 12 usa el PNG de marca y el splash iOS conserva las variantes de origen.
- Conservar deliberadamente: ajustes de SDK, permisos, version y User-Agent de 5.2.1. No trasladar `UIBackgroundModes` vacio ni el plist adicional sin uso: requieren comprobar push y la configuracion Firebase nativa antes de modificar los modos de segundo plano.
- Verificado localmente: configuracion JSON y Cordova XML coherentes, rutas y dimensiones de imagenes nativas declaradas, integridad de archivos copiados, y `git diff --check`. **Pendiente**: build web y builds nativos/pruebas en dispositivos; el destino no tiene instaladas las dependencias npm.

## Procedencia y metodo

- Origen: `../oposicionesnavaranda`, rama `master`, commit `607bdb5` (08-09-2025); destino: este repositorio, rama `latest`, tag `v5.2.1`, commit `66f26caf4`. Ambos arboles estaban limpios al auditar.
- El origen no comparte una historia incremental con este repositorio: `3105644` es un commit raiz con 4.826 archivos. Se compararon hashes y contenidos de su estado final con el tag upstream `v4.5.0` (4.557 archivos), ademas de revisar sus ocho commits posteriores. `package.json` del origen declara version 4.5.0; `moodle.config.json` y `config.xml` contienen ajustes posteriores de version 4.5.1.
- No se detectaron diferencias en archivos TypeScript entre el estado final de la marca y el tag `v4.5.0`; los cambios funcionales identificados se expresan en configuracion, recursos y estilos.
- La comparacion historica separa cambios de marca del salto 4.5 -> 5.2.1. No usar un diff completo entre las dos carpetas como lista de cambios propios, ni aplicar el commit raiz o los recursos antiguos en bloque.

## Cambios a trasladar

| Prioridad | Superficie en el destino | Evidencia en el origen | Accion propuesta |
| --- | --- | --- | --- |
| Alta | `moodle.config.json` / plantilla de despliegue | `app_id=com.oposicionesnavaranda.mobile`, `appname=OposicionesNavaranda`, `customurlscheme=oposicionesnavaranda`, sitio `https://aulavirtual.oposicionesnavaranda.es`, `sitename=Oposiciones Navaranda`, `onlyallowlistedsites=true`, `default_lang=es`, `forcedefaultlanguage=true`, `demo_sites={}`, politica de privacidad `https://oposicionesnavaranda.es/politica-de-privacidad/`, `notificoncolor=#3ca4b6` y enlaces de tienda Android e iOS (`id6743129735`). | Reaplicar sobre la configuracion 5.2.1 conservando claves y valores nuevos; documentar cuales deben ser parametrizables por entorno. Probar idioma, privacidad, acceso solo al sitio autorizado y enlaces profundos. No publicar credenciales ni tokens. |
| Alta | `config.xml` | ID `com.oposicionesnavaranda.mobile`, nombre, descripcion y autor propios; version 4.5.1, `android-versionCode=45100`, `versionCode=45003`, `ios-CFBundleVersion=4.5.1.0` y `AppendUserAgent=MoodleMobile 4.5.0 (45003)`. | Adaptar identidad y metadatos al esquema 5.2.1; decidir version y numeros de compilacion nuevos antes de publicar. **No copiar** literalmente los numeros ni el User-Agent 4.5. |
| Alta | `google-services.json`, `GoogleService-Info.plist` y configuracion Cordova | Firebase Android/iOS propio; sucesivas sustituciones/renombrados de plist en `22489c4`, `f4d5b89`, `c34aa20`, `fca5d00` y `607bdb5`. El ultimo commit deja `GoogleService-Info-ANDROID.plist` adicional. | Confirmar con el propietario de Firebase los ficheros vigentes de cada plataforma, bundle ID, project ID, push, esquemas OAuth y firmas; integrar solo los necesarios para 5.2.1, sin copiar historiales ni volcar valores sensibles a esta auditoria. |
| Alta | `resources/`, `src/assets/icon/`, `src/assets/img/` | Iconos Android/iOS, splash, `resources/icon.png`, `resources/splash.png`, `resources/android/icon-foreground.png`, `src/assets/icon/icon.png`, `favicon.ico`, `src/assets/img/login_logo.png` y `top_logo.png` propios. | Reutilizar fuentes graficas de marca y generar los tamanos/formato que exija el pipeline 5.2.1; comprobar icono, splash, login y cabecera en Android/iOS. Evitar copiar variantes generadas obsoletas sin comprobar que siguen en uso. |
| Media | `src/theme/globals.variables.scss` | `$blue` y `$cyan` pasan a `#3ca4b6`, `$green` a `#2a3f21`, `$brand-color` a `$blue`; dashboard logo y menu siempre visibles, progreso de curso y selector de seccion ocultos. | Adaptar colores y opciones funcionales por separado: `$core-dashboard-logo` ya no existe en 5.2.1; configurar `showTopLogo` (el destino usa `hidden`; `offline` muestra `top_logo.png` local en Inicio). Comprobar contrastes y navegacion. |
| Media | `src/theme/theme.light.scss` | Cabecera con fondo `var(--primary)` y texto `var(--white)` en lugar de blanco y color de texto normal. | Ajustar la cabecera del tema claro en 5.2.1; verificar botones, barras de estado, contraste y tema oscuro antes de decidir si este ultimo tambien debe personalizarse. |
| Media | `src/assets/img/completion/*.svg`, `src/assets/img/mod_legacy/*.svg` | Diferencias de contenido frente a `v4.5.0` en seis iconos de finalizacion y numerosos iconos de actividad. | Revisar visualmente los SVG y sus usos actuales antes de migrar: los de finalizacion siguen referenciados en 5.2.1, pero `mod_legacy` solo se selecciona para sitios Moodle anteriores a 4.0; para un sitio actual comprobar primero `src/assets/img/mod/`. |
| Media | `config.xml`, `resources/values/colors.xml` | Iconos adaptativos Android y splash por densidad; para iOS, iconos/splash por resolucion y `UIBackgroundModes` vacio. El XML de colores es identico en origen y destino. | Reconciliar declaraciones Cordova con el pipeline 5.2.1 (splash Android 12, edge-to-edge, SDK objetivo 36 e iOS minimo 15); conservar el XML de colores del destino y verificar que desactivar modos en segundo plano no rompe notificaciones u otras funciones requeridas. |

## Diferencias que no deben migrarse directamente

- El `.gitignore` del origen fue reemplazado por una plantilla generica: se perdieron exclusiones de `/platforms`, `/plugins`, `node_modules`, idiomas/entorno generados, etc. Conservar el del destino y reforzar solo exclusiones necesarias.
- `src/assets/lang/*.json` y otros resultados de generacion del volcado inicial no equivalen a traducciones propias; regenerar con el pipeline actual. `licenses.json`, `package-lock.json` y `upgrade.txt` son artefactos o informacion ligada a la version antigua, no personalizaciones que deban sobreescribirse.
- No incorporar ficheros `*:Zone.Identifier`, recursos obsoletos o parches/dependencias de Angular 17 y de la configuracion Cordova de Moodle 4.5 a la aplicacion Angular 20 / Moodle 5.2.1.
- `0d5106c` anadio temporalmente un segundo sitio, pero `607bdb5` lo retiro. El estado final tiene **un sitio** permitido; no restaurar la prueba de dos sitios sin una decision de producto.
- `moodle.config.example.json` coincide con upstream `v4.5.0`; la configuracion particular esta en `moodle.config.json`, fichero **versionado en ambos repositorios** y leido directamente por el build. Se ha adaptado el archivo del destino manteniendo sus claves nuevas, sin alterar el ejemplo generico.
- `enableonboarding=true` aparece en el origen, pero no es una clave de la configuracion 5.2.1: verificar si el flujo actual la reemplaza; no recuperar la opcion antigua a ciegas.

## Orden y criterios de aceptacion

1. Acordar identificadores finales, version/build numbers, politica de sitios y propiedad de Firebase/App Store/Play Store; decidir si el tema oscuro necesita la misma identidad visual.
2. Integrar configuracion de marca y `config.xml` sobre los ficheros de 5.2.1, preservando sus claves, plugins y permisos actuales. Revisar el cambio de `UIBackgroundModes` con responsables de push.
3. Incorporar fuentes de imagen y tema compatible, regenerar iconos/splash y revisar los SVG de actividades/finalizacion en el flujo de uso real.
4. Validar build web y builds nativas Android/iOS; en dispositivos, verificar instalacion/actualizacion con el ID anterior, conexion al aula virtual, bloqueo de otros sitios, enlaces profundos, login, notificaciones y navegacion con tema claro/oscuro. Registrar versiones, firmas y credenciales de distribucion fuera del repositorio.

## Riesgos y decisiones abiertas

- Los numeros de version del origen no son coherentes entre `config.xml`, configuracion y User-Agent; definir los nuevos conforme a las versiones ya publicadas para evitar rechazos de tienda o fallos de actualizacion.
- Los ficheros Firebase del origen y su historial necesitan comprobacion de vigencia y tratamiento de seguridad; no asumir que el plist con sufijo `ANDROID` corresponde a Android ni que la ultima sustitucion contiene los datos de iOS correctos.
- El plist iOS activo del origen tiene el bundle ID esperado, pero su proyecto Firebase es diferente del de `google-services.json`; el plist adicional comparte proyecto con Android, aunque Cordova no lo utiliza. Validar con el responsable de Firebase si esa separacion sigue siendo intencionada antes de publicar o probar push.
- Los splash antiguos no equivalen al splash de arranque Android 12; los iconos `mod_legacy` no personalizan las actividades de un sitio Moodle 4.4 o posterior. Verificar los recursos efectivos antes de copiar. La auditoria no demuestra aun que los builds nativos funcionen.
