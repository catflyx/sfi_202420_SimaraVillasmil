# U N I D A D  2
Protocolos ASCII
_________________________________________________________________________________________________________________________________________________________________________________________
# Ejercicio 1
Estudia con detenimiento el código para el controlador y para el computador. Busca la definición de todas las funciones usadas en la documentación de Arduino y de Microsoft.
- ¿Quién debe comenzar primero la comunicación, el *computador* o el *controlador*? **¿Por qué?**

####
Programa el controlador con este código:
``` c++
void setup() {
  Serial.begin(115200);
}

void loop() {
if(Serial.available()){
if(Serial.read() == '1'){
      Serial.print("Hello from Raspberry Pi Pico");
    }
  }
}
```
- Prueba la aplicación con ScriptCommunicator. **¿Cómo funciona?**
Es un programa que evalúa desde el la función `loop` si se ha presionado la tecla del '1'. Si es así, se imprime el mensaje `"Hello from Raspberry Pi Pico"`.
####
Crea un nuevo C# Script y un Game Object en Unity 2D. Añade el Script al GameObject. Ve al menu Assets y luego selecciona Open C# Project.
``` c++
using UnityEngine;
using System.IO.Ports;
public class Serial : MonoBehaviour
{
private SerialPort _serialPort =new SerialPort();
private byte[] buffer =new byte[32];

void Start()
    {
        _serialPort.PortName = "COM4";
        _serialPort.BaudRate = 115200;
        _serialPort.DtrEnable =true;
        _serialPort.Open();
        Debug.Log("Open Serial Port");
    }

void Update()
    {

				if (Input.GetKeyDown(KeyCode.A))
        {
            byte[] data = {0x31};// or byte[] data = {'1'};            
						_serialPort.Write(data,0,1);
            Debug.Log("Send Data");
        }

				if (Input.GetKeyDown(KeyCode.B))
        {
						if (_serialPort.BytesToRead >= 16)
            {
                _serialPort.Read(buffer, 0, 20);
                Debug.Log("Receive Data");
                Debug.Log(System.Text.Encoding.ASCII.GetString(buffer));
            }
        }

    }
}
```
Analiza:
- ¿Por qué es importante considerar las propiedades `PortName` y `BaudRate`? ¿Qué relación tienen las propiedades anteriores con el controlador?
####
`PortName` hay que considerarlo ya que es desde donde se leerá la placa, y necesita estar bien definido. Mientras tanto, `BaudRate` funfiona como el `Serial.begin()`, fijando la velocidad a la que se envía datos en 115200.

# Ejercicio 2
Ahora realiza este experimento. Modifica la aplicación del PC así:
``` c++
using UnityEngine;
using System.IO.Ports;
using TMPro;

public class Serial : MonoBehaviour
{
	private SerialPort _serialPort =new SerialPort();
	private byte[] buffer =new byte[32];

	public TextMeshProUGUI myText;

	private static int counter = 0;

	void Start()
    {
        _serialPort.PortName = "COM4";
        _serialPort.BaudRate = 115200;
        _serialPort.DtrEnable =true;
        _serialPort.Open();
        Debug.Log("Open Serial Port");
    }

void Update()
    {
        myText.text = counter.ToString();
        counter++;

				if (Input.GetKeyDown(KeyCode.A))
        {
            byte[] data = {0x31};// or byte[] data = {'1'};            
						_serialPort.Write(data,0,1);
            int numData = _serialPort.Read(buffer, 0, 20);
            Debug.Log(System.Text.Encoding.ASCII.GetString(buffer));
            Debug.Log("Bytes received: " + numData.ToString());
        }
    }
}
```
A continuación, debes adicionar a la aplicación un elemento de GUI tipo `Text - TextMeshPro` y luego, arrastrar una referencia a este elemento a `myText` (si no sabes cómo hacerlo llama al profe).
####
Y la aplicación del controlador:
``` c++
void setup()
{
		Serial.begin(115200);
}

void loop()
{
		if(Serial.available())
		{
				if(Serial.read() == '1')
				{
			      delay(3000);
			      Serial.print("Hello from Raspi");
				}
		}
}
```
Ejecuta la aplicación en Unity. Verás un número cambiar rápidamente en pantalla. Ahora presiona la `tecla A` (no olvides dar click en la pantalla Game). **¿Qué pasa? ¿Por qué crees que ocurra esto?**
####
En la consola dice que se han recibido *20 bytes* cada que se presiona la `tecla A`. Creería que esto sucede porque se está definiendo esta variable `int numData = _serialPort.Read(buffer, 0, 20);` con un 20, y esta misma es la que se pide que se imprima.
####
Prueba con el siguiente código. Luego, *ANALIZA CON DETENIMIENTO*. Una vez más, no olvides cambiar el puerto serial.
``` c++
using UnityEngine;
using System.IO.Ports;
using TMPro;

public class Serial : MonoBehaviour
{
		private SerialPort _serialPort =new SerialPort();
		private byte[] buffer =new byte[32];

		public TextMeshProUGUI myText;

		private static int counter = 0;

		void Start()
		{
		    _serialPort.PortName = "COM4";
		    _serialPort.BaudRate = 115200;
        _serialPort.DtrEnable =true;
        _serialPort.Open();
        Debug.Log("Open Serial Port");
		}

		void Update()
		{
        myText.text = counter.ToString();
        counter++;

				if (Input.GetKeyDown(KeyCode.A))
				{
            byte[] data = {0x31};// or byte[] data = {'1'};
            _serialPort.Write(data,0,1);
        }
				if (_serialPort.BytesToRead > 0)
			  {
	          int numData = _serialPort.Read(buffer, 0, 20);
            Debug.Log(System.Text.Encoding.ASCII.GetString(buffer));
            Debug.Log("Bytes received: " + numData.ToString());
        }
		}

```
**¿Funciona?**
No, al final le faltana un `}`.
####
- Por ejemplo, ¿Qué pasaría si al momento de ejecutar la instrucción `int numData = _serialPort.Read(buffer, 0, 20);` solo han llegado *10* de los *16* bytes del mensaje?
####
No se enviaría el mensaje completo, pues bytes menores a 16 ya se condiseran incompletos.
####
- ¿Cómo puede hacer tu programa para saber que ya tiene el mensaje completo?
####
Si tiene 16 o más bytes.
####
- ¿Cómo se podría garantizar que antes de hacer la operación `Read` tenga los *16 bytes* listos para ser leídos?
####
Que en el mínimo se ponga un 16 directamente, y asi no se enviarán mensajes con menor número de bytes. (`int numData = _serialPort.Read(buffer, 16, 20);`)
####
- Además, si los mensajes que envía el controlador tienen tamaños diferentes, ¿Cómo haces para saber que el mensaje enviado está **completo** o **faltan bytes** por recibir?
####
Ponerle un delimitador o `\n` al final a los mensajes siempre, pues al detectarse este delimitador el programa enviará todos los caractéres anteriores a este.

