# Repositorio-EcoFlow
//EMISOR

//Incluyendo la librería RF24
#include <nRF24L01.h>
#include <printf.h>
#include <RF24.h>
#include <RF24_config.h>

//Incluyendo la librería SPI (un bus) (para la comunicación/transferencia de datos) 
#include <SPI.h>

//Colocando los pines CE,CSN , respectivamente
RF24 Radio(7,8);

//Ahora, se creará la matriz de bytes que representará la dirección con la que se reconocerán entre sí los dos módulos. Esta dirección va a permitir elegir con //qué receptor es que nos vamos a comunicar. Aunque, en este caso, se usará la misma dirección tanto para el transmisor como el receptor
const byte identificacion[6]="00001";

void setup() {

  //Se inicializa el puerto serial para verificar que esté enviando el mensaje correctamente
  Serial.begin(9600);
  //Ahora, se iniciará el objeto creado inicialmente
  Radio.begin();

  //Una vez realizado lo anterior, se utilizará la función Radio.openWritingPipe(), donde
  //se pasará por parámetro la dirección del receptor al que se enviarán los datos (que
  //estará representada por la cadena de caracteres que hemos configurado)
  Radio.openWritingPipe(identificacion);

  //A continuación, se configurará el nivel del amplificador de potencia, mediante la
  //función Radio.setPALevel(). En este caso, se usará un nivel de prueba mínimo, ya que
  //ambos módulos (solo para la prueba) estarán cerca el uno del otro. En cambio, cuando
  //ya se implemente (dependiedo de la distancia en que estarán estos) se cambiará el
  //amplicador de potencia
  Radio.setPALevel(RF24_PA_MIN);

  //Ahora, establecemos al módulo como transmisor mediante la siguiente función
  Radio.stopListening();

}

void loop() {

  //Se coloca el código de turbidez de agua (ya que este es el emisor)
  //SENSOR DE TURBBIDEZ DE AGUA
  //Leyendo los valores del sensor
  float ValorSensor=analogRead(A0);
  
  //Obteniendo el voltaje (Pues los valores del sensor irán de 0-1023)
  float Voltaje=(ValorSensor)*(5/1024.0);
  //La turbidez es medida en NTU, así que primero se debe realizar la conversión
  
  float ValorNTU;
  //Fórmula que representa la relación de voltaje y turbidez
  ValorNTU=-(1120.4*Voltaje*Voltaje)+(5742.3*Voltaje)-4352.9;
  Serial.print("ValorNTU: ");
  Serial.println(ValorNTU);
  
  //Una vez configurado lo anterior, se creará una matriz que representará el mensaje que
  //se quiere enviar
  //const char Mensaje[]="Conexión arduinos - Grupo 5";

  //Ahora, para enviar dicho mensaje al receptor, usamos la función .write()
  Radio.write(&ValorNTU,sizeof(ValorNTU));
  //Ojo que el símbolo '&' es un indicador que apunta a la variable que contiene los datos
  //que se van a enviar. Y con el 'sideof()' se devuelve la cantidad total de bytes del mensaje

  //Establecemos el tiempo de espera entre envío
  delay(1000);


  //Valores de pH
  //Valores de pH
  float pH=analogRead(A1);
  //Colocando los valores del pH con las fórmulas que obtuvimos realizando las pruebas (acá se tienen 3, pero se escogerá la más precisa)
  //Fórmula 1
  float ValorPH1=-0.0316*pH+32.489;

  //Fórmula 2
  float ValorPH2=-0.0316*pH+13.887;

  //Fórmula 3
  float ValorPH3=-0.0291*pH+28.757;

  //Se enviarán los 3 valores al arduino receptor
  //pH1
  Radio.write(&ValorPH1,sizeof(ValorPH1));
  delay(1000);

  //pH2
  Radio.write(&ValorPH2,sizeof(ValorPH2));
  delay(1000);
  
  //pH3
  Radio.write(&ValorPH3,sizeof(ValorPH3));
  delay(1000);

  //Establecemos el tiempo de espera entre envío
  delay(1000);

}

//RECEPTOR

//#include <nRF24L01.h>
//#include <printf.h>
//#include <RF24.h>
//#include <RF24_config.h>

//De igual manera, incluimos ambas librerías
#include <nRF24L01.h>
#include <printf.h>
#include <RF24.h>
#include <RF24_config.h>

#include <SPI.h>

//Creamos un objeto RF24, los pines CE,CSN, respectivamente
RF24 Receptor(7,8); 

//Como se mencionó anteriormente, la dirección tanto para el transmisor como el receptor
//es la misma, entonces
const byte identificacion[6]="00001";

//ValorNTU que recibirá como mensaje
//float ValorNTU;
float ValorMed;

void setup() {
  //Se inicializa la comunicación serial para ver si está funcionando
  Serial.begin(9600);

  //Inicializamos el objeto que hemos creado
  Receptor.begin();

  //Se establecerá la dirección mediante la función .setReadingPipe(), pero como
  //anteriormente hemos dicho que la dirección será la misma, entonces
  Receptor.openReadingPipe(0,identificacion);

  //Se configura de igual manera el amplificador de potencia (se colocará mínimo, ya
  //que ambos módulos estarán cerca solo para la prueba)
  Receptor.setPALevel(RF24_PA_MIN);

  //Como este módulo se establecerá como 'receptor', entonces inicializamos la función
  // .startListening()
  Receptor.startListening();

}

void loop() {
  //Verificamos si está disponible
  if (Receptor.available()){
    //Entonces se crea una matriz de caracteres donde se recibirá el mensaje
    //char texto[32]="";

    //Se espera a que llegue algún mensaje o dato por RF desde el módulo
    //Receptor.read(&ValorNTU,sizeof(ValorNTU));
    Receptor.read(&ValorMed,sizeof(ValorMed));

    //Una vez que se transmite el dato, este será recibido y enviado al monitor serial
    Serial.print("El dato captado por el sensor es");
    Serial.println(ValorMed);


    
  }

}
