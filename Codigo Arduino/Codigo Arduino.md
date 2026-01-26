\#include <SimpleModbusSlave.h>

\#include <DHT.h>



\#define DHTTYPE DHT11



// entradas y salidas fisicas del arduino



\#define SensorTempSalon 3     // sensor de temperatura y humedad del salon

\#define VentiladorSalon 2     // indica si el radiador de aerotermia esta funcionando (1 = funcionando / 0 = apagado)

\#define SensorTempDorm1 5

\#define VentiladorDorm1 4

\#define SensorTempDorm2 6

\#define VentiladorDorm2 7

\#define SensorTempPatio 9

\#define SensorLuzPatio 8

\#define ActuadorLuzPatio 12

\#define SDledFrio 10          // Salida digital que activa el led que indica que la aerotermia genera frio

\#define SDledCalor 11

\#define SDmotorAire 13        // Salida digital que activa el rele o led que indica que la aerotermia esta funcionando





// bits que se pasan al registro modbus para el scadabr



\#define bitSensorLuzPatio 0     // indica si hay luz en el patio. Si es de noche tiene que encender la luz

\#define bitActuadorLuzPatio 1   // se pone a 1 para activar la luz del patio por la noche

\#define bitFrio 2               // indica que el aire acondicionado genera frio

\#define bitCalor 3              // indica que el aire acondicionado genera calor

\#define bitMotor 4              // Simula un motor de aerotermia





//////////////// Variables que van a funcionar en ScadaBR ///////////////////

enum 

{    

&nbsp;

&nbsp; mbTempSalon,

&nbsp; mbHumSalon,

&nbsp; mbTempDorm1,

&nbsp; mbHumDorm1,

&nbsp; mbTempDorm2,

&nbsp; mbHumDorm2,

&nbsp; mbTempPatio,

&nbsp; mbHumPatio,

&nbsp; mbTempMedia,

&nbsp; mbTempMaxima,

&nbsp; mbTempMinima,

&nbsp; mbSensorLuzPatio,

&nbsp; mbFRIO,

&nbsp; mbCALOR,

&nbsp; mbMOTOR, 

&nbsp; HOLDING\_REGS\_SIZE // leave this one

&nbsp; // total number of registers for function 3 and 16 share the same register array

&nbsp; // i.e. the same address space

};



unsigned int holdingRegs\[HOLDING\_REGS\_SIZE]; // function 3 and 16 register array

////////////////////////////////////////////////////////////



bool ed\_SensorLuzPatio;   // Valor de la entrada digital del sensor del patio



byte contador\_segundos=0;   // para contar segundos antes de llamar a lectura de sensores



DHT dhtSalon(SensorTempSalon, DHTTYPE),

&nbsp;   dhtDorm1(SensorTempDorm1, DHTTYPE),

&nbsp;   dhtDorm2(SensorTempDorm2, DHTTYPE),

&nbsp;   dhtPatio(SensorTempPatio, DHTTYPE);





unsigned int tempSalon, humSalon,    // variables donde se almacena las temperaturas y humedades

&nbsp;     tempDorm1, humDorm1,

&nbsp;     tempDorm2, humDorm2,

&nbsp;     tempPatio, humPatio;







void setup()

{

&nbsp; /\* parameters(HardwareSerial\* SerialPort,

&nbsp;               long baudrate, 

&nbsp;		unsigned char byteFormat,

&nbsp;               unsigned char ID, 

&nbsp;               unsigned char transmit enable pin, 

&nbsp;               unsigned int holding registers size,

&nbsp;               unsigned int\* holding register array)

&nbsp; \*/

&nbsp;	

&nbsp; modbus\_configure(\&Serial, 9600, SERIAL\_8N1, 1, 2, HOLDING\_REGS\_SIZE, holdingRegs);

&nbsp; //modbus\_update\_comms(9600, SERIAL\_8N1, 1);





&nbsp; pinMode(SensorTempSalon, INPUT);    // sensor dht de temperatura y humedad

&nbsp; pinMode(SensorTempDorm1, INPUT);    // sensor dht de temperatura y humedad

&nbsp; pinMode(SensorTempDorm2, INPUT);    // sensor dht de temperatura y humedad

&nbsp; pinMode(SensorTempPatio, INPUT);    // sensor dht de temperatura y humedad



&nbsp; pinMode(SensorLuzPatio, INPUT);     // sensor de luz



&nbsp; pinMode(VentiladorSalon, OUTPUT);

&nbsp; pinMode(VentiladorDorm1, OUTPUT);

&nbsp; pinMode(VentiladorDorm2, OUTPUT);

&nbsp; pinMode(ActuadorLuzPatio, OUTPUT);



&nbsp; dhtSalon.begin();

&nbsp; dhtDorm1.begin();

&nbsp; dhtDorm2.begin();

&nbsp; dhtPatio.begin();



}





void LeerTempHum()