# Ejercicio 3
Nótese que en los dos ejercicios anteriores, el PC primero le pregunta al controlador por datos (le manda un 1). **¿Y si el PC no pregunta?**
####
Realiza el siguiente experimento. Programa ambos códigos y analiza su funcionamiento:
``` c++
void task()
{
		enum class TaskStates
		{
				INIT,
				WAIT_INIT,
				SEND_EVENT
		};
		static TaskStates taskState = TaskStates::INIT;
		static uint32_t previous = 0;
		static u_int32_t counter = 0;

		switch (taskState)
		{
				case TaskStates::INIT:
				{
						Serial.begin(115200);
						taskState = TaskStates::WAIT_INIT;
						break;
				}
				case TaskStates::WAIT_INIT:
				{
						if (Serial.available() > 0)
						{
								if (Serial.read() == '1')
								{
										previous = 0; // Force to send the first value immediately
										taskState = TaskStates::SEND_EVENT;
								}
						}
						break;
				}
				case TaskStates::SEND_EVENT:
				{
						uint32_t current = millis();
						if ((current - previous) > 2000)
						{
								previous = current;
								Serial.print(counter);
								counter++;
						}
						if (Serial.available() > 0)
						{
							  if (Serial.read() == '2')
							  {
								    taskState = TaskStates::WAIT_INIT;
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
``` c++
using UnityEngine;
using System.IO.Ports;
using TMPro;

enum TaskState
{
    INIT,
    WAIT_START,
    WAIT_EVENTS
}

public class Serial : MonoBehaviour
{
		private static TaskState taskState = TaskState.INIT;
		private SerialPort _serialPort;
		private byte[] buffer;
		public TextMeshProUGUI myText;
		private int counter = 0;

		void Start()
    {
        _serialPort =new SerialPort();
        _serialPort.PortName = "COM3";
        _serialPort.BaudRate = 115200;
        _serialPort.DtrEnable =true;
        _serialPort.Open();
        Debug.Log("Open Serial Port");
        buffer =new byte[128];
    }

void Update()
    {
        myText.text = counter.ToString();
        counter++;

				switch (taskState)
        {
						case TaskState.INIT:
		            taskState = TaskState.WAIT_START;
                Debug.Log("WAIT START");
								break;
						case TaskState.WAIT_START:
								if (Input.GetKeyDown(KeyCode.A))
                {
                    byte[] data = {0x31};// start
                    _serialPort.Write(data,0,1);
                    Debug.Log("WAIT EVENTS");
                    taskState = TaskState.WAIT_EVENTS;
                }
								break;
						case TaskState.WAIT_EVENTS:
								if (Input.GetKeyDown(KeyCode.B))
                {
                    byte[] data = {0x32};// stop
                    _serialPort.Write(data,0,1);
                    Debug.Log("WAIT START");
                    taskState = TaskState.WAIT_START;
                }
								if (_serialPort.BytesToRead > 0)
                {
                    int numData = _serialPort.Read(buffer, 0, 128);
                    Debug.Log(System.Text.Encoding.ASCII.GetString(buffer));
                }
								break;
						default:
                Debug.Log("State Error");
								break;
        }
    }
}
```
¿Recuerdas las preguntas presentadas en el experimento anterior? **¿Aquí nos pasa lo mismo?**
####
No, pues una vez se presiona la `techa A` y se entra al **estado** `WAIT_START`, el pc empieza un conteo indefinido hasta que se presione la `tecla B`, la cual hará que el conteo pare y solo continue hasta que vuelva a presionarse desde la `tecla A`. El objeto de texto sin embargo, en ambos programas inicia apenas inicia el programa y no se detiene hasta que este acabe.

# Ejercicio 4
Ahora vas a analizar cómo puedes resolver el problema anterior. Puntualmente, analiza el siguiente programa del controlador:
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
		constexpr uint8_t button1Pin = 12;
		constexpr uint8_t button2Pin = 13;
		constexpr uint8_t button3Pin = 32;
		constexpr uint8_t button4Pin = 33;

		switch (taskState)
	  {
				case TaskStates::INIT:
				{
						Serial.begin(115200);
						pinMode(led, OUTPUT);
						digitalWrite(led, LOW);
						pinMode(button1Pin, INPUT_PULLUP);
						pinMode(button2Pin, INPUT_PULLUP);
						pinMode(button3Pin, INPUT_PULLUP);
						pinMode(button4Pin, INPUT_PULLUP);
						taskState = TaskStates::WAIT_COMMANDS;
						break;
				}
				case TaskStates::WAIT_COMMANDS:
				{
						if (Serial.available() > 0)
						{
								String command = Serial.readStringUntil('\n');
								if (command == "ledON")
								{
										digitalWrite(led, HIGH);
								}
								else if (command == "ledOFF")
								{
										digitalWrite(led, LOW);
								}
								else if (command == "readBUTTONS")
							  {
										Serial.print("btn1: ");
						        Serial.print(btnState(digitalRead(button1Pin)).c_str());
						        Serial.print(" btn2: ");
						        Serial.print(btnState(digitalRead(button2Pin)).c_str());
						        Serial.print(" btn3: ");
						        Serial.print(btnState(digitalRead(button3Pin)).c_str());
						        Serial.print(" btn4: ");
						        Serial.print(btnState(digitalRead(button4Pin)).c_str());
						        Serial.print('\n');
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
A continuación, analiza el siguiente programa del PC:
``` c++
using UnityEngine;
using System.IO.Ports;
using TMPro;

