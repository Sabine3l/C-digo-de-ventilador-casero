# C-digo-de-ventilador-casero
Código que permite crear una simulación de un sistema de refrigeración a partir de vibecoding.
#include <ESP32Servo.h>

// Definición de Pines
const int PIN_TERMISTOR = 34; 
const int PIN_LED = 2;        
const int PIN_SERVO = 18;     

// Parámetros del Termistor NTC MF52 10k
const float BETA = 3435;             
const float RESISTENCIA_REF = 10000; 
const float TEMP_REF = 298.15;       
const float RES_NTC_REF = 10000;     

// Configuración del sistema
const float UMBRAL_TEMPERATURA = 28.0; 
Servo miVentilador;
bool servoEnganchado = false; // Nos ayuda a saber si el servo está activo

void setup() {
  Serial.begin(115200);
  pinMode(PIN_LED, OUTPUT);
  
  // Forzar apagado absoluto al inicio
  digitalWrite(PIN_LED, LOW);
  miVentilador.detach(); // Asegura que el motor no reciba señal ni vibre
  
  Serial.println("Sistema optimizado iniciado.");
}

void loop() {
  int valorADC = analogRead(PIN_TERMISTOR);
  
  if (valorADC == 0) valorADC = 1;
  if (valorADC == 4095) valorADC = 4094;

  float voltaje = (valorADC * 3.3) / 4095.0;
  float resistenciaNTC = RESISTENCIA_REF * ((3.3 / voltaje) - 1.0);
  float temperaturaKelvin = 1.0 / ((1.0 / TEMP_REF) + (log(resistenciaNTC / RES_NTC_REF) / BETA));
  float temperaturaCelsius = temperaturaKelvin - 273.15;

  Serial.print("Temperatura actual: ");
  Serial.print(temperaturaCelsius);
  Serial.println(" °C");

  // LÓGICA DE CONTROL ESTRICTA
  if (temperaturaCelsius >= UMBRAL_TEMPERATURA) {
    // ---- ENCIENDE TODO SÓLO SI PASA LOS 28°C ----
    digitalWrite(PIN_LED, HIGH); // Enciende el LED con fuerza
    
    // Si el servo estaba apagado, lo encendemos
    if (!servoEnganchado) {
      miVentilador.attach(PIN_SERVO);
      servoEnganchado = true;
    }
    
Serial.println("¡Alerta! Ventilador 360° girando continuo.");
    miVentilador.write(180); // 180 activa el giro continuo en los servos 360
    delay(100);
  } 
  else {
    // ---- APAGADO TOTAL SI ES MENOR A 28°C ----
    digitalWrite(PIN_LED, LOW); // Apaga el LED por completo
    
    // Si el servo estaba encendido, lo desactivamos por completo para evitar vibraciones
    if (servoEnganchado) {
      miVentilador.write(0);  // Regresa a posición inicial
      delay(150);             // Le damos un instante para que llegue a 0 antes de apagarlo
      miVentilador.detach();  // Desconecta la señal del motor (Cero vibraciones)
      servoEnganchado = false;
    }
    
    delay(600); // Muestreo tranquilo si todo está frío
  }
}
