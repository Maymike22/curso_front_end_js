# Documentación Técnica Oficial: Portal de Servicios Nefrológicos

Este documento proporciona una descripción técnica detallada de la arquitectura, diseño e integración de servicios de la plataforma web estática para consultorios nefrológicos. Está diseñado para servir como anexo técnico y guía de referencia para los desarrolladores y mantenedores del proyecto.

---

## 1. Resumen Ejecutivo

El propósito principal de este sitio web es servir como un canal digital interactivo y de alta fidelidad para la presentación de servicios y la gestión preliminar de reservas de prácticas nefrológicas. La plataforma actúa como un portal informativo y funcional para pacientes que buscan atención médica especializada.

### Enfoque del Diseño
El diseño de la interfaz de usuario prioriza la accesibilidad y la confianza, elementos fundamentales en el ámbito de la salud. Se ha implementado una estética moderna, limpia y profesional apoyada en una paleta de colores coherente (tonos azules para evocar profesionalismo y seriedad, con acentos dorados para elementos destacados) y efectos visuales refinados. 

La experiencia de usuario está optimizada para permitir una navegación fluida, facilitando que el paciente encuentre la información de contacto, los consultorios físicos y realice pre-reservas de prácticas médicas de manera intuitiva y sin fricciones.

---

## 2. Arquitectura y Stack Tecnológico

El proyecto ha sido concebido bajo una arquitectura de **Front-End Estático y Serverless**. Esto significa que el procesamiento de datos del lado del servidor se delega a servicios externos especializados, eliminando la necesidad de desarrollar y mantener una infraestructura de base de datos o servidor propio.

### Stack de Tecnologías
*   **HTML5 Semántico:** Utilizado para estructurar el contenido de manera accesible y amigable para el SEO. Se emplean elementos nativos como `<header>`, `<nav>`, `<main>`, `<section>`, y `<footer>` que mejoran la indexación por motores de búsqueda y la compatibilidad con lectores de pantalla.
*   **CSS3 y Bootstrap 5:** Bootstrap 5 se emplea como base para el sistema de rejilla (*grid*), espaciados y componentes interactivos básicos (como el colapso del menú de navegación). Los estilos customizados se encuentran centralizados en [app.css], el cual define las variables globales del sistema, fuentes personalizadas (Inter) y componentes específicos como las clases `.tarjeta-premium` y `.tarjeta-servicio`.
*   **Vanilla JavaScript (ES6+):** Toda la lógica de interacción con el DOM, la gestión del estado del carrito de compras/reservas en memoria, y la comunicación asíncrona con el exterior se realiza con JavaScript nativo sin el peso o la sobrecarga de frameworks monolíticos (React, Angular o Vue). El catálogo de servicios nefrológicos se administra localmente mediante el archivo [productos.js].

### Justificación Técnica de la Elección
1.  **Rendimiento y Velocidad de Carga:** Al no requerir la compilación o descarga de librerías pesadas en tiempo de ejecución, el *Time to First Byte* (TTFB) y el *First Contentful Paint* (FCP) son excepcionalmente bajos. Las páginas cargan de manera casi instantánea, incluso en conexiones móviles lentas.
2.  **Costo Cero de Servidor y Mantenimiento:** La arquitectura serverless estática permite alojar la aplicación de forma gratuita en múltiples plataformas de CDN globales, con un costo de mantenimiento técnico nulo en comparación con un backend tradicional.
3.  **Seguridad y Confiabilidad:** Un sitio estático carece de una superficie de ataque para inyecciones SQL o exploits de ejecución remota de código en el servidor. La robustez y disponibilidad del sitio web son excepcionales, ya que es servido directamente desde una CDN.
4.  **Facilidad de Mantenimiento:** La estructura del proyecto está modularizada y es legible, permitiendo actualizaciones rápidas mediante modificaciones directas de archivos HTML y CSS estándares.

---

## 3. Diseño Responsivo (Mobile-First)

El sitio web está diseñado siguiendo la filosofía **Mobile-First** para asegurar una visualización impecable en dispositivos móviles, adaptándose de forma fluida a pantallas de mayor resolución mediante Media Queries estructurados.