enum TaskState
{
    INIT,
    WAIT_COMMANDS
}

public class Serial : MonoBehaviour //Tenía el class Serial pegado
{
		private static TaskState taskState = TaskState.INIT;
		private SerialPort _serialPort;
		private byte[] buffer;
		public TextMeshProUGUI myText;
		private int counter = 0;

		void Start()
    {
				_serialPort =new SerialPort();
        _serialPort.PortName = "COM4";
        _serialPort.BaudRate = 115200;
        _serialPort.DtrEnable =true;
        _serialPort.NewLine = "\n";
        _serialPort.Open();
        Debug.Log("Open Serial Port");
        buffer =new byte[128];
    }

		void Update()
    {
        myText.text = counter.ToString();
        counter++;

				switch (taskState)
        {
						case TaskState.INIT:
		            taskState = TaskState.WAIT_COMMANDS;
                Debug.Log("WAIT COMMANDS");
								break;
						case TaskState.WAIT_COMMANDS:
								if (Input.GetKeyDown(KeyCode.A))
                {
		                _serialPort.Write("ledON\n");
                    Debug.Log("Send ledON");
                }
								if (Input.GetKeyDown(KeyCode.S))
                {
                    _serialPort.Write("ledOFF\n");
                    Debug.Log("Send ledOFF");
                }
								if (Input.GetKeyDown(KeyCode.D))
                {
                    _serialPort.Write("readBUTTONS\n");
                    Debug.Log("Send readBUTTONS");
                }
								if (_serialPort.BytesToRead > 0)
                {
                    string response = _serialPort.ReadLine();
                    Debug.Log(response);
                }
								break;
						default:
                Debug.Log("State Error");
								break;
        }
    }
}
```
Todo bien -b

# Ejercicio 5
Con todo lo que has aprendido hasta ahora, vas a volver a darle una mirada al material expuesto desde el **ejercicio 1** hasta el **ejercicio 4**. Una iteración más cae bien. Pero la idea de este ejercicio es que le *expliques a un compañero cada ejercicio mientras tu compañero será hacerte preguntas*. Después se invierten los papeles.

# RETO BONIFICACIÓN
El reto consiste en implementar un sistema que permita, mediante una *interfaz gráfica* en Unity interactuar con el controlador. 
####
La idea será que puedas leer el **estado de una variable** que estará cambiando en el controlador y cambiar el **estado** del `LED verde` del controlador. Ten presente que aunque este ejercicio usa un controlador simple, los conceptos asociados a su manejo pueden fácilmente extrapolarse a dispositivos y sistemas más complejos.

**Protocolo de comunicación:**

- El PC SIEMPRE inicia la comunicación solicitando información al controlador. Es decir, desde la aplicación del PC siempre se solicita información y el controlador responde.
- Desde el PC se enviarán tres solicitudes: `read`, `outON`, `outOFF`.
- Para enviar los comandos anteriores usarás los botones de la interfaz de usuario.
- El controlador enviará los siguientes mensajes de respuesta a cada solicitud:
    - Respuesta a `read`: `estadoContador,estadoLED`. Por ejemplo, una posible respuesta será: `235,OFF`. Quiere decir que el contador está en 235 y el LED está apagado.
    - Respuesta a `outON` y `outOFF`: `estadoLED`. Es decir, el controlador recibe el comando, realiza la orden solicitada y devuelve el estado en el cual quedó el LED luego de la orden.
- No olvides que DEBES terminar TODOS los mensajes con el carácter NEWLINE (`\n`) para que ambas partes sepan que el mensaje está completo.
###
Código del controlador:
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
    constexpr uint8_t button1Pin = 12;
    constexpr uint8_t button2Pin = 13;
    constexpr uint8_t button3Pin = 32;
    constexpr uint8_t button4Pin = 33;

    switch (taskState)
    {
        case TaskStates::INIT:
        {
            Serial.begin(115200);
            pinMode(led, OUTPUT);
            digitalWrite(led, LOW);
            pinMode(button1Pin, INPUT_PULLUP);
            pinMode(button2Pin, INPUT_PULLUP);
            pinMode(button3Pin, INPUT_PULLUP);
            pinMode(button4Pin, INPUT_PULLUP);
            taskState = TaskStates::WAIT_COMMANDS;
            break;
        }
        case TaskStates::WAIT_COMMANDS:
        {
            if (Serial.available() > 0)
            {
                String command = Serial.readStringUntil('\n');
                if (command == "ledON")
                {
                    digitalWrite(led, HIGH);
                }
                else if (command == "ledOFF")
                {
                    digitalWrite(led, LOW);
                }
                else if (command == "readBUTTONS")
                {
                    Serial.print("btn1: ");
                    Serial.print(btnState(digitalRead(button1Pin)).c_str());
                    Serial.print(" btn2: ");
                    Serial.print(btnState(digitalRead(button2Pin)).c_str());
                    Serial.print(" btn3: ");
                    Serial.print(btnState(digitalRead(button3Pin)).c_str());
                    Serial.print(" btn4: ");
                    Serial.print(btnState(digitalRead(button4Pin)).c_str());
                    Serial.print('\n');
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
Código del PC:
``` c++
using UnityEngine;
using System.IO.Ports;
using TMPro;
using static UnityEditor.Experimental.AssetDatabaseExperimental.AssetDatabaseCounters;
using System.Threading.Tasks;
using System;

enum TaskState
{
    INIT,
    WAIT_COMMANDS
}

