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
Tu equipo ha sido contratado para desarrollar una aplicación del área de educación financiera en la modalidad de **Scape Room**, donde una aplicación correrá en el PC y la otra en el microcontrolador.  Debes relacionar *variables* de tipo *ambiental* con variables de tipo *psicológico* que permitan aprendizajes financieros.
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
Código del controlador:
``` c++

```
Código del PC:
``` c++

```
#### Interfaz

####
### *¿Cómo funciona?*
...




