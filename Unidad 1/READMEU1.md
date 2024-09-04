# U N I D A D  1
Software para sistemas embebidos
_________________________________________________________________________________________________________________________________________________________________________________________
### ¿Qué sucedio al cambiar el 100 por el 500?
El led en vez de apagar y prenderse bastante rápido, tenía pausas más alargadas entre estar prendido y entre estar apagado. Es decir, los lapsos de prender y apagarse se alargaron.

# Documentación | Ejercicio 5
El núcleo Arduino-Pico es un conjunto de herramientas y bibliotecas desarrollado por la comunidad para programar la placa Raspberry Pi Pico (RP2040) utilizando el entorno de Arduino. Está diseñado para hacer más fácil y divertido el uso de estas placas, integrándolas en el ecosistema de Arduino.

Este núcleo utiliza un conjunto de herramientas personalizado que incluye GCC 10.3 y Newlib 4.0.0, y no requiere la instalación de software adicional en tu sistema.

### Raspberry Pi Pico

La **Raspberry Pi Pico** es una placa de microcontrolador de bajo costo y alto rendimiento con interfaces digitales flexibles. Sus principales características son:

- **Microcontrolador**: Chip RP2040 diseñado por Raspberry Pi en el Reino Unido.
- **Procesador**: Dual-core Arm Cortex M0+ con un reloj flexible que puede alcanzar hasta 133 MHz.
- **Memoria**: 264kB de SRAM y 2MB de memoria flash a bordo.
- **Conectividad**: USB 1.1 con soporte para dispositivo y host.
- **Modos de bajo consumo**: Soporte para modos de suspensión y dormancia.
- **Programación fácil**: Programación mediante arrastrar y soltar utilizando almacenamiento masivo a través de USB.
- **GPIO**: 26 pines GPIO multifuncionales.
- **Interfaces**: 2 × SPI, 2 × I2C, 2 × UART, 3 × ADC de 12 bits, 16 canales PWM controlables.
- **Otros componentes**: Reloj y temporizador precisos en chip, sensor de temperatura, y bibliotecas de punto flotante aceleradas en chip.
- **Soporte PIO**: 8 máquinas de estado PIO programables para soporte de periféricos personalizados.

La Raspberry Pi Pico viene como un módulo con bordes castellados para soldar directamente a placas portadoras. La variante **Pico H** incluye encabezados pre-soldados y un conector de dep.

# Ejercicio 6
``` C++
void task1()
{
    // Definición de estados y variable de estado
    enum class Task1States
    {
        INIT,
        WAIT_TIMEOUT
    };
    static Task1States task1State = Task1States::INIT;

    // Definición de variables static (conservan su valor entre llamadas a task1)
    static uint32_t lastTime = 0;

    // Constantes
    constexpr uint32_t INTERVAL = 1000;

    // MÁQUINA de ESTADOS
    switch (task1State)
    {
    case Task1States::INIT:
    {
        // Acciones:
        Serial.begin(115200);

        // Garantiza los valores iniciales para el siguiente estado.
        lastTime = millis();
        task1State = Task1States::WAIT_TIMEOUT;

        Serial.print("Task1States::WAIT_TIMEOUT\n");

        break;
    }
    case Task1States::WAIT_TIMEOUT:
    {
        uint32_t currentTime = millis();

        // Evento
        if ((currentTime - lastTime) >= INTERVAL)
        {
            // Acciones:
            lastTime = currentTime;
            Serial.print(currentTime);
            Serial.print('\n');
        }
        break;
    }
    default:
    {
        Serial.println("Error");
    }
    }
}

void setup()
{
    task1();
}

void loop()
{
    task1();
}
```
### ¿Cómo se ejecuta este programa?

El programa define una máquina de estados que se ejecuta dentro de una función llamada `task1()`. Esta función se llama tanto en `setup()` como en `loop()` en el entorno de Arduino, lo que significa que se ejecutará repetidamente durante el tiempo que la placa esté encendida. La máquina de estados tiene dos estados principales: `INIT` y `WAIT_TIMEOUT`.