public class Serial : MonoBehaviour
{

private static TaskState taskState = TaskState.INIT;
private SerialPort _serialPort;
private byte[] buffer;

public TextMeshProUGUI myText;
private int counter = 0;

    public GameObject ledOn; static bool led;
    public TextMeshProUGUI textbox; private string box;

    void Start()
    {
    _serialPort = new SerialPort();
    _serialPort.PortName = "COM4";
    _serialPort.BaudRate = 115200;
    _serialPort.DtrEnable = true;
    _serialPort.NewLine = "\n";
    _serialPort.Open();
    Debug.Log("Open Serial Port");
    buffer = new byte[128];

        ledOn.SetActive(false); led = false;
    }

void Update()
{
    myText.text = counter.ToString();
    counter++;

        textbox.text = box;

    switch (taskState)
    {
        case TaskState.INIT:
            taskState = TaskState.WAIT_COMMANDS;
            Debug.Log("WAIT COMMANDS");
            box = "WAIT COMMANDS";
            break;
        case TaskState.WAIT_COMMANDS:
            if (_serialPort.BytesToRead > 0)
            {
                string response = _serialPort.ReadLine();
                Debug.Log(response);
                //box = response;
            }
            break;
        default:
            Debug.Log("State Error");
            box = "State Error" ;
            break;
    }
}
    public void ON()
    {
        _serialPort.Write("ledON\n");
        Debug.Log("LED = ON\n");
        box = "LED = ON\n"; ledOn.SetActive(true); led = true;
    }
    public void OFF()
    {
        _serialPort.Write("ledOFF\n");
        Debug.Log("LED = OFF\n");
        box = "LED = OFF\n"; ledOn.SetActive(false); led = false;
    }
    public void READ()
    {
        _serialPort.Write("readBUTTONS\n");
        Debug.Log(counter + "," + (led == true ? "ON" : "OFF") + "\n");
        Debug.Log("Send readBUTTONS\n");

        box = counter + "," + (led == true ? "ON" : "OFF") + "\n";
        //box = "Send readBUTTONS\n" ;
    }
}
```
#### Interfaz
![image](https://github.com/user-attachments/assets/b0e9a1e4-1544-4b23-88bb-038eefd4eac8)
####
### *¿Cómo funciona?*
El programa empieza y en en la interfaz se visualizan seis elementos principales:
- En la esquina superior, está el `contador`.
- En la parte inferior, está el `cuadro de texto` por donde se imprimirán los mensajes indicados.
- Encima de este, están los tres *botones* que representan las funciones `ON`, `OFF` y `READ` respectivamente.
- El cuadrado encima de estos, es una `representación del led`, el cual cambiará cada que se presionen los botones de `ON` u `OFF`.
####
Cuando el programa inicia, lo único que se verá será el contador subiendo, la representación del led apagado, y en el cuadro de texto la oración **"WAIT COMMANDS"**. Si se presiona el botón `ON`, el led se encenderá, y en el cuadro de texto dirá **"LED = ON"**; y se se presiona `OFF`, viceversa. Al presionar `READ` sin embargo, el cuadro de texto mostrará el número en el que se encontraba el contador al presionar dicho botón, y al lado el estado actual del led (ON/OFF).

# TRABAJO FINAL
Tu equipo ha sido contratado para desarrollar una aplicación del área de educación financiera en la modalidad de **Escape Room**, donde una aplicación correrá en el PC y la otra en el microcontrolador.  Debes relacionar *variables* de tipo *ambiental* con variables de tipo *psicológico* que permitan aprendizajes financieros.
####
Debes construir una **narrativa** de la experiencia y detallar lo que el usuario **debe realizar** y debe tener un propósito de aprendizaje.
####
### **La aplicación en el PC debe:**
- *Interactuar* con el usuario por medio de elementos de interfaz de usuario, tales como `botones`, `cajas de texto`, `textos en pantalla`, entre otros. Nota: Solo se debe usar la *consola para depuración y no para interacción*.
- Leer el **estado** de *tres (3) variables* de la aplicación del microcontrolador.
- La aplicación del PC deberá **solicitar** el *valor* y *estado* de las variables a través de una `única instrucción` y el microcontrolador debe **reportar** toda la información de las tres variables en un `único mensaje`… Ojo, no por separado, en una misma trama.
- Permitir configurar la *velocidad* a la cual **cambiará la variable** y si debe cambiar o no, como también, permitir **definir** el *valor inicial* de la variable.

### **La aplicación en el microcontrolador debe:**
- Verificar si debe *cambiar* la variable y modificarla en tiempo real, siempre y cuando **esté habilitada para cambiar**b. La función de cambio será simplemente `aumentar en uno el valor previo a la velocidad especificada`.
- Mantener un `LED funcionando a una frecuencia de 1 Hz`. El objetivo de este LED es que se pueda *verificar de manera visual* que la aplicación en el microcontrolador NUNCA se bloquea. Esto implica que la **señal de activación se genera en el PC**.
###
_________________________________________________________________________________________________________________________________________________________________________
####
#### Nombre del Escape Room
"El Reto Financiero Familiar"

#### Historia y Objetivo
Los jugadores son hijos que desean pedir dinero a sus padres para una actividad especial (por ejemplo, un viaje, una consola de videojuegos, o un proyecto personal). Sin embargo, sus padres se niegan a darles el dinero hasta que demuestren tener un buen conocimiento sobre la gestión financiera. Los participantes deben superar una serie de desafíos financieros que los padres han preparado para demostrar que pueden manejar el dinero de manera responsable. Solo si completan todas las pruebas con éxito, obtendrán la "aprobación financiera" de sus padres y podrán obtener el dinero.

- *Estructura del Escape Room*
 Habrán 4 etapas, cada una centrada en un aspecto clave de la educación financiera, pero adaptada al contexto de la historia familiar.
###
Código del controlador:
``` c++
Igual al ejercicio 4 (No se pudo conectar apropiadamente otro código)
```
Código del PC:
``` c++
using UnityEngine;
using System.IO.Ports;
using TMPro;
using static UnityEditor.Experimental.AssetDatabaseExperimental.AssetDatabaseCounters;
using System.Threading.Tasks;
using System;
using static Unity.Burst.Intrinsics.Arm;

