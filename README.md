# trabajo practico 04 — mascotas en adopcion

## descripcion


aplicacion web construida con express y ejs que permite consultar mascotas en adopcion y agregar registros temporalmente mediante un formulario. los datos iniciales se cargan desde un archivo json y las nuevas mascotas se agregan en memoria.

la aplicacion produce paginas html renderizadas en el servidor, utilizando un layout principal, parciales de encabezado y pie, y vistas especificas para cada seccion.

## instalacion

---en consola ingresar "npm install"----y ejecute
--- ingresar npm start----- inicializa 
la aplicacion queda disponible en http://localhost:3000. para detener el servidor: ctrl + c.

paginas y rutas
metodo |	ruta	           |descripcion
GET	   |  /	pagina         |inicial con presentacion del refugio
GET	   | /mascotas	       |listado de mascotas en adopcion
GET	   | /mascotas/nueva	 |formulario para agregar una mascota
GET	   | /mascotas/:id	   |detalle de una mascota por identificador
POST	 | /mascotas	       |procesa el formulario y crea la mascota en memoria
estructura de vistas
views/layouts/main.ejs: estructura html general compartida por todas las paginas. contiene <!doctype html>, el <head>, el encabezado, el <main> con <%- body %> y el pie.
  -* views/partials/encabezado.ejs: navegacion principal reutilizable.
  -* views/partials/pie.ejs: pie de pagina reutilizable.
  -* views/inicio.ejs: contenido de la pagina inicial.
  -* views/mascotas/lista.ejs: listado con tarjetas y estado vacio.
  -* views/mascotas/detalle.ejs: detalle de una mascota.
  -* views/mascotas/nueva.ejs: formulario de nueva mascota.
  -* views/no-encontrado.ejs: pagina html para errores 404.

## -> diferencia entre layout, vista y parcial.

  -layout: es el marco html general que se repite en todas las paginas. define el <head>, la estructura del <body> y los puntos donde se insertan el encabezado, el contenido variable y el pie.
  -vista: es el contenido especifico de una pagina. cada ruta renderiza una vista distinta (inicio.ejs, lista.ejs, detalle.ejs, nueva.ejs, no-encontrado.ejs).
  -parcial: es un fragmento reutilizable que se incluye dentro de otras plantillas, como el encabezado o el pie. no representa una pagina completa.

el layout conserva la estructura compartida.

## recursos estaticos
/css/estilos.css

/img/mascota.svg

/js/app.js

express.static convierte la carpeta public en la raiz publica del servidor. por eso el nombre public no aparece en las urls: el archivo fisico public/css/estilos.css se sirve como /css/estilos.css.

## formulario
el formulario usa POST /mascotas y express.urlencoded para interpretar los datos enviados. la validacion minima comprueba:
  campos completos (nombre, especie, estado, descripcion), edad numerica mayor o igual a cero, estado dentro de los valores permitidos ("En adopción", "Reservada", "Adoptada"). si hay error, responde con estado 400 (muestra la alerta : <% if (error) { %>
    <p class="error" role="alert"><%= error %></p> <% } %> ) y vuelve a renderizar el formulario conservando los valores ingresados. si todo es valido, agrega la mascota al arreglo en memoria y redirige a /mascotas.
datos enviados mediante res.render (http://localhost:3000/mascotas)
res.render(vista, datos) envia un objeto con datos a la plantilla ejs. por ejemplo:
JavaScript
res.render("mascotas/lista", {
  titulo: "mascotas en adopcion",
  mascotas,
});
la vista recibe titulo y mascotas y los utiliza con <%= %> para mostrarlos escapados. los datos nunca viajan al navegador como .ejs; solo viaja el html ya generado.

  ## funcion de express.static
express.static(carpeta) registra un middleware que sirve los archivos de una carpeta como recursos publicos. express usa esa carpeta como raiz, por eso public desaparece de la url.

  ## funcion de express.urlencoded
express.urlencoded({ extended: false }) interpreta los cuerpos de formularios html enviados con Content-Type: application/x-www-form-urlencoded y deja los datos disponibles en req.body.

  ## recorrido POST, redireccion y GET
el navegador envia POST /mascotas con los datos del formulario.
express interpreta el cuerpo con express.urlencoded.
el manejador valida los datos.
si son validos, agrega la mascota al arreglo en memoria y responde con res.redirect("/mascotas") (estado 302).
el navegador hace una nueva solicitud GET /mascotas.
express renderiza el listado actualizado con estado 200.
este patron se llama POST → redirect → GET y evita reenviar el formulario al actualizar la pagina.

persistencia de los datos
los cambios se realizan unicamente en el arreglo en memoria. al reiniciar el servidor, el arreglo vuelve a cargarse desde datos/mascotas.json y las mascotas agregadas desaparecen. no se escribe en el archivo json.
la razon es que la funcion principal vuelve a ejecutar la lectura del json cada vez que se inicia el proceso, y el push del POST modifica solamente el arreglo en memoria, no el archivo en disco.
