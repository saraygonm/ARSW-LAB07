### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW
### Laboratorio - Broker de Mensajes STOMP con WebSockets + HTML5 Canvas.


Este ejercicio se basa en la documentación oficial de SprinbBoot, para el [manejo de WebSockets con STOMP](https://spring.io/guides/gs/messaging-stomp-websocket/).

En este repositorio se encuentra una aplicación SpringBoot que está configurado como Broker de mensajes, de forma similar a lo mostrado en la siguiente figura:

![](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/images/message-flow-simple-broker.png)

En este caso, el manejador de mensajes asociado a "/app" aún no está configurado, pero sí lo está el broker '/topic'. Como mensaje, se usarán puntos, pues se espera que esta aplicación permita progragar eventos de dibujo de puntos generados por los diferentes clientes.

## Parte I.


1. Haga que la aplicación HTML5/JS permita ingresar, a través de dos campos, un valor de X y Y. Agregue un botón con una acción (definida en el módulo de JavaScript) que convierta los datos ingresados en un objeto JavaScript que tenga las propiedades X y Y, y los publique en el tópico: /topic/newpoint . Para esto tenga en cuenta (1) usar el cliente STOMP creado en el módulo de JavaScript, (2) enviar la representación textual del objeto JSON (usar JSON.stringify). Por ejemplo:

	```javascript
	stompClient.send("/topic/newpoint", {}, JSON.stringify({x:10,y:10}));
```

2. Dentro del módulo JavaScript modifique el método de conexión al WebSocket, para que la aplicación se suscriba al tópico "/topic/newpoint" (en lugar del tópico /TOPICOXX). Asocie como 'callback' de este suscriptor una función que muestre en un mensaje de alerta (alert()) el evento recibido. Como se sabe que en el tópico indicado se publicarán sólo puntos, extraiga el contenido enviado con el evento (objeto JavaScript en versión de texto), conviértalo en objeto JSON, y extraiga de éste sus propiedades (coordenadas X y Y). Para extraer el contenido del evento use la propiedad 'body' del mismo, y para convertirlo en objeto, use JSON.parse. Por ejemplo:

	```javascript
	var theObject=JSON.parse(message.body);
```
3. Compile y ejecute su aplicación. Abra la aplicación en varias pestañas diferentes (para evitar problemas con el caché del navegador, use el modo 'incógnito' en cada prueba).
4. Ingrese los datos, ejecute la acción del botón, y verifique que en todas la pestañas se haya lanzado la alerta con los datos ingresados.
5. Haga commit de lo realizado, y agregue un TAG para demarcar el avance de la parte 1:

	```bash
git tag -a v0.1 -m "Parte1"
	```

## Parte II.

Para hacer mas útil la aplicación, en lugar de capturar las coordenadas con campos de formulario, las va a capturar a través de eventos sobre un elemento de tipo \<canvas>. De la misma manera, en lugar de simplemente mostrar las coordenadas enviadas en los eventos a través de 'alertas', va a dibujar dichos puntos en el mismo canvas.

1. Agregue un elemento de tipo [Canvas](http://www.w3schools.com/html/html5_canvas.asp) de al menos 800x600 pixeles.
2. Tenga en cuenta las líneas 21 a 35 de [este ejemplo](http://www.html5canvastutorials.com/advanced/html5-canvas-mouse-coordinates/) para asociar un evento de mouse al canvas. A diferencia del ejemplo, usted lo va a hacer en el módulo javascript, en la sección que se ejecuta cuando el documento se ha cargado por completo ($(document).ready), y usará un evento de tipo "mousedown" en lugar de "mousemove".
3. Haga que el 'callback' asociado al tópico /topic/newpoint en lugar de mostrar una alerta, dibuje un punto en el canvas en las coordenadas enviadas con los eventos recibidos. Para esto puede [dibujar un círculo de radio 1](http://www.w3schools.com/html/html5_canvas.asp).
4. Ejecute su aplicación en varios navegadores (y si puede en varios computadores, accediendo a la aplicación mendiante la IP donde corre el servidor). Compruebe que a medida que se dibuja un punto, el mismo es replicado en todas las instancias abiertas de la aplicación.

5. Haga commit de lo realizado, y agregue un TAG para demarcar el avance de la parte 2:

	```bash
git tag -a v0.2 -m "Parte2"
	```

## Parte III.

En la configuración anterior, la aplicación de SpringBoot cumple el papel de 'Broker' pasivo, lo que significa que no tiene manera de controlar los mensajes recibidos y propagados. Se va a hacer una configuración alterna en la que, en lugar de que se propaguen automáticamente los mensajes, éstos será recibidos y procesados por el servidor, de manera que se pueda decidir qué hacer con los mismos. 

1. Cree una nueva clase que haga el papel de 'Controlador' para ciertos mensajes STOMP (en este caso, aquellos enviados a través de "/app/newpoint"). A este controlador se le inyectará un SimpMessagingTemplate, un Bean de Spring que permitirá publicar eventos en un determinado tópico. Por ahora, se definirá que cuando se intercepten los eventos enviados a "/app/newpoint" (que se supone deben incluir un punto), se mostrará por pantalla el punto recibido, y luego se procederá a reenviar el evento al tópico al cual están suscritos los clientes "/topic/newpoint".

	```java
	
	@Controller
	public class STOMPMessagesHandler {
		
		@Autowired
		SimpMessagingTemplate msgt;
	    
		@MessageMapping("/newpoint")    
		public void getLine(Point pt) throws Exception {
			System.out.println("Nuevo punto recibido en el servidor!:"+pt);
			msgt.convertAndSend("/topic/newpoint", pt);
		}
	}

	```
2. Ajuste su cliente para que, en lugar de publicar los puntos en el tópico /topic/newpoint, lo haga en /app/newpoint . Ejecute de nuevo la aplicación y rectifique que funcione igual, pero ahora mostrando en el servidor los detalles de los puntos recibidos.

3. Una vez rectificado el funcionamiento, se quiere aprovechar este 'interceptor' de eventos para cambiar ligeramente la funcionalidad:

	1. Se va a manejar un nuevo tópico llamado '/topic/newpolygon', en donde el lugar de puntos, se recibirán listas de puntos.
	2. El manejador de eventos de /app/newpoint, además de propagar los puntos mediante el tópico '/app/newpoints', llevará el control de últimos 4 puntos recibidos(que podrán haber sido dibujados por diferentes clientes). Cuando se completen los cuatro puntos, publicará la lista de puntos en el tópico '/topic/newpolygon'. Recuerde que esto se realizará concurrentemente, de manera que REVISE LAS POSIBLES CONDICIONES DE CARRERA!.
	3. El cliente, ahora también se suscribirá al tópico '/topic/newpolygon'. El 'callback' asociado a la recepción de eventos en el mismo debe, con los datos recibidos, dibujar un polígono, [tal como se muestran en ese ejemplo](http://www.arungudelli.com/html5/html5-canvas-polygon/).
	4. Verifique la funcionalidad: igual a la anterior, pero ahora dibujando polígonos cada vez que se agreguen cuatro puntos.
	
	

## Parte IV.

Normalmente, las aplicaciones integran Brokers de eventos y APIs REST. Suponga que se quiere permitir colaborar con los dibujos desde otro tipo de dispositivos que no soportan WebSockets. Para esto, haga un API REST con un recurso '/puntos', que únicamente maneje el verbo 'POST'. Haga que al recibir estas peticiones, el API haga lo mismo que realiza el manejador de los eventos publicados en '/app/newpoints': publicar un nuevo punto en /topic/newpoint, y publicar un polígono cuando se completen los cuatro puntos. Tenga en cuenta que esto implica que el controlador del API REST y el manejador de eventos deben compartir la variable que lleva la cuenta del número de puntos dibujados.


De nuevo, abra varios clientes, y verifique que mediante el comando curl (desde una terminal) sea posible agregar puntos (Nota: para esto, puede inyectar el _SimpMessagingTemplate_ al controlador del API).


## Opcional

Con la configuración actual, con tan solo abrir la aplicación, se realiza la conexión al Broker y la suscripción a los tópicos. Haga los ajustes necesarios para que la conexión no sea implícita, y que la misma se realice a través de un botón. Igualmente, agregue un botón de 'desconectar'.

Haga commit de lo realizado, y agregue un TAG para demarcar la versión final:

```bash	
git tag -a v0.3 -m "Final"	
```
## Opcional

Puede ajustar su cliente para que, además de eventos de mouse, [detecte eventos de pantallas táctiles](http://www.homeandlearn.co.uk/JS/html5_canvas_touch_events.html), de manera que los clientes móviles también puedan interactuar con la aplicación!.


### Criterios de evaluación

1. La aplicación propaga correctamente los puntos entre todas las instancias abierta de la misma.
2. La aplicación propaga correctamente el evento de creación del polígono, cuando colaborativamente se insertan cuatro puntos.
3. En la implementación se tuvo en cuenta la naturaleza concurrente del ejercicio. Por ejemplo, si se mantiene el conjunto de los puntos recibidos en una colección, la misma debería ser de tipo concurrente (thread-safe).
4.a. [Puntos opcionales] La aplicación permite controlar la conexión/desconexión al broker de mensajes.
4.b. [Puntos opcionales] La aplicación acepta eventos táctiles.