enum TaskState2
{
    INIT,
    WAIT_COMMANDS
}

public class Serial2 : MonoBehaviour
{

    private static TaskState taskState = TaskState.INIT;
    private SerialPort _serialPort;
    private byte[] buffer;
    
    public TextMeshProUGUI textbox; private string box;
    public GameObject Qbox;
    public TextMeshProUGUI textQ; private string question;

    public GameObject ButtonA; public GameObject ButtonB; public GameObject ButtonC; public GameObject ButtonD; public GameObject ButtonE;

    static bool next; static string[] Atext = new string[40]; static int cont;
    static string[] Btext = new string[20]; static int cont2;

    static char respuesta;

    void Start()
    {
        _serialPort = new SerialPort();
        _serialPort.PortName = "COM4";
        _serialPort.BaudRate = 115200;
        _serialPort.DtrEnable = true;
        _serialPort.NewLine = "\n";
        _serialPort.Open();
        Debug.Log("Open Serial Port");
        buffer = new byte[128];

        Qbox.SetActive(true); next = false; cont = 0;

        ButtonA.SetActive(false); ButtonB.SetActive(false); ButtonC.SetActive(false); ButtonD.SetActive(false); ButtonE.SetActive(false);

        Atext[0] = "Los jugadores son hijos que desean pedir dinero a sus padres para una actividad especial (por ejemplo, un viaje, una consola de videojuegos, o un proyecto personal).";
        Atext[1] = "Sin embargo, sus padres se niegan a darles el dinero hasta que demuestren tener un buen conocimiento sobre la gestión financiera.";
        Atext[2] = "Los participantes deben superar una serie de desafíos financieros que los padres han preparado para demostrar que pueden manejar el dinero de manera responsable.";
        Atext[3] = "Solo si completan todas las pruebas con éxito, obtendrán la \"aprobación financiera\" de sus padres y podrán obtener el dinero.";
        Atext[4] = ". . . Empezemos.";
        //ETAPA 1
        Atext[5] = "Los jugadores tienen un presupuesto total de 5 millones pesos para el viaje, distribuidos en una tarjeta de débito y cuatro tarjetas de crédito.";
        Atext[6] = "Deben organizar y optimizar sus gastos para aprovechar al máximo su presupuesto y al final de pagar todos los gastos deben de quedar más de 500 mil pesos para emergencias.";
        Atext[7] = "Distribución del Dinero en las Tarjetas (Modificada):";
        Atext[8] = " ¬ Tarjeta de Débito: 2,000,000 COP\n ¬ Tarjeta de Crédito A (Tasa Alta, Sin Beneficios): 500,000 COP\n ¬ Tarjeta de Crédito B (Tasa Media, Cashback en Alimentación): 500,000 COP\n ¬ Tarjeta de Crédito C (Tasa Alta, Descuentos en Viajes): 1,500,000 COP\n ¬ Tarjeta de Crédito D (Tasa Media, Beneficios en Compras en Línea): 500,000 COP\n";
        Atext[9] = "..";
        Atext[10] = "..";
        Atext[11] = " a) Tarjeta de Débito\n b) Tarjeta de Crédito A\n c) Tarjeta de Crédito B\n d) Tarjeta de Crédito C\n e) Tarjeta de Crédito D\n";
        Atext[12] = "Te equivocaste, vuelve a intentar.";
        Atext[13] = "¡Bien hecho! Continuemos. . .";
        //ETAPA 2
        Atext[14] = "Los padres le dan al niño 20,000 COP a la semana como mesada. El niño tiene varios deseos: quiere comprar una bicicleta, un nuevo control para su consola y un juguete bay.";
        Atext[15] = "Sin embargo, debe aprender a ahorrar y planificar sus gastos para poder comprar estos objetos sin gastar más de lo que tiene.";
        Atext[16] = "Los padres le han dicho que debe demostrar una buena capacidad de ahorro antes de darle el dinero.";
        Atext[17] = "Detalles Iniciales:";
        Atext[18] = " ¬ Ingreso semanal: 20,000 COP\r\n ¬ Gastos semanales: Se presentan 4 gastos recurrentes que el niño tiene cada semana.\r\n ¬ Objetos a comprar y sus costos:\r\n ¬ Bicicleta: 100,000 COP\r\n ¬ Control de consola: 70,000 COP\r\n ¬ Bay: 50,000 COP\r\n ¬ Regla de Emergencias: El 20% de lo que no se gaste durante la semana se debe reservar para emergencias y no se puede usar para ahorro.";
        Atext[19] = "..";
        Atext[20] = "..";
        Atext[21] = "Line21";
        Atext[22] = "Line22";
        Atext[23] = "Line23";

        //

        Btext[0] = "El Reto Financiero Familiar";
        Btext[1] = "Etapa 1: El Presupuesto para el Viaje";
        Btext[2] = "Etapa 2: Ahorro para Comprar lo Deseado";
        Btext[3] = "Etapa 3: Decidiendo Dónde Invertir la Mesada";
        Btext[4] = "Line5";
        Btext[5] = "Line6";
    }