1. **Estado `INIT`**:
   - Se inicializa la comunicación serial con `Serial.begin(115200)`.
   - Se establece el tiempo inicial `lastTime` usando `millis()`.
   - Se cambia el estado a `WAIT_TIMEOUT` y se imprime el mensaje `"Task1States::WAIT_TIMEOUT\n"` por el puerto serial.
   - Este estado se ejecuta solo una vez ya que no se cambia de estado por el resto del programa.

2. **Estado `WAIT_TIMEOUT`**:
   - En cada iteración del `loop()`, se comprueba si ha transcurrido el intervalo de tiempo definido por `INTERVAL` (1000 ms o 1 segundo).
   - Si ha transcurrido el intervalo, se actualiza `lastTime` y se imprime el valor de `currentTime` en el monitor serie.
   - Este estado se ejecuta repetidamente cada vez que se cumple el intervalo de 1 segundo.

### ¿Pudiste ver este mensaje: `Serial.print("Task1States::WAIT_TIMEOUT\n");`. ¿Por qué crees que ocurre esto?

No, y esto no es porque el mensaje no se haya imprimido. Lo que sucede es que el mensaje se envía tan pronto el programa inicia, es decir, incluso antes que pase el primer segundo, y ya que el monutor serie toca volverlo a abrir cada vez que se vuelve a iniciarlizar el programa, el mensaje se pierde al estar en todo el inicio; solo alcanzándose a ver los mensajes a partir del primer o segundo segundo transcurrido desde que se inicializó el programa.

### ¿Cuántas veces se ejecuta el código en el case Task1States::INIT?

Una vez.

# Ejercicio 7
Observa la función 
```c++
millis(); 
```
¿Para qué sirve?
###
Esta devuelve la cantidad de milisegundos que han transcurrido desde que se inicio el programa.


# Ejercicio 8
Realiza un programa que envíe un mensaje al pasar un segundo, dos segundos y tres segundos. Luego de esto debe volver a comenzar.
```c++
   void mess()
{
    // Definición de estados y variable de estado
    enum class MessSTates
    {
        ONE,
        TWO,
        THREE
    };
    static MessSTates state = MessSTates::ONE;
    static uint32_t previousTime = millis();
    uint32_t currentTime = millis();

    // MÁQUINA de ESTADOS
    switch (state)
    {
    case MessSTates::ONE:
    {
        if ((currentTime - previousTime) >= 1000)
        {
            Serial.print("UNO SECOND"); Serial.print('\n');
            previousTime = currentTime;
            state = MessSTates::TWO;
        }
        break;
    }
    case MessSTates::TWO:
    {
        if ((currentTime - previousTime) >= 1000)
        {
            Serial.print("DOS SECOND"); Serial.print('\n');
            previousTime = currentTime;
            state = MessSTates::THREE;
        }        
        break;
    }
    case MessSTates::THREE:
    {
        if ((currentTime - previousTime) >= 1000)
        {
            Serial.print("TRES SECOND"); Serial.print('\n'); Serial.print('\n');
            previousTime = currentTime;
            state = MessSTates::ONE;
        }
        break;
    }
    default:
    {
        Serial.println("Error");
    }
    }
}

void setup()
{
    Serial.begin(9600);
}

void loop()
{
    mess();
}
```
En el README.md del repositorio responde:
- ¿Cuáles son los estados del programa?
#### 
   - ONE: Para el mensaje de un segundo, y llamar al estado `TWO`.
   - TWO: Para el mensaje de dos segundos, y llamar el estado `THREE`.
   - THREE: Para el mensaje de tres segundos, y llamar el estado `ONE`, para que sea un bucle.

- ¿Cuáles son los eventos?
#### 
Son los `if` incorporados en cada estado, por los cuales se imprime el mensaje y se reasigna el estado de `state`.

- ¿Cuáles son las acciones?
#### 
`Serial.begin(9600)` son las acciones presentes en el programa.

# Ejercicio 9
Te voy a mostrar el código para la task1 y luego con tu equipo vas a construir las demás tareas. La frecuencia del mensaje será de 1 Hz.

