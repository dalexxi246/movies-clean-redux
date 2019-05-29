# Movies Shopping Cart for Android - Spanish Version

Implementando una arquitectura Unidireccionál, aplicando los conceptos de Clean Architecture y programación funcional.

### Funcionalidades esperadas.

**Listado de películas.**
  - Obtener un listado de peliculas desde la Api [The Movie Database](www.themoviedb.org).
  - Guardar el listado de películas localmente, con el fin de poder 
  consultarlo aún sin conexión a internet.
  - Mostrar por cada pelicula si se encuentra agregada al carrito.

**Carrito offline.**
  - Seleccionár peliculas desde el listado para agregarlas al carrito.
  - Eliminar peliculas desde el carrito.
  - Persistir el carrito de forma offline, con el fin de poder consultarlo 
  aún sin conexión a internet. 
  
**Detalle de una pelicula**
  - Al dar Tap sobre una pelicula del listado de peliculas, se debe abrir 
  una vista con todos los detalles de la pelicula seleccionada.
  - Mostrar si la pelicula visualizada se encuentra en el carrito, y sus 
  unidades agregadas.
  
### Arquitectura.  

Se implementarán los siguientes capas para la aplicación:

**domain:** En esta capa central se aloja toda la logica del negocio. 
Solamente contiene codigo Kotlin y no tiene dependencias externas. 
En esta capa se utiliza un contenedor de estado basado en la arquitectura **Redux**,
la cual permite que el flujo de datos sea unidireccional y predecible.
Esta capa se compone del Store, los Reducers, los middlewares, el estado de la aplicación y las interfaces de los repositorios de datos.

**data:** Contiene la implementacion de los repositorios y fuentes de datos 
que provienen del exterior, como las conexiones a bases de datos y 
llamados a servicios web. Tambien tiene mappers que permiten transformar 
los datos recibidos de las entidades externas en datos del dominio.  