    void Update()
    {
        box = Atext[cont];
        textbox.text = box;

        question = Btext[cont2];
        textQ.text = question;

        switch (taskState)
        {
            case TaskState.INIT:
                taskState = TaskState.WAIT_COMMANDS;
                Debug.Log("BEGIN");
                break;
            case TaskState.WAIT_COMMANDS:
                if (_serialPort.BytesToRead > 0)
                {
                    string response = _serialPort.ReadLine();
                    Debug.Log(response);
                }
                break;
            default:
                Debug.Log("State Error");
                box = "ERROR";
                break;
        }

        if (next)
        {
            cont++; next = false;
            if ((cont-1) == 12)
            {
                cont = 5; cont2 = 1;
            }
        }

        if (cont == 5)
        {
            cont2 = 1;
        }
        if (cont == 11)
        {
            ButtonA.SetActive(true); ButtonB.SetActive(true); ButtonC.SetActive(true); ButtonD.SetActive(true); ButtonE.SetActive(true);
        }

        if (cont == 12)
        {
            ButtonA.SetActive(false); ButtonB.SetActive(false); ButtonC.SetActive(false); ButtonD.SetActive(false); ButtonE.SetActive(false);
            switch (respuesta)
            {
                case 'a': cont = 13;  break;
                default: cont = 12; break;
            }
        }
        if (cont == 14)
        {
            cont2 = 2;
        }
    }

    public void A()
    {
        respuesta = 'a'; Debug.Log("respuesta = " + respuesta);
    }
    public void B()
    {
        respuesta = 'b'; Debug.Log("respuesta = " + respuesta);
    }
    public void C()
    {
        respuesta = 'c'; Debug.Log("respuesta = " + respuesta);
    }
    public void D()
    {
        respuesta = 'd'; Debug.Log("respuesta = " + respuesta);
    }
    public void E()
    {
        respuesta = 'e'; Debug.Log("respuesta = " + respuesta);
    }
    public void NEXT()
    {
        next = true; Debug.Log("Passed to next -> text");
    }
}
```
Versión del Unity package:
``` c++
using UnityEngine;
using System.IO.Ports;
using TMPro;
using static UnityEditor.Experimental.AssetDatabaseExperimental.AssetDatabaseCounters;
using System.Threading.Tasks;
using System;
// UnityEngine.UI;
using UnityEngine.Apple;

enum TaskState2
{
    INIT,
    WAIT_COMMANDS
}



public class Serial2 : MonoBehaviour
{

    // Añadir una referencia a changeselect
    public changeselect changeSelectInstance;

    private static TaskState2 taskState = TaskState2.INIT;
    private SerialPort _serialPort;
    private byte[] buffer;

    public TextMeshProUGUI textbox; private string box;
    public GameObject Qbox;
    public TextMeshProUGUI textQ; private string question;

    public GameObject ButtonA; public GameObject ButtonB; public GameObject ButtonC; public GameObject ButtonD; public GameObject ButtonE;

    static bool next; static string[] Atext = new string[50];
    private static int cont; 
    static string[] Btext = new string[20]; static int cont2;

    static char respuesta;

    public TextMeshProUGUI timer;
    private int s = 0; private float sm = 0; private int m = 0; private int countdown = 0; private string cero = "0";