### Estrategias de Adaptabilidad Empleadas
*   **Rejillas Flexibles (Flexbox y CSS Grid):** Se utiliza la rejilla de Bootstrap 5 basada en Flexbox (`row` y `col-*`) para reorganizar automáticamente los elementos visuales. Por ejemplo, en pantallas móviles los elementos de las tarjetas de servicios se apilan verticalmente (`col-12`), mientras que en pantallas medianas y de escritorio se expanden horizontalmente en múltiples columnas (`col-md-6`, `col-lg-3`).
*   **Elementos Flexibles y Flex-Wrap:** Se evita el uso de anchos fijos en contenedores principales, utilizando porcentajes o unidades relativas (`vw`, `vh`, `rem`, `em`).
*   **Media Queries para Componentes Propios:** En [app.css] se definen reglas específicas para ajustar los espaciados, tamaños de fuente y alturas de elementos clave según los puntos de ruptura estándar del dispositivo. Por ejemplo, la sección principal (`.seccion-principal`) posee márgenes dinámicos superiores para evitar la superposición con la barra de navegación persistente (`.navbar`).
*   **Menú de Navegación Adaptativo:** El navbar se colapsa automáticamente en un botón de hamburguesa (`navbar-toggler`) en pantallas táctiles y móviles, optimizando el espacio en pantalla y mejorando la ergonomía táctil.

---

## 4. Integración de Servicios Externos: FormSubmit

Para solventar la ausencia de un backend de correo, se utiliza el servicio externo **FormSubmit** (formsubmit.co), el cual captura las peticiones HTTP POST desde el cliente y las enruta como correos electrónicos formateados al buzón del administrador.

El proyecto implementa dos variantes de envío utilizando este servicio:

### A. Envío Tradicional con Redirección (contacto.html)
En la sección de contacto directo ([contacto.html], el formulario web se comunica con FormSubmit de forma directa mediante la estructura HTML tradicional:

```html
<form action="https://formsubmit.co/miguezexequiel@hotmail.com" method="POST">
    <!-- Parámetros de Control -->
    <input type="hidden" name="_captcha" value="false" />
    <input type="hidden" name="_next" value="gracias.html" />
    
    <!-- Campos del Formulario -->
    <input type="text" name="Nombre" required />
    <input type="email" name="Email" required />
    <textarea name="Mensaje" required></textarea>
    
    <button type="submit">Enviar Mensaje</button>
</form>
```

#### Flujo de Ejecución:
1.  El usuario completa los campos obligatorios del formulario y presiona el botón de envío.
2.  El navegador realiza una petición HTTP POST directa al endpoint de FormSubmit (`https://formsubmit.co/miguezexequiel@hotmail.com`).
3.  FormSubmit procesa los datos y, en lugar de retornar una respuesta JSON cruda, lee el parámetro oculto `_next` y redirige la ventana del usuario automáticamente a la página local de confirmación ([gracias.html].

---

### B. Envío Programático Asíncrono con AJAX (carrito.html)
Para el carrito de compras y pre-reserva de prácticas médicas ([carrito.html], se requería que el envío ocurriese en segundo plano para no interrumpir el flujo interactivo ni vaciar el estado visual del usuario bruscamente. Para ello, se intercepta el proceso mediante la API `fetch` de JavaScript.

#### Implementación del Envío Asíncrono:
```javascript
function enviarPedidoFormSubmit(datosComprador) {
    const datosEnvio = new FormData();
    datosEnvio.append("Nombre", datosComprador.nombre);
    datosEnvio.append("Apellido", datosComprador.apellido);
    datosEnvio.append("DNI", datosComprador.dni);
    datosEnvio.append("Teléfono", datosComprador.telefono);

    // Mapear dinámicamente cada práctica médica reservada en el carrito
    carrito.forEach((elemento, indice) => {
        datosEnvio.append(
            `Práctica ${indice + 1}`,
            `${elemento.nombre} (Cant: ${elemento.cantidad}) - Subtotal: $${(elemento.precio * elemento.cantidad).toLocaleString('es-AR')}`
        );
    });

    const subtotal = carrito.reduce((acumulador, elemento) => acumulador + (elemento.precio * elemento.cantidad), 0);
    const iva = subtotal * IVA_PORCENTAJE;
    const total = subtotal + iva;

    datosEnvio.append("Subtotal", `$${subtotal.toLocaleString('es-AR', { minimumFractionDigits: 2 })}`);
    datosEnvio.append("IVA (21%)", `$${iva.toLocaleString('es-AR', { minimumFractionDigits: 2 })}`);
    datosEnvio.append("Total General", `$${total.toLocaleString('es-AR', { minimumFractionDigits: 2 })}`);

    // Parámetros especiales de FormSubmit
    datosEnvio.append("_subject", `Nueva Reserva de Prácticas - ${datosComprador.nombre} ${datosComprador.apellido}`);
    datosEnvio.append("_captcha", "false");
    datosEnvio.append("_template", "box"); // Diseño del correo en formato bento box

    // Envío asíncrono utilizando el endpoint de AJAX de FormSubmit
    return fetch("https://formsubmit.co/ajax/miguezexequiel@hotmail.com", {
        method: "POST",
        body: datosEnvio
    })
    .then(respuesta => {
        if (!respuesta.ok) throw new Error("Error en el envío");
        return response.json(); // En código original usa respuesta.json()
    });
}
```

#### Flujo de Ejecución:
1.  El usuario hace clic en el botón de confirmación en la vista del carrito.
2.  La función `confirmarPedido()` valida que el formulario de datos del comprador sea correcto mediante `reportValidity()`.
3.  Se construye un objeto `FormData` con la información del cliente y se itera sobre el arreglo global `carrito` para añadir cada práctica reservada como un campo diferenciado.
4.  Se realiza una petición POST con `fetch` a la URL especial `https://formsubmit.co/ajax/miguezexequiel@hotmail.com`.
5.  El servidor de FormSubmit responde asíncronamente con un objeto JSON indicando el éxito de la operación. El código del cliente procesa esta respuesta, muestra una confirmación visual en pantalla, y procede a vaciar el carrito localmente de manera limpia.

---

### C. Seguridad y Configuración
*   **Desactivación de Captcha (`_captcha = false`):** Para mejorar la fricción en la experiencia de usuario (evitando que el usuario deba resolver retos visuales), se deshabilita el captcha. FormSubmit implementa algoritmos heurísticos en sus servidores para detectar spam de manera transparente.
*   **Estructuración visual (`_template = box`):** En la integración AJAX del carrito, se solicita el envío utilizando la plantilla `box` de FormSubmit, lo cual estructura el correo resultante en una tabla compacta y estética de lectura rápida.
*   **Activación inicial:** Por motivos de seguridad, la primera vez que se envía un formulario hacia una nueva dirección de correo (en este caso, `miguezexequiel@hotmail.com`), FormSubmit envía un email de confirmación a dicha casilla. El receptor debe hacer clic en el enlace adjunto para autorizar al dominio a enviar correos.

---

## 5. Guía de Ejecución y Despliegue

Al ser un proyecto estático compuesto únicamente por archivos planos (HTML, CSS, JS), no requiere procesos de compilación o transpilación.

### Ejecución Local
Para visualizar y probar el proyecto de forma local simulando un entorno de servidor web:

1.  **Mediante VS Code (Live Server):**
    *   Instalar la extensión **Live Server** de *Ritwick Dey*.
    *   Hacer clic derecho sobre [index.html] y seleccionar **"Open with Live Server"**.
    *   El navegador se abrirá en la dirección `http://127.0.0.1:5500/index.html`.

2.  **Mediante Node.js (http-server):**
    *   Tener Node.js instalado.
    *   Abrir una terminal en el directorio raíz del proyecto y ejecutar:
        ```bash
        npx http-server
        ```
    *   Acceder a `http://localhost:8080`.

3.  **Mediante Python:**
    *   Abrir una terminal en el directorio del proyecto y ejecutar:
        *   Python 3: `python -m http.server 8000`
        *   Python 2: `python -m SimpleHTTPServer 8000`
    *   Acceder a `http://localhost:8000`.

### Despliegue en Producción
El despliegue en un entorno productivo es directo e instantáneo. Las opciones recomendadas son:

1.  **GitHub Pages (Recomendado):**
    *   Subir el proyecto a un repositorio de GitHub.
    *   Ir a *Settings > Pages*.
    *   Seleccionar la rama `main` (o la rama de producción) y la carpeta `/root`.
    *   Hacer clic en *Save*. El sitio estará en línea en la URL proveída por GitHub.

2.  **Vercel / Netlify / Cloudflare Pages:**
    *   Vincular el repositorio de Git a la plataforma seleccionada.
    *   Establecer la configuración de compilación vacía (sin comando de build, directorio de salida raíz `./`).
    *   La plataforma detectará automáticamente los archivos estáticos y configurará una red de entrega de contenido (CDN) global.
