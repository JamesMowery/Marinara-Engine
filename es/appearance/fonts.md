# Fuentes personalizadas y Google Fonts

Esta guía muestra cómo cambiar la fuente que Marinara Engine usa en toda la app. Puedes elegir la fuente incorporada, añadir tus propios archivos de fuente o descargar una fuente de Google Fonts por su nombre.

## Elegir una fuente para la app

La opción de fuente está en **Settings** (Configuración), en la pestaña **Appearance** (Apariencia), dentro de la sección **Text & Scale** (Texto y escala).

1. Abre **Settings** y haz clic en la pestaña **Appearance**.
2. Busca la sección **Text & Scale**.
3. Abre el menú desplegable **Font** (Fuente).
4. Elige una fuente de la lista.

La opción predeterminada es **Default (Inter)**. Inter es una fuente limpia elegida para leer en pantalla. Cualquier fuente personalizada que añadas aparece en el mismo menú desplegable **Font**, debajo de la opción predeterminada.

Los campos de prompt del editor de presets y el Example Dialogue de un personaje usan una fuente monoespaciada mientras está seleccionada la fuente predeterminada. Si eliges cualquier otra fuente, también la usan. Otros campos de prompt, como la plantilla de prompt de un agente, siguen siendo monoespaciados.

Tu elección de fuente se sincroniza entre dispositivos. Cuando eliges una fuente, cada navegador y dispositivo conectado al mismo servidor de Marinara cambia a ella. Para saber cómo funciona esta sincronización, consulta la guía [Resumen de Settings](../settings/settings-overview.md).

## Añadir tus propias fuentes

Puedes añadir una fuente personalizada colocando un archivo de fuente en una carpeta del servidor. Esta es la máquina que ejecuta Marinara.

1. Busca la carpeta `data/fonts/` dentro de la carpeta de datos de Marinara en la máquina del servidor.
2. Copia tu archivo de fuente en esa carpeta.
3. Vuelve a **Settings**, luego **Appearance**, luego **Text & Scale**.
4. Abre el menú desplegable **Font**. Tu fuente ahora aparece en la lista.
5. Selecciónala.

Marinara lee estos tipos de archivo de fuente: `.ttf`, `.otf`, `.woff` y `.woff2`. Los archivos con cualquier otra extensión se ignoran.

Marinara crea un nombre visible a partir del nombre del archivo. Por ejemplo, un archivo llamado `OpenSans-Bold.ttf` aparece como "Open Sans". Así que nombra tus archivos de forma clara si quieres una lista ordenada.

Los archivos de fuente en la carpeta `data/fonts/` viven en el servidor. Cada dispositivo que se conecta al mismo servidor de Marinara puede usarlos. Tu elección de fuente también se sincroniza entre esos dispositivos, así que todos muestran la misma fuente.

## Descargar desde Google Fonts

Marinara puede obtener una fuente directamente de Google Fonts por ti. El servidor necesita acceso a internet para que esto funcione.

1. Abre **Settings**, luego **Appearance**, luego **Text & Scale**.
2. Busca el campo **Google Fonts**.
3. Escribe el nombre exacto de la fuente, por ejemplo `Fira Code` o `Lora`.
4. Haz clic en **Add** (Añadir).
5. Espera a que termine la descarga. La nueva fuente aparece entonces en el menú desplegable **Font**.

Escribe el nombre exactamente como lo escribe Google Fonts. El enlace **Browse fonts at fonts.google.com** (Explorar fuentes en fonts.google.com) está junto al campo. Abre el sitio de Google Fonts en una pestaña nueva para que puedas buscar nombres.

El nombre solo puede usar letras, números y espacios. Si descargas la misma fuente otra vez más adelante, Marinara reemplaza la copia antigua en lugar de crear un duplicado.

Si la descarga falla, lee el mensaje de error. Cuando Marinara no puede acceder a Google Fonts, te dice que revises tu conexión a internet. Cuando dice que la fuente no se encontró, hay dos causas posibles. El nombre puede no coincidir con una fuente en Google Fonts. O la fuente puede no tener un grosor normal (400), que es el estilo normal no negrita. Revisa la ortografía y comprueba en el sitio de Google Fonts que la fuente ofrezca un estilo Regular.

## Open Fonts Folder es solo local

Junto al menú desplegable **Font** hay un botón **Open Fonts Folder** (Abrir carpeta de fuentes). Abre la carpeta `data/fonts/` en el explorador de archivos de la máquina del servidor.

Este botón actúa sobre el servidor, no sobre el dispositivo desde el que estás viendo Marinara. Si ejecutas Marinara en tu propia computadora, te abre la carpeta. Si te conectas desde un teléfono o una segunda computadora, el botón no hace nada útil para ti. En ese caso, copia tú mismo tus archivos de fuente en la carpeta `data/fonts/` del servidor.

## Guías relacionadas

- [Ajustes de apariencia](appearance-settings.md)