El objetivo es que hagas un programa donde tengas 3 tareas. La tarea 1 enviará un mensaje a 1 Hz., la tarea 2 a 0.5 Hz., la tarea 3 a 0.25 Hz.

Te voy a dejar como ejemplo el programa de una de las tareas. Te queda entonces el reto de realizar las otras tareas. No olvides sincronizar tu repositorio local con el remoto donde está la evaluación.
```c++
void task1(){
    enum class Task1States{
        INIT,
        WAIT_FOR_TIMEOUT
    };

    static Task1States task1State = Task1States::INIT;
    static uint32_t lastTime;
    static constexpr uint32_t INTERVAL = 1000;

    switch(task1State){
        case Task1States::INIT:{
            Serial.begin(115200);
            lastTime = millis();
            task1State = Task1States::WAIT_FOR_TIMEOUT;
            break;
        }

        case Task1States::WAIT_FOR_TIMEOUT:{
            // evento 1:            
            uint32_t currentTime = millis();
            if( (currentTime - lastTime) >= INTERVAL ){
                lastTime = currentTime;
                Serial.print("mensaje a 1Hz\n"); Serial.print('\n');
            }
            break;
        }

        default:{
            break;
        }
    }

}

void task2(){
    enum class Task1States{
        INIT,
        WAIT_FOR_TIMEOUT
    };

    static Task1States task1State = Task1States::INIT;
    static uint32_t lastTime;
    static constexpr uint32_t INTERVAL = 500;

    switch(task1State){
        case Task1States::INIT:{
            Serial.begin(115200);
            lastTime = millis();
            task1State = Task1States::WAIT_FOR_TIMEOUT;
            break;
        }

        case Task1States::WAIT_FOR_TIMEOUT:{
            // evento 1:            
            uint32_t currentTime = millis();
            if( (currentTime - lastTime) >= INTERVAL ){
                lastTime = currentTime;
                Serial.print("mensaje a 0.5Hz\n");
            }
            break;
        }

        default:{
            break;
        }
    }

}

void task3(){
    enum class Task2States{
        INIT,
        WAIT_FOR_TIMEOUT
    };

    static Task2States task2State = Task2States::INIT;
    static uint32_t lastTime;
    static constexpr uint32_t INTERVAL = 250;

    switch(task2State){
        case Task2States::INIT:{
            Serial.begin(115200);
            lastTime = millis();
            task2State = Task2States::WAIT_FOR_TIMEOUT;
            break;
        }

        case Task2States::WAIT_FOR_TIMEOUT:{
            // evento 1:            
            uint32_t currentTime = millis();
            if( (currentTime - lastTime) >= INTERVAL ){
                lastTime = currentTime;
                Serial.print("mensaje a 0.25Hz\n");
            }
            break;
        }

        default:{
            break;
        }
    }

}

void setup()
{
    task1(); task2(); task3();
}

void loop()
{
    task1(); task2(); task3();
}
```