    void Start()
    {
        _serialPort = new SerialPort();
        _serialPort.PortName = "COM4";
        _serialPort.BaudRate = 115200;
        _serialPort.DtrEnable = true;
        _serialPort.NewLine = "\n";
        _serialPort.Open();
        Debug.Log("Open Serial Port");
        buffer = new byte[128];

        Qbox.SetActive(true); next = false; cont = 0;

        ButtonA.SetActive(false); ButtonB.SetActive(false); ButtonC.SetActive(false); ButtonD.SetActive(false); ButtonE.SetActive(false);

        //
        // Función para enviar el dato usuario a través del puerto serial



        //

        Atext[0] = "Los jugadores son hijos que desean pedir dinero a sus padres para una actividad especial (por ejemplo, un viaje, una consola de videojuegos, o un proyecto personal).";
        Atext[1] = "Sin embargo, sus padres se niegan a darles el dinero hasta que demuestren tener un buen conocimiento sobre la gestión financiera.";
        Atext[2] = "Los participantes deben superar una serie de desafíos financieros que los padres han preparado para demostrar que pueden manejar el dinero de manera responsable.";
        Atext[3] = "Solo si completan todas las pruebas con éxito, obtendrán la \"aprobación financiera\" de sus padres y podrán obtener el dinero.";
        Atext[4] = ". . . Empecemos";

        //ETAPA 1
        Atext[5] = "Presupuesto Total y Métodos de Pago:\r\nLos participantes tendrán su presupuesto total dividido en tres tarjetas, cada una con diferentes características. Deben elegir cuál usar para cada compra, tomando en cuenta los beneficios o desventajas que ofrece cada método de pago.";
        Atext[6] = "Tarjeta de Débito:\r\nCantidad disponible: 500.000 COP\r\nCaracterísticas: El dinero sale inmediatamente del presupuesto sin ningún costo adicional.";
        Atext[7] = "Tarjeta de Crédito (con descuento en algunas compras):\r\nCantidad disponible: 400.000 COP\r\nCaracterísticas: Esta tarjeta ofrece un 10% de descuento en las compras de Entretenimiento y Souvenirs.";
        Atext[8] = "Tarjeta de Crédito (sin descuento, pero con intereses bajos):\r\nCantidad disponible: 300.000 COP\r\nCaracterísticas: No ofrece descuentos, pero tiene intereses bajos (5%) si se usa.";
        
        Atext[9] = "Tiempo Base:\r\nTemperatura normal (<30°C): Tienen 5 minutos (300 segundos) para completar la etapa.\r\nTemperatura alta (>30°C): El tiempo se reduce a 3 minutos (180 segundos) debido a la irritabilidad y presión ambiental.";
        Atext[10] = "Modificaciones al Tiempo:\r\nCompra de refrescos: Cada refresco comprado añade 10 segundos adicionales al tiempo total, permitiendo tomar decisiones más calmadas.";
        Atext[11] = "Condiciones de Éxito:\r\nDeben gestionar bien los tres métodos de pago para dejar al menos 500.000 COP para emergencias.\r\nSi usan mal las tarjetas o no aprovechan los descuentos, podrían exceder el presupuesto o generar intereses innecesarios.";
        Atext[12] = "Lista de Compras:\r\nLos participantes deben asignar su presupuesto a diferentes necesidades y lujos durante el viaje. Dependiendo de sus decisiones, podrían quedarse sin suficiente dinero para las compras importantes.";

        Atext[13] = "Comida básica:\r\nPrecio: 300.000 COP (pago con cualquier tarjeta).\r\nDescripción: Necesaria para todo el viaje. Es obligatorio que la compren.";
        Atext[14] = "Bebidas refrescantes:\r\nPrecio por refresco: 50.000 COP (pago con cualquier tarjeta).\r\nDescripción: Opcional. Los participantes pueden decidir cuántos refrescos comprar. Cada refresco comprado otorga 10 segundos adicionales al tiempo.";
        Atext[15] = "Entretenimiento (actividades recreativas):\r\nPrecio: 400.000 COP (10% de descuento si se paga con la tarjeta de crédito con descuentos, lo que reduce el costo a 360.000 COP).\r\nDescripción: Actividades recreativas, opcional. Solo se obtiene el descuento si se usa la tarjeta con beneficio.";
        Atext[16] = "Souvenirs:\r\nPrecio: 200.000 COP (10% de descuento si se paga con la tarjeta de crédito con descuentos, lo que reduce el costo a 180.000 COP).\r\nDescripción: Opcional.";
        Atext[17] = "Transporte local:\r\nPrecio: 300.000 COP (pago con cualquier tarjeta).\r\nDescripción: Necesario para moverse durante el viaje. Es obligatorio.";
        Atext[18] = "..";
        Atext[19] = "..";
        Atext[20] = "..";
        Atext[21] = " a) Tarjeta de Débito\n b) ..";
        Atext[22] = "Te equivocaste, vuelve a intentar.";
        Atext[23] = "¡Bien hecho! Continuemos. . .";

        //ETAPA 2
        Atext[20] = "Los participantes deben revisar una serie de ofertas de crédito e inversión.";
        Atext[20] = "La tarea parece simple, pero el ambiente de trabajo puede influir en su capacidad de concentración. La iluminación ambiental juega un papel crucial: demasiada luz o muy poca puede afectar tanto el tiempo como la calidad de las decisiones financieras que tomen.";
        Atext[20] = "En condiciones de baja luz, los jugadores comienzan a experimentar fatiga visual y falta de concentración, lo que lleva a más distracción y mayor probabilidad de errores.";
        
        Atext[20] = "Condiciones del entorno:";
        Atext[20] = "Luz adecuada:\r\nDescripción: La iluminación es perfecta para la tarea. Los jugadores tienen un ambiente cómodo donde pueden concentrarse mejor en los detalles de las ofertas financieras.\r\nTiempo disponible: 7 minutos para analizar las ofertas de crédito e inversión.";
        Atext[20] = "Luz baja:\r\nDescripción: La iluminación es insuficiente, lo que genera fatiga visual y distracción. Si bien tienen más tiempo, la calidad de su concentración disminuye significativamente.\r\nTiempo disponible: 9 minutos para analizar las ofertas, pero con una penalización de fatiga que aumenta la probabilidad de errores.";
        Atext[20] = "Los participantes recibirán una tabla con diferentes ofertas de crédito e inversiones , y deberán seleccionar las mejores opciones en función de los intereses, plazos y riesgos involucrados. Sin embargo, las ofertas tienen detalles que pueden pasar por alto si no están lo suficientemente concentradas.";
        
        Atext[20] = "Ofertas para analizar:";
        Atext[20] = "Crédito personal:\r\nInterés bajo pero largo plazo (10 años).\r\nIdeal para aquellos que no necesitan el dinero inmediatamente pero quieren reducir los pagos mensuales.";
        Atext[20] = "Préstamo Rápido:\r\nInterés alto, plazo corto (6 meses).\r\nConveniente para emergencias, pero con un costo elevado en intereses.\r\nInversión de Alto Riesgo: Alto potencial de retorno.\r\nIdeal para quienes obtienen ganancias rápidas, pero con un riesgo considerable de pérdida.";
        Atext[20] = "Inversión Conservadora:\r\n\r\nRetorno seguro, bajo crecimiento.\r\nPerfecta para aquellos que prefieren no arriesgar su capital.";
        Atext[20] = "Los jugadores deben elegir el mejor equilibrio entre riesgo y beneficio, tomando decisiones basadas en la información que tienen.";
        Atext[20] = "..";
        Atext[20] = "..";

        //ETAPA 3
        Atext[20] = "Los jugadores deben gestionar su presupuesto mensual para ahorrar lo suficiente y comprar un artículo importante.";
        Atext[20] = "Sin embargo, la tarea se complica por la presión ambiental causada por los niveles de humedad, lo que afecta su capacidad para tomar decisiones rápidas al cubrir sus necesidades básicas.";
        Atext[20] = "Artículo para comprar:\r\nArtículo Deseado: Un teléfono móvil de 250.000 COP .\r\nMeta de ahorro: Los jugadores deben ahorrar 30.000 COP por mes para alcanzar la meta en 9 meses, pero cualquier gasto innecesario afectará su capacidad de ahorro.";
        Atext[20] = "Variables:";
        Atext[20] = "Variable Ambiental: Nivel de humedad (medido con un sensor de humedad).\r\nSi la humedad es alta, los jugadores sienten incomodidad, lo que limita su capacidad de tomar decisiones de manera óptima.";
        Atext[20] = "Condiciones del entorno:";
        Atext[20] = "Humedad moderada (baja humedad):\r\nDescripción: El ambiente es cómodo, con humedad en un nivel óptimo.\r\nTiempo para decidir el precio de los gastos básicos: 6 segundos .";
        Atext[20] = "Humedad alta:\r\nDescripción: El ambiente es sofocante y la humedad genera incomodidad, lo que afecta la capacidad de pensar claramente.\r\nTiempo para decidir el precio de los gastos básicos: 3 segundos , lo que incrementa el riesgo de elegir un precio más alto ";
        Atext[20] = "El objetivo sigue siendo ahorrar lo suficiente cada mes para comprar un artículo importante, pero cada vez que los jugadores se enfrenten a un gasto esencial (como alimentación, transporte o entretenimiento), se les presentarán tres posibles precios.y tendrán";
        
        Atext[20] = "Gastos Esenciales:\r\n";
        Atext[20] = "Alimentación:\r\nOpción 1: 35.000 COP\r\nOpción 2: 40.000 COP\r\nOpción 3: 45.000 COP";
        Atext[20] = "Transporte:\r\nOpción 1: 15.000 COP\r\nOpción 2: 20.000 COP\r\nOpción 3: 25.000 COP\r\n";
        Atext[20] = "Entretenimiento:\r\nOpción 1: 8.000 COP\r\nOpción 2: 10.000 COP\r\nOpción 3: 12.000 COP";
        Atext[20] = "..";
        Atext[20] = "..";

        //ETAPA 4
        Atext[20] = "..";
        Atext[20] = "..";
        Atext[20] = "..";
        Atext[20] = "..";
        Atext[20] = "..";
        Atext[20] = "..";
        Atext[20] = "..";

        //

        Btext[0] = "El Reto Financiero Familiar";
        Btext[1] = "Etapa 1: El Clima y la Toma de Decisiones";
        Btext[2] = "Etapa 2: Luz y concentración";
        Btext[3] = "Etapa 3: Humedad y Decisiones bajo Presión";
        Btext[4] = "Etapa 4: . . .";
        Btext[5] = "¡Prueba terminada!";
    }

