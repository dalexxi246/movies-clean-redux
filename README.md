# Movies Shopping Cart for Android - Spanish Version

Implementando una arquitectura Unidireccionál, aplicando los conceptos de Clean Architecture y programación funcional.

### Funcionalidades esperadas.

**Listado de películas.**
  - Obtener un listado de peliculas desde la Api [The Movie Database](https://www.themoviedb.org).
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
Esta capa se compone del Store, los Reducers, los middlewares, el estado 
de la aplicación y las interfaces de los repositorios de datos.

**data:** Contiene la implementacion de los repositorios y fuentes de datos 
que provienen del exterior, como las conexiones a bases de datos y 
llamados a servicios web. Tambien tiene mappers que permiten transformar 
los datos recibidos de las entidades externas en datos del dominio.  

**feature_xyz:** Una capa que comience con el prefijo _feature_ contiene 
la logica para construir las vistas y mantener su estado. Una vista se 
conforma con un ViewModel, el cual mantiene un estado inmutable de las vistas. 
Dicho estado solo puede ser mutado mediante acciones, las cuales representan la 
intención del usuario para realizar una interacción con la app. Por otra 
parte las actiivities y fragments de esta capa solamente se comunican con 
el ViewModel enviandole acciones y reaccionando a los cambios en el estado 
que mantiene el ViewModel.   

### ¿Porqué Redux en el dominio?

Redux es una arquitectura en la cual el flujo de datos se realiza de manera 
unidireccional. Esta arquitectura se basa en 3 principios fundamentales:

- El estado es la unica fuente de la verdad: 
El estado de toda tu aplicación esta almacenado en un árbol guardado en un único store. 

- El estado es inmutable; es un objeto que solo tiene propiedades de solo lectura, por lo cual
no puede ser modificado directamente.
La única forma de modificar el estado es emitiendo una acción, el cual es 
un objeto describiendo qué ocurrió.

- Los cambios se realizan con funciones puras:
Para especificar como el árbol de estado es transformado por las acciones, 
se utilizan reducers puros. 

Estos 3 principios son los que hacen que las aplicaciones basadas en Redux sean facilmente mantenibles,
altamente testeable, y que los bugs sean minimos y predecibles. 

### Componentes de Redux

Ya he mencionado algunos conceptos clave para entender Redux, pero a continuación serán detallados.

- Estado de la aplicación: Es un objeto donde se mantiene la informacion relevante para la aplicación en tiempo de ejecución. 
Este estado se conforma con propiedades de solo lectura, por lo que no puede ser modificado directamente (es inmutable). 

- **Acciones:** La unica forma de modificar el estado global de la aplicacion es mediante acciones. Una acción es un 
objeto que describe el tipo de flujo que se desea iniciar en la aplicación mediante una interacción externa (un gesto realizado
por el usuario, una notificación push, la respuesta de un servicio, la lectura de un archivo desde la base de datos, etc.) 

- **Reducers:** Un reducer es una funcion pura que recibe como parametro 
el estado actual y una acción, y retorna un nuevo estado, como resultado de la transformacion
interna realizada por el reducer.

- **Store:** Es el objeto que aloja el estado, recibe las acciones, utiliza los reducers definidos para transformar dichas acciones
en un nuevo estado, y notifica al exterior las actualizaciones del estado. El Store expone 3 metodos para interactuar con él:

    - getState(): Retorna el estado actual.
    - dispatch(action): Ejecuta el llamado a los reducers para cambiar el estado.
    - subscribe(listener): Registra los listeners que estan atentos a los cambios del estado. 

Si bien para aplicaciónes sencillas es mas que suficiente tener un contenedor de estado con estos componentes, para la mayoria de 
aplicaciónes que realizan procesos asincronos (consumo de servicios web, lectura de datos en persistencia, 
navegación y espera de decisiones por parte del usuario), es necesario incorporar un mecanismo para que las actualizaciónes del estado tambien 
puedan realizarse de forma asincrona. Es ahi donde entran los **Middlewares**.

Un Middleware es una funcion con la siguiente estructura:

`typealias Middleware = (Action, () -> State, (Action) -> Unit) -> Unit`

- **Action:** El primer parametro es la accion recibida inicialmente
- **() -> State:** El segundo es una funcion mediante la cual podemos acceder al estado actual.
- **(Action) -> Unit:** El tercero es una funcion dispatcher, mediante la 
cual se va a realizar el dispatch de la acción resultante del procesamiento hecho por el Middleware.

Un flujo asincrono, como por ejemplo una petición a la API, podria representarse 
mediante un Middleware de la siguiente forma:

1. El middleware se ejecuta recibiendo la acción inicial (`FetchMovies`), 
la cual realiza el cambio del estado `isLoading` a true, mostrando al usuario una 
pantalla de carga:
2. Dentro del Middleware hay una dependencia externa (función para consumir el Web Service), 
la cual invocamos para realizar la petición y esperar su respuesta.
3. Una vez recibida, la respuesta se transforma en una nueva acción dependiendo del resultado.
4.  Si es positivo y obtenemos peliculas del servicio, Se invoca la funcion 
dispatcher con una accion `SetMovies(movies: List<Movie>)`, la cual invoca al reducer
que actualiza el estado con la lista de peliculas, y coloca el estado 
`isLoading` en false para que desaparezca la pantalla de carga.
5. Si el servicio nos revuelve un error, ésta respuesta se transforma en 
la accion `NotifyErrorFetchingMovies` la cual al enviarse al reducer hace 
que la pantalla de carga desaparezca (estado `isLoading` pasa a `false`), 
y ademas muestra un mensaje con el error. 

 