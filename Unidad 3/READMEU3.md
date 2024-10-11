# U N I D A D  3
Protocolos dinarios
_________________________________________________________________________________________________________________________________________________________________________________________
# Ejercicio 1
**¿Cómo se ve un protocolo binario?** Para explorar este concepto te voy a mostrar una hoja de datos de un [sensor](http://www.chafon.com/productdetails.aspx?pid=382) comercial que usa un protocolo de comunicación binario. La idea es que tu explores su hoja de datos ([datasheet](https://drive.google.com/file/d/1uDtgNkUCknkj3iTkykwhthjLoTGJCcea/view?pli=1)), tanto como sea posible, invitándote a que observes con detenimiento hasta la página 5.
####
Durante la lectura, intenta dar respuestas a las siguientes preguntas:
- ¿Cómo se ve un protocolo binario?
####
Es una serie de bits que representan ciertos tipos de información, siendo por lo general de 8 bits o más (1 byte o más). De esta forma, cada byte puede almacenar una información específica, con valores diferentes que varían desde 0 a 255 bytes.

- ¿Puedes describir las partes de un mensaje?
####
1. **Cabecera (`Header`):**
La cabecera generalmente contiene información sobre el mensaje en sí, como el tipo de mensaje, el tamaño de los datos y la dirección del dispositivo que envía o recibe.
2. **Cuerpo (`Payload`):**
Esta es la parte principal del mensaje que contiene los datos útiles que se quieren transmitir, como lecturas del sensor, configuraciones o comandos.
3. **Pie (`Footer` o CRC):**
Puede incluir un código de verificación (Checksum o CRC) para asegurar la integridad del mensaje. A veces, también puede incluir una señal de finalización.

- ¿Para qué sirve cada parte del mensaje?
####
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

Si es *Little endian* se envía lo de la derecha primero, y si es *Big endian* lo de la izquierda.

# Ejercicio 4
**¡Desempolva ScriptCommunicator!**
Para este ejercicio vas a necesitar una herramienta que te permita ver los bytes que se están transmitiendo sin interpretarlos como caracteres ASCII. Usa **ScriptCommunicator** en los sistemas operativos Windows o Linux y **CoolTerm** en el sistema operativo MacOS (te soporta la arquitectura Mx).

¿Cómo transmitir un número en punto flotante? Veamos dos alternativas:
####
Opción 1:
``` c++
void setup() {
    Serial.begin(115200);
}

void loop() {
    // 45 60 55 d5
    // https://www.h-schmidt.net/FloatConverter/IEEE754.html
    static float num = 3589.3645;

    if(Serial.available()){
        if(Serial.read() == 's'){
            Serial.write ( (uint8_t *) &num,4);
        }
    }
}
```
Opción 2:
``` c++
void setup() {
    Serial.begin(115200);
}

void loop() {
// 45 60 55 d5// https://www.h-schmidt.net/FloatConverter/IEEE754.htmlstatic
float num = 3589.3645;
static uint8_t arr[4] = {0};

if(Serial.available()){
if(Serial.read() == 's'){
            memcpy(arr,(uint8_t *)&num,4);
            Serial.write(arr,4);
        }
    }
}
```
Aquí primero se copia la información que se desea transmitir a un buffer o arreglo:
####
Preguntas:
- ¿En qué *endian* estamos transmitiendo el número?
####
Big endian, ya que se está leyendo primero el número desde la izquierda.

- Y si queremos transmitir en el *endian* contrario, ¿Cómo se modifica el código?
####
idk

Pausa… A continuación, te dejo una posible solución a la pregunta anterior.
####
Opción 1: (sigue sin funcionar bien)
``` c++
void setup() {
    Serial.begin(115200);
}

void loop() {
    // 45 60 55 d5
    // https://www.h-schmidt.net/FloatConverter/IEEE754.html
    static float num = 3589.3645;

    if (Serial.available()) {
        char command = Serial.read();  // Leer un solo carácter

        if (command == 's') {
            Serial.write((uint8_t *) &num, 4); // Enviar el número original
        }
        
        if (command == 'a') {
            // Convertir el número a cadena
            String numStr = String(num);
            // Invertir la cadena
            String reversedStr = String();
            for (int i = numStr.length() - 1; i >= 0; i--) {
                reversedStr += numStr.charAt(i);
            }

            // Convertir la cadena invertida de vuelta a float
            float reversedNum = reversedStr.toFloat();

            // Enviar el número invertido
            Serial.write((uint8_t *) &reversedNum, 4); // Enviar el número invertido
        }
    }
```
Opción 2:
``` c++
void setup() {
    Serial.begin(115200);
}

void loop() {
// 45 60 55 d5// https://www.h-schmidt.net/FloatConverter/IEEE754.htmlstatic
float num = 3589.3645;
static uint8_t arr[4] = {0};

if(Serial.available()){
if(Serial.read() == 's'){
            memcpy(arr,(uint8_t *)&num,4);
for(int8_t i = 3; i >= 0; i--){
              Serial.write(arr[i]);
            }
        }
    }
}
```

# Ejercicio 5
Ahora te voy a pedir que practiques. La idea es que transmitas dos números en punto flotante en ambos endian.
####
Envía tres números en punto flotante.
``` c++
void setup() {
    Serial.begin(115200);
}

void loop() {
// 45 60 55 d5// https://www.h-schmidt.net/FloatConverter/IEEE754.htmlstatic
float num = 3589.3645; float num2 = 3647.0957; float num3 = 9460.2130;

static uint8_t arr[4] = {0};

if(Serial.available()){
if(Serial.read() == 's'){
            memcpy(arr,(uint8_t *)&num,4);
            Serial.write(arr,4);

Serial.write('\n');

            memcpy(arr,(uint8_t *)&num2,4);
            for(int8_t i = 3; i >= 0; i--)
            {
              Serial.write(arr[i]); 
            }

            Serial.write('\n');
            
            memcpy(arr,(uint8_t *)&num3,4);
            Serial.write(arr,4);
        }
    }
}
```

# Ejercicio 6
En este punto, te pido que repases, bien sea desde lo expuesto en la unidad anterior o remitiéndose a la documentación de C# de Microsoft.
####
**¿Para qué sirven los siguientes tres fragmentos de código y qué están haciendo?**
####
Para que Unity detecte el port. <` ✓ `>
``` c++
SerialPort _serialPort = new SerialPort();
_serialPort.PortName = "/dev/ttyUSB0";
_serialPort.BaudRate = 115200;
_serialPort.DtrEnable = true;
_serialPort.Open();
```
Se envía info al controlador. <` ✓ `>
``` c++
byte[] data = { 0x01, 0x3F, 0x45};
_serialPort.Write(data,0,1);
```
Pone el índice del buffer (4 para 4 bytes), y el otro es un if para ver si los bytes que se pueden leer son mayores o igual a 4, y escribirá los bytes enviados en punto flotante recorriendo el array. <` ✓ `>
``` c++
byte[] buffer =new byte[4];
.
.
.

if(_serialPort.BytesToRead >= 4){

    _serialPort.Read(buffer,0,4);
for(int i = 0;i < 4;i++){
        Console.Write(buffer[i].ToString("X2") + " ");
    }
}
```
....
####
Inventa una aplicación en Unity que utilice TODOS los métodos anteriores. Ten presente que necesitarás, además, inventar también la aplicación del microcontrolador.
####
#### Códigos
Microcontrolador:
``` c++
String btnState(uint8_t btnState)
{
    if(btnState == HIGH)
    {
        return "OFF";
    }
    else
        return "ON";
}

void task()
{
    enum class TaskStates
    {
        INIT,
        WAIT_COMMANDS
    };
    static TaskStates taskState = TaskStates::INIT;
    constexpr uint8_t led = 25;

    switch (taskState)
    {
        case TaskStates::INIT:
        {
            Serial.begin(115200);
            pinMode(led, OUTPUT);
            digitalWrite(led, LOW);
            taskState = TaskStates::WAIT_COMMANDS;
            break;
        }
        case TaskStates::WAIT_COMMANDS:
        {
            if (Serial.available() > 0)
            {
              String command = Serial.readStringUntil('\n');
              
              if (command == "triggerOn"){
              digitalWrite(led, HIGH);
              Serial.print("Trigger on");
              }
              else {
                if (command == "triggerOff"){
              Serial.print("Trigger off"); digitalWrite(led, LOW);
              }
              }
            }
            break;
        }
        default:
        {
            break;
        }
    }
}

void setup()
{
  task();
}

void loop()
{
  task();
}
```
Unity:
``` c++
using UnityEngine;
using System.IO.Ports;
using TMPro;
using static UnityEditor.Experimental.AssetDatabaseExperimental.AssetDatabaseCounters;
using System.Threading.Tasks;
using System;
using System.Runtime.ConstrainedExecution;

enum TaskState
{
    INIT,
    WAIT_COMMANDS
}

public class Serial1 : MonoBehaviour
{

    private static TaskState taskState = TaskState.INIT;
    private SerialPort _serialPort;
    private byte[] buffer;

    public GameObject flashlight; bool f; bool c;

    public GameObject shadow; int test = 0;

    private float s = 0f; private float fc = 0; string command = ""; int ss = -1;

    void Start()
    {
        _serialPort = new SerialPort();
        _serialPort.PortName = "COM4";
        _serialPort.BaudRate = 115200;
        _serialPort.DtrEnable = true;
        _serialPort.NewLine = "\n";
        _serialPort.Open();

        Debug.Log("Open Serial Port");
        byte[] buffer = new byte[4];

        flashlight.SetActive(false); f = false; c = false;
        shadow.SetActive(false);
    }

    void Update()
    {
        switch (taskState)
        {
            case TaskState.INIT:
                taskState = TaskState.WAIT_COMMANDS;
                Debug.Log("WAIT COMMANDS");
                break;
            case TaskState.WAIT_COMMANDS:
                if (_serialPort.BytesToRead >= 4) //cuando la linterna se haya prendido (o intentado) 4 veces.
                {
                    _serialPort.Write("triggerOn\n");
                    _serialPort.Read(buffer, 0, 4);
                    for (int i = 0; i < 4; i++)
                    {
                        //Console.Write(buffer[i].ToString("X2") + " ");
                        Debug.Log(buffer[i].ToString("X2") + " ");
                    }
                }
                else
                {
                    if (_serialPort.BytesToRead > 0)
                    {
                        _serialPort.Write("triggerOff\n");
                        string response = _serialPort.ReadLine();
                        command = response; Debug.Log(command);
                    }
                }
                break;
            default:
                Debug.Log("State Error");
                break;
        }

        if (Input.GetKeyDown("f") && fc <= 0)
        {
            flashlight.SetActive(true); f = true; c = false; fc = 1f; test++;
            Debug.Log("flaslight ON: " + test); ON();
        }
        if (f)
        {
            s += Time.deltaTime; // Debug.Log("flaslight countdowm: " + s);
        }
        if (s >= 0.1)
        {
            f = false; s = 0;
            flashlight.SetActive(false);
            Debug.Log("flaslight OFF"); c = true;
        }
        if (c && fc >= 0)
        {
            fc -= Time.deltaTime; // Debug.Log("flaslight cooldown: " + fc);
        }

        if (command == "Trigger on" || test == 4)
        {
            TRIGGER();
        }
    }
    void ON()
    {
        byte[] data = { 0x01, 0x3F, 0x45 };
        _serialPort.Write(data, 0, 1);
        Debug.Log("Sended flash");
    }
    private void TRIGGER()
    {
        float scaleFactor = 1.1f; ss++; shadow.SetActive(true); test = 0;

        Debug.Log("SHADOW SIZE : " + ss);

        if (ss > 0)
        {
            shadow.transform.localScale *= scaleFactor;
        }

        RandomLocation();
    }
    private void RandomLocation()
    {
        // Definir los límites manualmente
        float minX = 12f; // Límite izquierdo
        float maxX = 800f;  // Límite derecho
        float minY = 10f; // Límite inferior
        float maxY = 300f;  // Límite superior

        // Generar coordenadas aleatorias dentro de los límites definidos
        float randomX = UnityEngine.Random.Range(minX, maxX);
        float randomY = UnityEngine.Random.Range(minY, maxY);

        // Mover el objeto a la posición aleatoria
        shadow.transform.position = new Vector3(randomX, randomY, 0);
    }
}
```
#### Interfaz
La pantalla está completamente oscura, hasta que se enciende la linterna. Después de cierto tiempo aparece una "sombra".
####
![image](https://github.com/user-attachments/assets/031907ec-5031-4855-a207-d4fb05f7629c)

#### *¿Cómo funciona?*
Al iniciar el usuario será recibido por una pantalla completamente negra, y deberá presionar la tecla 'f' para poder `encender una linterna` por algunos milisegundos. Después de un tiempo, una sombra se podrá ver una "sombra" apareciendo cada vez más cerca. Esta se activa cuando el microcontrolador recibe al menos *4 o más bytes*. Esto hará que se `mueva de lugar` y se `acerque` cada que 4 o más bytes sean enviados.

# RETO
Vas a enviar 2 números en punto flotante desde un microcontrolador a una aplicación en Unity usando comunicaciones binarias. Además, inventa una aplicación en Unity que modifique dos dimensiones de un game object usando los valores recibidos.
####
Te voy a dejar una ayuda. Revisar el siguiente fragmento de código… *¿Qué hace?*
``` c++
byte[] buffer = new byte[4];
.
.
.
if(_serialPort.BytesToRead >= 4){
  _serialPort.Read(buffer,0,4);
  Console.WriteLine(System.BitConverter.ToSingle(buffer,0));
```
Presta especial atención `System.BitConverter.ToSingle`. Ahora, te pediré que busques en la documentación de Microsoft de C# qué más te ofrece `System.BitConverter`.
#### Códigos
Controlador:
``` c++

```
Unity:
``` c++

```
#### Interfaz


### *¿Cómo funciona?*


# TRABAJO FINAL
En este trabajo final vas a crear un protocolo binario para comunicar la aplicación del PC y el microcontrolador.
####
1. Vas a leer mínimo **tres variables**: la primera será la `variable real temperatura` a través del *sensor* incorporado en el Raspberry Pi Pico y las otras pueden ser `variables creadas a través de contadores`. En este trabajo las variables *deben* ser números en `punto flotante` y **el delta de cambio ya no es UNO**, sino un número en punto flotante.
2. Para leer las variables del microcontrolador, el *PC enviará un BYTE* (tu decides cuál). El microcontrolador responderá la solicitud con la información de *TODAS las variables*. Por cada variable se deben indicar tres elementos:
    - el `valor actual`
    - si `está habilitada`
    - el `intervalo` o `delta` de *cambio de la variable*.
4. Debes incluir un *byte extra* al final que cambiará en función de la información que envíes (*checksum*). Parte del ejercicio es consultar cómo puedes construir el `checksum` e implementarlo. La idea con este byte es que el receptor pueda verificar que la **información recibida no se dañó en el camino**.
5. El microcontrolador deberá mantener un **LED funcionando** a una frecuencia de `0.5 Hz`. El objetivo de este LED es que verifiques de manera visual que la aplicación en el microcontrolador *NUNCA se bloquea*.
####
Para desarrollar lo anterior **crea una mejor versión de la experiencia trabajada en la unidad anterior** y haz  los ajustes para incluir la variable temperatura en caso de que no la tengas y los otros elementos a los que haya lugar de acuerdo a los requerimientos.

#### Códigos
Controlador:
``` c++

```
Unity:
``` c++

```
#### Interfaz


### *¿Cómo funciona?*