# Ejercicio 11
![image](https://github.com/user-attachments/assets/0ed6bf80-82e6-40d7-b2bd-0ebf591791f9)
1. Analiza el programa. ¿Por qué enviaste la letra con el botón send? ¿Qué evento verifica si ha llegado algo por el puerto serial?
#### 
Porque el botón `send` envía los datos actuales ingresados al puerto serial, y luego el `if (Serial.available() > 0)` verifica si efectivamente se han ingresado datos o no. Si se han ingresado, el valor será mayor a cero y se imprimirá el mensaje.

2. [Abre](https://www.asciitable.com/) esta tabla.
####
3. Analiza los números que se ven debajo de las letras. Nota que luego de la r, abajo, hay un número. ¿Qué es ese número?
#### 
El valor en datos numéricos de cada tecla del teclado.

4. ¿Qué relación encuentras entre las letras y los números?
#### 
Tienen un una especie de orden, siendo el de los números ascendente.

5. ¿Qué es el 0a al final del mensaje y para qué crees que sirva?
#### 
Editando el mensaje, por ahora se podría presumir que representa el `\n`, osea, el salto de línea.

6. Nota que luego de verificar si hay datos en el puerto serial se DEBE HACER UNA LECTURA del puerto. Esto se hace para retirar del puerto el dato que llegó. Si esto no se hace entonces parecerá que siempre tiene un datos disponible en el serial para leer. ¿Tiene sentido esto? Si no es así habla con el profe.
#### 
Digamos que sí -b.

# Ejercicio 12
``` c++
void task1()
{
    enum class Task1States    {
        INIT,
        WAIT_DATA
    };
    static Task1States task1State = Task1States::INIT;

    switch (task1State)
    {
    case Task1States::INIT:
    {
        Serial.begin(115200);
        task1State = Task1States::WAIT_DATA;
        break;
    }

    case Task1States::WAIT_DATA:
    {
        // evento 1:        // Ha llegado al menos un dato por el puerto serial? 
       if (Serial.available() > 0)
        {
            // DEBES leer ese dato, sino se acumula y el buffer de recepción 
            //del serial se llenará.   
		        Serial.read();
            uint32_t var = 0;

            // Almacena en pvar la dirección de var.      
			      uint32_t *pvar = &var;

            // Envía por el serial el contenido de var usando 
           // el apuntador pvar.    
            Serial.print("var content: ");
            Serial.print(*pvar);
            Serial.print('\n');

            // ESCRIBE el valor de var usando pvar   
            *pvar = 10;
            Serial.print("var content: ");
            Serial.print(*pvar);
            Serial.print('\n');
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
    task1();
}

void loop()
{
    task1();
}
```
![image](https://github.com/user-attachments/assets/d7e2064e-f1af-449c-ac29-c202209cb98a)
La variable `pvar` se conoce como puntero. Simplemente es una variable en la cual se almacenan direcciones de otras variables. En este caso, en pvar se almacena la dirección de `var`. Nota que debes decirle al compilador el tipo de la variable (uint32_t en este caso) cuya dirección será almacenada en pvar.
Ejecuta el programa. Observa lo que hace. Ahora responde las siguientes preguntas mediante un ejercicio de ingeniería inversa:

- ¿Cómo se declara un puntero?
####
Se declara poniéndole el tipo de variable de la variable a la cual se le almacenará la dirección, de la forma: `tipoDeVariable *pVariableReferente;`. Lo que lo hace un puntero es ponerle el '*', y para identificarlo ponerle la 'p' al inicio.

- ¿Cómo se define un puntero? (cómo se inicializa)
####
Se le pueden asignar otros valores acorde al tipo de variable de `pvar`, igualándolo a dicho valor. Por ejemplo en el caso de que sea un `uint32_t`, se pueden poner números enteros, como sería en: `*pvar = 10;`

- ¿Cómo se obtiene la dirección de una variable?
#### 
Se iguala a la variable a la cual se desea almacenar la dirección, fijándose que ambas tengas el mismo tipo de variable. En este caso se iguala a `var`, quedando de la forma: `*pvar = &var;`.

- ¿Cómo se puede leer el contenido de una variable por medio de un puntero?
#### 
Se pone el puntero dentro de un `Serial.print()` para que se pueda imprimir y visualizar el valor almacenado en el puntero y en la variable vinculada al puntero.

- ¿Cómo se puede escribir el contenido de una variable por medio de un puntero?
#### 
Se iguala el puntero al valor deseado, y automáticamente el valor que tenía la variable vinculado a este se cambiará al del puntero.

# Ejercicio 13
Vas a escribir el siguiente programa, pero antes de ejecutarlo vas a tratar de lanzar una HIPÓTESIS de qué hace. Luego lo vas a ejecutar y compararás el resultado con lo que creías. Si el resultado no es el esperado, no deberías seguir al siguiente ejercicio hasta que no experimentes y salgas de la duda.
``` c++
static void changeVar(uint32_t *pdata)
{
    *pdata = 10;
}

static void printVar(uint32_t value)
{
    Serial.print("var content: ");
    Serial.print(value);
    Serial.print('\n');
}

void task1()
{
    enum class Task1States    {
        INIT,
        WAIT_DATA
    };
    static Task1States task1State = Task1States::INIT;

    switch (task1State)
    {
    case Task1States::INIT:
    {
        Serial.begin(115200);
        task1State = Task1States::WAIT_DATA;
        break;
    }

    case Task1States::WAIT_DATA:
    {
        // evento 1:        // Ha llegado al menos un dato por el puerto serial?        
	if (Serial.available() > 0)
        {
            Serial.read();
            uint32_t var = 0;
            uint32_t *pvar = &var;
            printVar(*pvar);
            changeVar(pvar);
            printVar(var);
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
    task1();
}

void loop()
{
    task1();
}
```
### Hipótesis
Lo que el programa hace es hace un tarea que utiliza dos métodos auxiliares. Inicialmente, dentro de la tarea se crea una variable `var` y su respectivo puntero `pvar`, estos se igualan y se imprime el valor del puntero, luego, usando los métodos externos, se cambia el valor del puntero y luego imprime la variable `var`. En teoría, solo se deberían imprimir dos ceros cuando se haya ingresado un valor con el `send`.

### Qué hace realmente
![image](https://github.com/user-attachments/assets/ab091042-d531-4584-b67f-643ed57fc0e5)
Aparentemente, al cambiar el valor del puntero de `pvar` también se cambió, de alguna forma, la variable `var`.

# Ejercicio 14
Realiza un programa que intercambie mediante una función el valor de dos variables.
``` c++
static void changeVar(uint32_t *pdata,uint32_t vata)
{
    *pdata = vata;
    vata = 30;
     //data = 15; No importa si se le cambia el valor a la variable que tiene el puntero, solo permite cambiar el valor si se le cambia el valor al puntero.
}

static void printVar(uint32_t value)
{
    Serial.print("var content: ");
    Serial.print(value);
    Serial.print('\n');
}

void task1()
{
    enum class Task1States    {
        INIT,
        WAIT_DATA
    };
    static Task1States task1State = Task1States::INIT;

    switch (task1State)
    {
    case Task1States::INIT:
    {
        Serial.begin(115200);
        task1State = Task1States::WAIT_DATA;
        break;
    }

    case Task1States::WAIT_DATA:
    {
        // evento 1:        // Ha llegado al menos un dato por el puerto serial?        
  if (Serial.available() > 0)
        {
            Serial.read();
            uint32_t var = 10;
            uint32_t *pvar = &var;
            uint32_t var2 = 20;
            printVar(var); printVar(var2); Serial.print('\n'); Serial.print('\n');
            changeVar(pvar,var2); var = 15;
            printVar(var); printVar(var2); Serial.print('\n'); Serial.print('\n');
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
    task1();
}

void loop()
{
    task1();
}
```

# Ejercicio 15
``` c++
static void processData(uint8_t *pData, uint8_t size, uint8_t *res)
{
    uint8_t sum = 0;
    for (int i = 0; i < size; i++)
    {
        sum = sum + (pData[i] - 0x30);
    }
    *res = sum;
}

void task1()
{
    enum class Task1States    {
        INIT,
        WAIT_DATA
    };
    static Task1States task1State = Task1States::INIT;
    static uint8_t rxData[5];
    static uint8_t dataCounter = 0;

    switch (task1State)
    {
    case Task1States::INIT:
    {
        Serial.begin(115200);
        task1State = Task1States::WAIT_DATA;
        break;
    }

    case Task1States::WAIT_DATA:
    {
        // evento 1:        if (Serial.available() > 0)
        {
            rxData[dataCounter] = Serial.read();
            dataCounter++;
            if (dataCounter == 5)
            {
                uint8_t result = 0;
                processData(rxData, dataCounter, &result);
                dataCounter = 0;
                Serial.print("result: ");
                Serial.print(result);
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
    task1();
}

void loop()
{
    task1();
}
```
### Utf8
![image](https://github.com/user-attachments/assets/55cbfd6b-2b82-48ba-9202-8d4ebbb21c3f)
### Mixed
![image](https://github.com/user-attachments/assets/6499f6da-8830-4a32-96df-902809b7515d)

Piensa en las siguientes cuestiones:
- ¿Por qué es necesario declarar `rxData` static? y si no es static ¿Qué pasa? ESTO ES IMPORTANTE, MUCHO.
#### 
Para que su valor se mantenga constante durante todo el programa, pues si no se le pone el static, su valor declarado se resetearía al pasar a otras funciones.

- `dataCounter` se define static y se inicializa en 0. ¿Cada vez que se ingrese a la función loop `dataCounter` se inicializa a 0? ¿Por qué es necesario declararlo static?
#### 
No, ya que se declaró como static, su valor será constante a menos que sea cambiado manualmente.

- Observa que el nombre del arreglo corresponde a la dirección del primer elemento del arreglo. Por tanto, usar en una expresión el nombre rxData (sin el operador []) equivale a &rxData[0].
- En la expresión `sum = sum + (pData[i] - 0x30);` observa que puedes usar el puntero pData para indexar cada elemento del arreglo mediante el operador [].
- Finalmente, la constante `0x30` en `(pData[i] - 0x30)` ¿Por qué es necesaria?
#### 
Idk.

# Ejercicio 16
Qué crees que ocurre cuando:
- ¿Qué pasa cuando hago un [Serial.available()](https://www.arduino.cc/reference/en/language/functions/communication/serial/available/)?
#### 
Este almacena los datos (en bytes) que se hayan enviado hacia el puerto serial.

- ¿Qué pasa cuando hago un [Serial.read()](https://www.arduino.cc/reference/en/language/functions/communication/serial/read/)?
#### 
Lee los datos que se enviaron al puerto serial.

- ¿Qué pasa cuando hago un `Serial.read()` y no hay nada en el buffer de recepción?
#### 
Como no hay nada que leer, este se predetermina a -1 (?).

- Un patrón común al trabajar con el puerto serial es este:
``` c++
if(Serial.available() > 0)
{
    int dataRx = Serial.read()
}
```
- ¿Cuántos datos lee `Serial.read()`?
#### 
Lee los datos que se hayan enviado, que en este caso si o si serían mayores a 0.

- ¿Y si quiero leer más de un dato? No olvides que no se pueden leer más datos de los disponibles en el buffer de recepción porque no hay más datos que los que tenga allí.
#### 
Se pone en el `if` que solo se lea cuando el `Serial.available()` sea mayor a uno, y se crea una variable para cada dato.

- ¿Qué pasa si te envían datos por serial y se te olvida llamar `Serial.read()`?
#### 
Los bytes enviados se van a ir acomuluando, es decir, los bytes enviados aumentarán a medida que más sean enviado, mientras que los recibidos permanecerán en cero; esto hasta que se reciban/lean con un `Serial.read()` y los bytes recibidos se actualicen.
#### bytes recibidos e enviados con un Serial.read() activo desde siempre:
![image](https://github.com/user-attachments/assets/f4eaecd6-0712-4253-a2f9-963488c75a44)

# Ejercicio 17
Así se pueden leer 3 datos que han llegado al puerto serial:
``` c++
if(Serial.available() >= 3)
{
    int dataRx1 = Serial.read();
    int dataRx2 = Serial.read();
    int dataRx3 = Serial.read();
}
```
¿Qué escenarios podría tener en este caso?
``` c++
if(Serial.available() >= 2)
{
    int dataRx1 = Serial.read();
    int dataRx2 = Serial.read();
    int dataRx3 = Serial.read();
}
```
El `if` funciona de tal forma que solo lee cuando se han enviado dos o más bytes. En el caso que solo se envién dos, se guardarían en las variables dataRx1 y dataRx2 los datos leídos de estos dos bytes, pero la tercera variable quedaría vacía; por lo cual, esta se mostraría como un -1.
![image](https://github.com/user-attachments/assets/02e912d4-1152-436d-81dd-df518cad1cd3)

# Ejercicio 18
Piense cómo podrías hacer lo siguiente:
1. Crea una aplicación con una tarea.
2. La tarea debe tener su propio buffer de recepción y una capacidad para 32 bytes.
3. La tarea almacena los datos del serial en su propio buffer de recepción (el buffer será un arreglo).
4. El buffer debe estar encapsulado en la tarea.
5. Los datos almacenados en el buffer no se pueden perder entre llamados a la tarea.
6. La tarea debe tener algún mecanismo para ir contando la cantidad de datos que han llegado. ¿Cómo lo harías?
####
Inventa un programa que ilustre todo lo anterior y en el archivo README.md escribe cómo solucionaste el problema.
``` c++
#define BUFFER_SIZE 32

void task1()
{
    // Estados de la tarea
    enum class Task1States {
        INIT,
        WAIT_DATA
    };

    // Estado inicial de la tarea
    static Task1States task1State = Task1States::INIT;

    // Buffer de recepción encapsulado
    static uint8_t rxBuffer[BUFFER_SIZE];

    // Contador de bytes recibidos
    static uint8_t dataCounter = 0;

    switch (task1State)
    {
    case Task1States::INIT:
    {
        Serial.begin(115200);
        task1State = Task1States::WAIT_DATA;
        break;
    }

    case Task1States::WAIT_DATA:
    {
        // Verifica si hay datos disponibles en el puerto serial
        while (Serial.available() > 0 && dataCounter < BUFFER_SIZE)
        {
            // Lee el próximo byte del puerto serial y lo almacena en el buffer
            rxBuffer[dataCounter] = Serial.read();
            dataCounter++;
            Serial.print("Hay ");
            Serial.print(dataCounter);
            Serial.print(" datos en el buffer");
            Serial.print('\n');
        }

        // Si el buffer se llena, se puede procesar o hacer algo con los datos aquí
        if (dataCounter == BUFFER_SIZE)
        {
            // Procesar los datos del buffer, por ejemplo, imprimir el contenido
            Serial.print("Buffer lleno: ");
            for (uint8_t i = 0; i < BUFFER_SIZE; i++) {
                Serial.print((char)rxBuffer[i]); // Convertir a carácter para imprimir
            }
            Serial.println();

            // Reiniciar el contador si es necesario o continuar con el procesamiento
            dataCounter = 0; // Para reutilizar el buffer, reiniciamos el contador
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
    // Llama a la tarea una vez para inicializarla
    task1();
}

void loop()
{
    // Llama a la tarea repetidamente para manejar la recepción de datos
    task1();
}
```

# Trabajo final
En una experiencia un grupo terrorista llamado “Disedentes del tiempo” realizan la configuración de una central nuclear global con la que se realizará la emisión de radiación nuclear al mundo. El sistema utiliza una interfaz de usuario simulada mediante el puerto serial, implementada en un entorno de software de Arduino.

En esta configuración inicial el sistema inicia en modo de configuración, mostrando una vez el mensaje CONFIG. En este modo se le configura el tiempo para abrir la cámara, por defecto tiene 5 seg y puede configurarse  entre 1 y 40 segundos con los botones S(Subir) y B(Bajar) en pasos de 1 segundo. Tan pronto se ajuste el tiempo de apertura de la cámara se termina con la letra L (listo) y se observa la cuenta en una pantalla, al final de la cual se abre la cámara y se hace la emisión de la radiación.  Cuando la cuenta regresiva termine debe mostrarse el mensaje “RADIACIÓN NUCLEAR ACTIVA” y a los dos segundos pasar nuevamente al modo CONFIG.

La misión del participante de la experiencia es salvar el mundo, por lo cual debe descifrar con pistas e ingresar el código de acceso, para lo cual  se envía por el puerto serial la secuencia 'C' seguido de la clave numérica (por ejemplo, 'C1234'). Si ingresas la clave correcta debe aparecer el mensaje “SALVASTE AL MUNDO”, sino la cuenta regresiva debe continuar hasta el fatal desenlace.

Te dejo el siguiente material trabajado en clase como complemento a la información del trayecto de actividades.

[Presentación de sistemas embebidos](https://upbeduco-my.sharepoint.com/:b:/g/personal/vera_perez_upb_edu_co/EaIafHzSSxFOmAdXvPJzVxIBcPcu_KEhUCUu-k56LJJ4GQ?e=UVVPgY)
``` c++
enum class SystemState
{
    CONFIG,
    COUNTDOWN,
    RADIATION,
    SAFETY
};

SystemState currentState = SystemState::CONFIG;
const int defaultTime = 5;
const int maxTime = 40;
const int minTime = 1;
int countdownTime = defaultTime;
const String accessCode = "1234"; // Código de acceso correcto
String enteredCode = "";
unsigned long countdownStartTime = 0;

void setup() {
    Serial.begin(115200);
    delay(10000);
    Serial.println("Sistema Iniciado."); Serial.println('\n');
    Serial.println("Modo CONFIG: Establezca el tiempo de apertura de la cámara."); Serial.println('\n');
    Serial.println("Presione 'S' para subir el tiempo, 'B' para bajar el tiempo."); Serial.println('\n');
    Serial.println("Cuando esté listo, presione 'L' para iniciar la cuenta regresiva."); Serial.println('\n');
    Serial.print("Tiempo actual configurado: "); Serial.println('\n');
    Serial.println(countdownTime);
}

void loop()
{
    switch (currentState)
{
        case SystemState::CONFIG:
            handleConfig();
            break;
        case SystemState::COUNTDOWN:
            handleCountdown();
            break;
        case SystemState::RADIATION:
            handleRadiation();
            break;
        case SystemState::SAFETY:
            Serial.println("SALVASTE AL MUNDO");
            delay(5000);
            while (true); // Detener el programa
            break;
    }
}

void handleConfig()
{
    if (Serial.available() > 0)
    {
        char input = Serial.read();
        if (input == 'S' && countdownTime < maxTime)
        {
            countdownTime++;
            Serial.print("Tiempo configurado: ");
            Serial.println(countdownTime);
        } else if (input == 'B' && countdownTime > minTime)
        {
            countdownTime--;
            Serial.print("Tiempo configurado: ");
            Serial.println(countdownTime);
        } else if (input == 'L')
        {
            Serial.print("Cuenta regresiva en: ");
            Serial.println(countdownTime);
            currentState = SystemState::COUNTDOWN;
            countdownStartTime = millis();
        } else if (input == '\n')
        {
            Serial.println("");
        }
        else {
            Serial.println("Entrada no válida. Use 'S' para subir, 'B' para bajar, 'L' para iniciar.");
        }
    }
}

void handleCountdown()
{
    unsigned long currentTime = millis();
    unsigned long elapsedTime = (currentTime - countdownStartTime) / 1000;
    int remainingTime = countdownTime - elapsedTime;

    if (remainingTime > 0)
    {
        Serial.print("Tiempo restante: ");
        Serial.println(remainingTime);

        if (Serial.available() > 0)
        {
            char input = Serial.read();
            if (input == 'C')
            {
                enteredCode = Serial.readStringUntil('\n');
                if (enteredCode == accessCode)
                {
                    currentState = SystemState::SAFETY;
                } else
                {
                    Serial.println("Código incorrecto. Continuando cuenta regresiva.");
                }
            } else
            {
                Serial.println("Entrada no válida durante la cuenta regresiva. Ingrese 'C' seguido del código.");
            }
        }

        delay(1000);
    } else
    {
        currentState = SystemState::RADIATION;
    }
}

void handleRadiation()
{
    Serial.println("RADIACIÓN NUCLEAR ACTIVA");
    delay(2000); // Esperar 2 segundos
    currentState = SystemState::CONFIG;
    Serial.println("Volviendo al modo CONFIG. Establezca el tiempo de apertura de la cámara.");
    countdownTime = defaultTime; // Reiniciar tiempo de configuración
}
```
## Diagrama de estados 
![diagrama de estados](https://github.com/user-attachments/assets/598f47e3-2375-4a9a-8b80-7c26fcfb7ad0)