{



&nbsp;// Codigo para leer temperatura y humedad de todas las entradas



&nbsp; tempSalon = dhtSalon.readTemperature(); // lee la temperatura del salon

&nbsp; humSalon = dhtSalon.readHumidity(); // lee la humedad del salon



&nbsp; tempDorm1 = dhtDorm1.readTemperature(); // lee la temperatura del dormitorio 1

&nbsp; humDorm1 = dhtDorm1.readHumidity(); // lee la humedad del dormitorio 1



&nbsp; tempDorm2 = dhtDorm2.readTemperature(); // lee la temperatura del dormitorio 2

&nbsp; humDorm2 = dhtDorm2.readHumidity(); // lee la humedad del dormitorio 2



&nbsp; tempPatio = dhtPatio.readTemperature(); // lee la temperatura del Patio

&nbsp; humPatio = dhtPatio.readHumidity(); // lee la humedad del Patio





&nbsp; // Paso los valores a los registros de modbus que se pasan a scadabr



&nbsp; holdingRegs\[mbTempSalon] = tempSalon;

&nbsp; holdingRegs\[mbHumSalon] = humSalon;



&nbsp; holdingRegs\[mbTempDorm1] = tempDorm1;

&nbsp; holdingRegs\[mbHumDorm1] = humDorm1;



&nbsp; holdingRegs\[mbTempDorm2] = tempDorm2;

&nbsp; holdingRegs\[mbHumDorm2] = humDorm2;



&nbsp; holdingRegs\[mbTempPatio] = tempPatio;

&nbsp; holdingRegs\[mbHumPatio] = humPatio; 



&nbsp; holdingRegs\[mbTempMedia] = (unsigned int) (tempSalon + tempDorm1 + tempDorm2) / 3;   // El modbus necesita un valor entero



}





void ControlTempFrioCalor()

{



&nbsp; // Si la temperatura media es mayor que la temperatura maxima, se activa el led frio para indicar que se tienen que bajar las temperaturas



&nbsp; if (holdingRegs\[mbTempMedia] > holdingRegs\[mbTempMaxima]) {



&nbsp;       digitalWrite(SDmotorAire, 1);   // El arduino activa el motor de la aerotermia

&nbsp;       digitalWrite(SDledFrio, 1);   // activa el frio

&nbsp;       digitalWrite(SDledCalor,0);   // y desactiva el calor



&nbsp;       holdingRegs\[mbMOTOR] = 1;

&nbsp;       holdingRegs\[mbFRIO] = 1;

&nbsp;       holdingRegs\[mbCALOR] = 0;



&nbsp; }





&nbsp; // Si la temperatura media es menor que la temperatura minima, se activara el led de calor para indicar que tiene que aumentar la temperatura



&nbsp; if (holdingRegs\[mbTempMedia] < holdingRegs\[mbTempMinima]) {



&nbsp;       digitalWrite(SDmotorAire, 1);   // El arduino activa el motor de la aerotermia

&nbsp;       digitalWrite(SDledFrio, 0);   //desactiva el frio

&nbsp;       digitalWrite(SDledCalor,1);   //activa el calor     



&nbsp;       holdingRegs\[mbMOTOR] = 1;     

&nbsp;       holdingRegs\[mbFRIO] = 0;

&nbsp;       holdingRegs\[mbCALOR] = 1;



&nbsp; }     





&nbsp; // Si la temperatura media esta en el rango normal, desactiva todo



&nbsp;if ((holdingRegs\[mbTempMedia] >= holdingRegs\[mbTempMinima]) \& (holdingRegs\[mbTempMedia] <= holdingRegs\[mbTempMaxima])) {

&nbsp;

&nbsp;   digitalWrite(SDmotorAire, 0);   // El arduino desactiva el motor de la aerotermia

&nbsp;   digitalWrite(SDledFrio, 0);     //desactiva el frio

&nbsp;   digitalWrite(SDledCalor,0);     // y desactiva el calor



&nbsp;   holdingRegs\[mbMOTOR] = 0;

&nbsp;   holdingRegs\[mbFRIO] = 0;

&nbsp;   holdingRegs\[mbCALOR] = 0;



&nbsp;}





}





void ControlLuzPatio()

{



&nbsp; ed\_SensorLuzPatio = digitalRead(SensorLuzPatio);      // Lee el estado del sensor digital



&nbsp; digitalWrite(ActuadorLuzPatio, ed\_SensorLuzPatio);    // Escribe la salida digital 



&nbsp; holdingRegs\[mbSensorLuzPatio] = ed\_SensorLuzPatio;    // lo paso al registro modbus



}







void loop()

{

&nbsp; // modbus\_update() is the only method used in loop(). It returns the total error

&nbsp; // count since the slave started. You don't have to use it but it's useful

&nbsp; // for fault finding by the modbus master.

&nbsp; 

&nbsp; modbus\_update();

&nbsp; 

&nbsp; delay(100);   // espera 100 ms



&nbsp; contador\_segundos = contador\_segundos + 1;



&nbsp; if (contador\_segundos>=50) {    // leo cada 5 segundos (5000 mS)



&nbsp;   LeerTempHum();  // Leo sensores de temperatura y humedad

&nbsp;   ControlTempFrioCalor();



&nbsp;   contador\_segundos = 0;



&nbsp; }

&nbsp;  

&nbsp; ControlLuzPatio();



&nbsp; 

}