    void Update()
    {
        changeSelectInstance.SetCount(cont);
        // Reciba cosas del Port
        if (_serialPort.IsOpen && _serialPort.BytesToRead > 0)
        {
            string response = _serialPort.ReadLine();
            Debug.Log(response);
        }

        // Contador de tiempo
        if (cont == 21 || cont == 40)
        {
            timer.text = m.ToString() + ":" + cero + s.ToString();
            sm += Time.deltaTime; s = Convert.ToInt32(sm);

            int timeLeft = Mathf.Max(0, Mathf.FloorToInt(countdown - s));

            if (s > 9.99)
            {
                cero = "";
            }
            if (s > 59.99)
            {
                m++; sm = 0; cero = "0";
            }
        }
        else
        {
            sm = 0; cero = "0"; m = 0;
            s = Convert.ToInt32(sm);
            timer.text = m.ToString() + ":" + cero + s.ToString();
        }

        // Cajas de texto
        box = Atext[cont];
        textbox.text = box;

        question = Btext[cont2];
        textQ.text = question;

        // TASKSTATE
        switch (taskState)
        {
            case TaskState2.INIT:
                taskState = TaskState2.WAIT_COMMANDS;
                Debug.Log("BEGIN");
                break;
            case TaskState2.WAIT_COMMANDS:
                if (_serialPort.BytesToRead > 0)
                {
                    string response = _serialPort.ReadLine();
                    Debug.Log(response);
                }
                break;
            default:
                Debug.Log("State Error");
                box = "ERROR";
                break;
        }

        // NEXT()
        if (next)
        {
            cont++; next = false;
            /*if ((cont - 1) == 12)
            {
                cont = 5; cont2 = 1;
            }*/

            Debug.Log("Passed to next -> text : " + cont);
        }

        if (cont == 30)
        {
            cont2 = 1;
        }
        if (cont == 21)
        {
            ButtonA.SetActive(true); ButtonB.SetActive(true); ButtonC.SetActive(true); ButtonD.SetActive(true); ButtonE.SetActive(true);
        }

        if (cont == 22)
        {
            ButtonA.SetActive(false); ButtonB.SetActive(false); ButtonC.SetActive(false); ButtonD.SetActive(false); ButtonE.SetActive(false);
            switch (respuesta)
            {
                case 'a': cont = 13; break;
                default: cont = 12; break;
            }
        }
        if (cont == 30)
        {
            cont2 = 2;
        }
    }

    public void EnviarDatoUsuario(string datousuario)
    {
        if (_serialPort.IsOpen)
        {
            _serialPort.WriteLine(datousuario);
            Debug.Log("Dato enviado: " + datousuario);
        }
        else
        {
            Debug.LogError("Puerto serial no está abierto");
        }
    }

    private void OnDestroy()
    {
        if (_serialPort.IsOpen)
        {
            _serialPort.Close();
            Debug.Log("Puerto serial cerrado");
        }
    }

    public void A()
    {
        respuesta = 'a'; Debug.Log("respuesta = " + respuesta);
    }
    public void B()
    {
        respuesta = 'b'; Debug.Log("respuesta = " + respuesta);
    }
    public void C()
    {
        respuesta = 'c'; Debug.Log("respuesta = " + respuesta);
    }
    public void D()
    {
        respuesta = 'd'; Debug.Log("respuesta = " + respuesta);
    }
    public void E()
    {
        respuesta = 'e'; Debug.Log("respuesta = " + respuesta);
    }
    public void NEXT()
    {
        next = true;
    }
}
```
#### Interfaz
![image](https://github.com/user-attachments/assets/9cfbc5d9-6292-4478-9abc-98302b5b4cd5)
(todo activo)
####
### *¿Cómo planeamos que funcionará?*
Al darle `Start`, se usará el botón de '->' para pasar el texto en el `text box`, en el cuál le dará toda la información. El juego se divide en **4 unidades**, y al acabar cada unidad, se hará una pregunta y se activarán los botones de respuesta: `a`, `b`, `c`, `d` y `e`. Al mismo tiempo, se activará un temporizador. Si el tiempo se acaba o termina el tiempo, se empieza la unidad actual denuevo. Si la respuesta es correcta, pasa a la siguiente.

Además, antes de responder las preguntas, se activará la caja de *`ingresar texto...`*, esta será para ingresar qué variables se cambiarán desde Unity al controlador, y luego se procederá a responder las preguntas.

El programa termina cuando se acaban las 4 unidadaes.



