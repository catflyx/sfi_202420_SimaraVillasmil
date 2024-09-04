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
        _serialPort.PortName = "COM3";
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
- ¿Por qué es importante considerar las propiedades *PortName* y *BaudRate*?
texto.
####
- ¿Qué relación tienen las propiedades anteriores con el controlador?
texto.

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
        _serialPort.PortName = "COM3";
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
		    _serialPort.PortName = "COM3";
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

####
- Por ejemplo, ¿Qué pasaría si al momento de ejecutar la instrucción `int numData = _serialPort.Read(buffer, 0, 20);` solo han llegado *10* de los *16* bytes del mensaje?

####
- ¿Cómo puede hacer tu programa para saber que ya tiene el mensaje completo?

####
- ¿Cómo se podría garantizar que antes de hacer la operación Read tenga los 16 bytes listos para ser leídos?

####
- Además, si los mensajes que envía el controlador tienen tamaños diferentes, ¿Cómo haces para saber que el mensaje enviado está completo o faltan bytes por recibir?


# Ejercicio 3



# Ejercicio 4



# Ejercicio 5


# RETO BONIFICACIÓN
