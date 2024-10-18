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
Microcontrolador:
``` c++
void setup() {
    Serial.begin(9600);  // Inicializa la comunicación serie a 9600 bps
    pinMode(13, OUTPUT); // Configura el pin 13 como salida
}

void loop() {
    // Genera dos números de punto flotante aleatorios
    float num1 = random(0, 100) / 10.0; // Un número entre 0.0 y 10.0
    float num2 = random(0, 100) / 10.0; // Un número entre 0.0 y 10.0

    //Serial.print(num1); Serial.print(","); Serial.print(num2);
    //Serial.print('\n'); Serial.print('\n');

    // Envía los números en formato binario
    Serial.write((uint8_t*)&num1, sizeof(num1)); // Envía el primer float
    Serial.write((uint8_t*)&num2, sizeof(num2)); // Envía el segundo float
    //Serial.print('\n');
    
    // Enciende el LED
    digitalWrite(13, HIGH);
    delay(200); // Mantiene el LED encendido por un momento
    digitalWrite(13, LOW); // Apaga el LED
    
    delay(1000); // Espera 1 segundo antes de enviar nuevamente
}
```
Unity:
``` c++
using UnityEngine;
using System.IO.Ports;
using System;
public class Serial2 : MonoBehaviour
{
    private SerialPort _serialPort;
    private Vector3 position;
    private float x, y; int b;
    public SpriteRenderer sprite;
    void Start()
    {
        // Cambia "COM4" por el puerto que estés usando
        _serialPort = new SerialPort("COM4", 9600);
        try
        {
            _serialPort.Open();
        }
        catch (Exception e)
        {
            Debug.LogError("Error al abrir el puerto serie: " + e.Message);
        }
        x = 0; y = 0; b = 0;
        position = new Vector3(x, y, transform.position.z);
        transform.position = position;
    }
    void Update()
    {
        // Obtiene la posición de la cámara
        Camera mainCamera = Camera.main;

        // Calcula los límites en el espacio del mundo
        Vector3 min = mainCamera.ViewportToWorldPoint(new Vector3(0, 0, mainCamera.nearClipPlane));
        Vector3 max = mainCamera.ViewportToWorldPoint(new Vector3(1, 1, mainCamera.nearClipPlane));

        if (_serialPort.IsOpen && _serialPort.BytesToRead >= 8)
        {
            byte[] buffer = new byte[8];
            _serialPort.Read(buffer, 0, 8);

            x = BitConverter.ToSingle(buffer, 0);
            y = BitConverter.ToSingle(buffer, 4);

            x = Mathf.Clamp(x, min.x, max.x);
            y = Mathf.Clamp(y, min.y, max.y);

            // Modificar la posición del GameObject
            position = new Vector3(x, y, transform.position.z);
            transform.position = position;

            Debug.Log("Triangle moved to (" + x + "," + y + ")");
        }
        if (b >= 8)
        {
            x = UnityEngine.Random.Range(min.x, max.x);
            y = UnityEngine.Random.Range(min.y, max.y);

            position = new Vector3(x, y, transform.position.z);
            transform.position = position;

            Debug.Log("Triangle moved to (" + x + "," + y + ")");

            b = 0;
        }

        if (Input.GetKeyDown("a"))
        {
            sprite.flipY = false;
            SendKeyPress(1); // Envía un byte (1) para indicar que se presionó "a"
            b++; Debug.Log("Bytes enviados: " + b);
        }
        if (Input.GetKeyDown("d"))
        {
            sprite.flipY = true;
            SendKeyPress(2); // Envía un byte (2) para indicar que se presionó "d"
            b++; Debug.Log("Bytes enviados: " + b);
        }

        void SendKeyPress(byte key)
        {
            if (_serialPort.IsOpen)
            {
                _serialPort.Write(key.ToString());
            }
        }
    }

    void OnApplicationQuit()
    {
        if (_serialPort != null && _serialPort.IsOpen)
        {
            _serialPort.Close();
        }
    }
}
```
#### Interfaz
![image](https://github.com/user-attachments/assets/06d35c64-da7d-45fa-83ff-4a3f30fc31cd)

### *¿Cómo funciona?*
El triangulo se voltea presionando 'a' para la `izquierda`, y 'd' para la `derecha`. Cuando se voltea 8 veces o más, signfica que también se han enviado `8 bytes` o más, por tanto, cuando esto suceda, dos números creados aleatoriamente por el controlador serán enviados a Unity para que los use como coordenadas para el triángulo. Entonces, al enviarse los `8 bytes`, el triángulo `cambiará de posición`.

El programa funcionará de esta manera indefinidamente.

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
Microcontrolador:
``` c++
const int ledPin = 25; 
unsigned long previousMillis = 0; 
const long interval = 250; 
float currentLumenes = 50.0; float currentTemperature = 22.0; 
float pesos = 100000.0; 
bool lumenesHabilitados = true; bool temperaturaHabilitada = true; 
float deltaLumenes = 5.0; float deltaTemperatura = 0.5; float deltaPesos = -100.0;

// Función para parpadear el LED 
void blinkLED() { unsigned long currentMillis = millis();

if (currentMillis - previousMillis >= interval)
{
    previousMillis = currentMillis;
    int ledState = digitalRead(ledPin);
    digitalWrite(ledPin, !ledState);
}
}

void setup() { Serial.begin(9600); pinMode(ledPin, OUTPUT); }

void loop() { blinkLED(); // Parpadeo del LED

// Si el PC envía un byte de solicitud, responder con la información de las variables
if (Serial.available() > 0)
{
    char request = Serial.read();  // Leer el BYTE de solicitud
    if (request == 'A')  // Usaremos el BYTE 'A' como solicitud
    {
        // Enviar los valores de todas las variables
        Serial.print("Lumenes:");
        Serial.print(currentLumenes);
        Serial.print(", Habilitado:");
        Serial.print(lumenesHabilitados ? "1" : "0");
        Serial.print(", Delta:");
        Serial.println(deltaLumenes);

        Serial.print("Temperatura:");
        Serial.print(currentTemperature);
        Serial.print(", Habilitada:");
        Serial.print(temperaturaHabilitada ? "1" : "0");
        Serial.print(", Delta:");
        Serial.println(deltaTemperatura);

        Serial.print("Pesos:");
        Serial.print(pesos);
        Serial.print(", Habilitada:");
        Serial.print((pesos >= 0) ? "1" : "0");
        Serial.print(", Delta:");
        Serial.println(deltaPesos);
    }
}
}
```
Unity:
``` c++
using TMPro;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.SceneManagement;
using System.IO.Ports;

public class SerialCommunication : MonoBehaviour
{
    private SerialPort serialPort;
    public TextMeshProUGUI pesosText;
    public Slider temperatureSlider;
    public Slider lumenSlider;
    public Button buyButton;
    public Button buyExpensiveButton;    // Botón para la compra cara
    public Button buyCheapButton;        // Botón para la compra barata
    public TextMeshProUGUI mensajeErrorText;

    void Start()
    {

        if (GameManager.instance.comunicacionHabilitada)
        {
            mensajeErrorText.alpha = 0;
            // Usar el baud rate seleccionado
            serialPort = new SerialPort("COM4", GameManager.instance.baudRate);
            try
            {
                serialPort.Open();
                serialPort.ReadTimeout = 1000;
            }
            catch (System.Exception e)
            {
                Debug.LogError("Error al abrir el puerto serial: " + e.Message);
            }
        }

        // Cargar los valores desde el GameManager
        lumenSlider.value = GameManager.instance.lumenes;
        temperatureSlider.value = GameManager.instance.temperatura;
        pesosText.text = "Pesos: " + GameManager.instance.pesos;

        buyButton.onClick.AddListener(SendValuesToMicrocontroller);
        buyExpensiveButton.onClick.AddListener(BuyExpensiveLight);
        buyCheapButton.onClick.AddListener(BuyCheapLight);
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Return))
        {
            CheckTemperatureAndChangeScene();
        }
        
        if (serialPort.IsOpen && serialPort.BytesToRead > 0)
        {
            string receivedData = serialPort.ReadLine();
            Debug.Log("Datos recibidos del microcontrolador: " + receivedData);
        }
        
         if (Input.GetKeyDown(KeyCode.Return))
        {
            CheckLumenesAndChangeScene();
        }
    }

    void SendValuesToMicrocontroller()
    {
        if (serialPort.IsOpen)
        {
            float lumenes = Mathf.RoundToInt(lumenSlider.value);
            float temperatura = temperatureSlider.value;

            string dataToSend = $"L={lumenes},T={temperatura:F2},P={GameManager.instance.pesos}";
            serialPort.WriteLine(dataToSend);
            Debug.Log("Datos enviados al microcontrolador: " + dataToSend);

            if (GameManager.instance.pesos >= GameManager.instance.precioRL && GameManager.instance.pesosHabilitados == true)
            {
                GameManager.instance.pesos -= GameManager.instance.precioRL;
                if (GameManager.instance.luzHabilitada == true)
                {
                    lumenSlider.value += GameManager.instance.aumentoLyT;
                }
                else
                {
                    mensajeErrorText.alpha = 100;
                    mensajeErrorText.text = "Luz no habilitada";
                }
                if (GameManager.instance.temperaturaHabilitada == true)
                {
                    temperatureSlider.value += GameManager.instance.aumentoLyT;
                }
                else
                {
                    mensajeErrorText.alpha = 100;
                    mensajeErrorText.text = "Temperatura no habilitada";
                }               
                pesosText.text = "Pesos: " + GameManager.instance.pesos;
            }
            else
            {
                mensajeErrorText.alpha = 100;
                mensajeErrorText.text = "No hay suficiente dinero para realizar la compra. o no está habilitada";
            }
        }
    }

    void CheckTemperatureAndChangeScene()
    {
        float currentTemperature = temperatureSlider.value;

        if (currentTemperature >= 26f)
        {
            Debug.Log("Temperatura suficiente. Cambiando a la escena Piso4.");
            SceneManager.LoadScene("Piso4Lore");
        }
        else if (GameManager.instance.pesosHabilitados == false)
        {
            SceneManager.LoadScene("Piso4Lore");
        }
        else
        {
            Debug.Log("La temperatura es demasiado baja para cambiar de escena.");
        }
    }

    void BuyExpensiveLight()
    {
        float cost = GameManager.instance.precioVelon;  // Precio de la luz cara
        float lumenIncrease = GameManager.instance.aumento_L;  // Aumento significativo de lúmenes

        if (GameManager.instance.pesos >= cost)
        {
            // Reducir el dinero y aumentar los lúmenes
            GameManager.instance.pesos -= cost;
            GameManager.instance.lumenes += lumenIncrease;

            // Actualizar la UI
            pesosText.text = "Pesos: " + GameManager.instance.pesos;
            lumenSlider.value = GameManager.instance.lumenes;

            Debug.Log("Luz cara comprada. Lúmenes: " + GameManager.instance.lumenes + ", Pesos: " + GameManager.instance.pesos);
        }
        else
        {
            Debug.LogWarning("No tienes suficientes pesos para comprar el velón.");
        }
    }
    void BuyCheapLight()
    {
        float cost = GameManager.instance.precioCP;  // Precio de la luz barata
        float lumenIncrease = GameManager.instance.aumento_l;  // Aumento menor de lúmenes

        if (GameManager.instance.pesos >= cost)
        {
            // Reducir el dinero y aumentar los lúmenes
            GameManager.instance.pesos -= cost;
            GameManager.instance.lumenes += lumenIncrease;

            // Actualizar la UI
            pesosText.text = "Pesos: " + GameManager.instance.pesos;
            lumenSlider.value = GameManager.instance.lumenes;

            Debug.Log("Luz barata comprada. Lúmenes: " + GameManager.instance.lumenes + ", Pesos: " + GameManager.instance.pesos);
        }
        else
        {
            Debug.LogWarning("No tienes suficientes pesos para comprar las chispitas mariposa.");
        }
    }
    void CheckLumenesAndChangeScene()
    {
        if (GameManager.instance.lumenes >= 100)
        {
            Debug.Log("Lúmenes suficientes. Cambiando a la escena Piso3.");
            SceneManager.LoadScene("Piso3");  // Cambiar a la escena Piso3
        }
        else
        {
            Debug.Log("No tienes suficientes lúmenes para cambiar de escena.");
        }
    }

    private void OnApplicationQuit()
    {
        if (serialPort.IsOpen)
        {
            serialPort.Close();
        }
    }
}
```
#### Interfaz
![image](https://github.com/user-attachments/assets/5fe7f88b-e47a-45df-9747-cbbe182e3697)

