# U N I D A D  3
Protocolos dinarios
_________________________________________________________________________________________________________________________________________________________________________________________
# Ejercicio 1
**¿Cómo se ve un protocolo binario?** Para explorar este concepto te voy a mostrar una hoja de datos de un [sensor](http://www.chafon.com/productdetails.aspx?pid=382) comercial que usa un protocolo de comunicación binario. La idea es que tu explores su hoja de datos ([datasheet](https://drive.google.com/file/d/1uDtgNkUCknkj3iTkykwhthjLoTGJCcea/view?pli=1)), tanto como sea posible, invitándote a que observes con detenimiento hasta la página 5.
####
Durante la lectura, intenta dar respuestas a las siguientes preguntas:
- ¿Cómo se ve un protocolo binario?
Es una serie de bits que representan ciertos tipos de información, siendo por lo general de 8 bits o más (1 byte o más). De esta forma, cada byte puede almacenar una información específica, con valores diferentes que varían desde 0 a 255 bytes.
####
- ¿Puedes describir las partes de un mensaje?
1. **Cabecera (`Header`):**
La cabecera generalmente contiene información sobre el mensaje en sí, como el tipo de mensaje, el tamaño de los datos y la dirección del dispositivo que envía o recibe.
2. **Cuerpo (`Payload`):**
Esta es la parte principal del mensaje que contiene los datos útiles que se quieren transmitir, como lecturas del sensor, configuraciones o comandos.
3. **Pie (`Footer` o CRC):**
Puede incluir un código de verificación (Checksum o CRC) para asegurar la integridad del mensaje. A veces, también puede incluir una señal de finalización.
####
- ¿Para qué sirve cada parte del mensaje?
El `Header` sirve como forma de identificar qué tipo de mensaje se envía y su propósito, siendo generalmente fundamental para que se lea correctamente el mensaje. El `Payload` contiene la información que se desea transmitir, y es por así decirlo la carga de relevancia o lo que se quiere extraer del mensaje. Finalmente, el `Footer` contiene códigos de que sirvan para verificar que el mensaje ha sido recibido de forma idónea y que no hayan ocurrido errores.

# Ejercicio 2
En [este](https://www.arduino.cc/reference/en/language/functions/communication/serial/) enlace vas a mirar los siguientes métodos. Te pediré que, por favor, los tengas a mano porque te servirán para resolver problemas. Además, en este punto, hagamos un repaso de las funciones que han apoyado la comunicación seral:

- Serial.available() -> Lee los bytes enviados.
- Serial.read() -> Lee datos que se ingresen.
- Serial.readBytes(buffer, length) -> Lee los caracteres recibidos y los manda a un buffer especificado.
- Serial.write() -> Escribe datos en lenguaje binario, y los envía como bytes o series de bytes.
####
Nótese que la siguiente función no está en la lista de repaso:
``` c++
Serial.readBytesUntil()
```
**¿Sospecha por qué se ha excluido?** La razón es porque en un protocolo binario usualmente no tiene un carácter de *FIN DE MENSAJE*, como si ocurre con los protocolos ASCII, donde usualmente el último carácter es el `\n`.

# Ejercicio 3
Cuando trabajamos con protocolos binarios es necesario transmitir variables que tienen una longitud mayor a un byte. Por ejemplo, los números en punto flotante cumplen con el [estándar IEEE754](https://www.h-schmidt.net/FloatConverter/IEEE754.html) y se representan con 4 bytes.

Algo que debemos decidir al trabajar con números como los anteriormente descritos es el orden en el cual serán transmitidos sus bytes. En principio tenemos dos posibilidades: i) transmitir primero el byte de menor peso (*little endian*) o transmitir primero el byte de mayor peso (*big endian*). Por lo tanto, al diseñar un protocolo binario se debe escoger una de las dos posibilidades.

# Ejercicio 4
<aside>
**¡Desempolva ScriptCommunicator!**

Para este ejercicio vas a necesitar una herramienta que te permita ver los bytes que se están transmitiendo sin interpretarlos como caracteres ASCII. Usa **ScriptCommunicator** en los sistemas operativos Windows o Linux y **CoolTerm** en el sistema operativo MacOS (te soporta la arquitectura Mx).

</aside>

¿Cómo transmitir un número en punto flotante? Veamos dos alternativas:
####
Opción 1:
``` c++

```

# Ejercicio 5
Texto

####
Programa el controlador con este código:
``` c++

```

# Ejercicio 6
Texto

####
Programa el controlador con este código:
``` c++

```

# RETO
