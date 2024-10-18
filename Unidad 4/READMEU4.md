# U N I D A D  4
Aprendizaje integrativo
_________________________________________________________________________________________________________________________________________________________________________________________
# Ejercicio 1: caso de estudio-plugin Ardity
Lo primero que te propondré es que pongas a funcionar el demo del plugin. Todo lo relacionado con el plugin estará en [este](https://github.com/DWilches/Ardity) repositorio. Comienza leyendo el archivo README.md.

Cuando te sientas listo para comenzar a experimentar ten presente que la versión de Unity con la cual se realizó el proyecto fue la `2018.2`.  Abre el proyecto con tu versión de Unity.

Ve a la carpeta Scenes y comienza a analizar cada una de las escenas `DemoScene`. Son en total *5*. En uno de los ejercicios que vienen vamos a analizar juntos la scene `DemoScene_UserPoll_ReadWrite`. Tu debes decidir luego *con cuál scene trabajar* y tratar de analizarla a fondo tal como lo haremos a continuación.

***Información***
- **Escena:** ...
- ...

``` c++

```


# Ejercicio 2: concepto de hilo
Observa y analiza [este](https://youtu.be/5wpSidCEJn4) video donde te explicarán rápidamente el concepto de *hilo*.

Te voy a pedir que veas [este](https://youtu.be/nv1NUR-qjcU) video corto de `15 minutos` donde verás aplicado el concepto de hilo y por qué es necesario.

Ahora, no te voy a pedir que hagas lo siguiente ya, claro, a menos que seas una persona muy curiosa y además tengas tiempo. En [este](https://learn.microsoft.com/en-us/dotnet/standard/threading/managed-threading-basics) sitio puedes profundizar sobre el concepto de hilo.

### ¿Qué es un hilo? 
...



``` c++

```


# Ejercicio 3
Ahora, vamos a analizar más detalladamente una de las escenas demo de Ardity: `DemoScene_UserPoll_ReadWrite`

Primero, vamos a analizar rápidamente el código de arduino (para un protocolo ASCII):
``` c++
uint32_t last_time = 0;

void setup()
{
    Serial.begin(9600);
}

void loop()
{
	// Print a heartbeat
	if ( (millis() - last_time) >  2000)
    {
        Serial.print("Arduino is alive!!");
        Serial.print('\n');
        last_time = millis();
    }

// Send some message when I receive an 'A' or a 'Z'.
switch (Serial.read())
    {
			case 'A':
            Serial.print("That's the first letter of the abecedarium.");
            Serial.print('\n');
						break;
			case 'Z':
            Serial.print("That's the last letter of the abecedarium.");
            Serial.print('\n');
						break;
    }
}
```
Consideraciones a tener presentes con este código:

- La velocidad de comunicación es de 9600. Esa misma velocidad se tendrá que configurar del lado de Unity para que ambas partes se puedan entender.
- Nota que no estamos usando la función delay(). Estamos usando millis para medir tiempos relativos. Nota que cada dos segundos estamos enviando un mensaje indicando que el arduino está activo: `Arduino is alive!!`
- Observa que el buffer del serial se lee constantemente. NO estamos usando el método available() que usualmente utilizamos. ¿Recuerdas lo anterior? Con available() nos aseguramos que el buffer de recepción tiene al menos un byte para leer; sin embargo, cuando usamos Serial.read() sin verificar antes que tengamos datos en el buffer, es muy posible que el método devuelva un -1 indicando que no había nada en el buffer de recepción. NO OLVIDES ESTO POR FAVOR.
- Por último nota que todos los mensajes enviados por arduino finalizan con:
``` c++
Serial.print('\n');
```
¿Recuerdas en la unidad 2 para qué hacemos esto? **¿Podrías desde ahora predecir qué tipo de protocolo utilizará este demo (ASCII o binario)?**

...

Ahora analicemos la parte de Unity/Ardity. Para ello, carguemos una de las escenas ejemplo: `DemoScene_UserPoll_ReadWrite` (ya me da pereza pegar todo eso) .....


``` c++

```

# Ejercicio 4?



``` c++

```

# RETO


#### Códigos
Microcontrolador:
``` c++

```
Unity:
``` c++

```
#### Interfaz
...

#### *¿Cómo funciona?*
...









