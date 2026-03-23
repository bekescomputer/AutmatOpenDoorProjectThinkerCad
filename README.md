#  Ajtó nyitó mechanizmus szervertereremhez

### Projekt leírása:

A projektet az az ötlet hívta életre, hogy a szerverterembe ne csak kulcsal lehessen bemenni, hanem azbonítással. A rendszer a jelen állapotában csak fixen beégetett kódot tud lekezelni, de eltudja dönteni jó-e vagy sem. Helyes kód esetén a nyitó mechanizmus kinyitja a szervón keresztül az ajtót
és erről hangjelzést is ad. 5 másodperc utána szervó zárja az ajtó nyitó mechanizmust. 
Erről digitális kijelzőn ad információt részünkre.

![OpenDoorSystem_Electronic_Projekt.png](https://github.com/bekescomputer/AutmatOpenDoorProjectThinkerCad/blob/main/OpenDoorSystem_Electronic_Projekt.png?raw=true)

### Default belépés

Az ajtó nyitása alap értékként az 1234# kódot veszi fel.

A # jel zárja a kód bevitelt
A C pedig törli ha rosszul nyomtunk le egy értéket

### Projekt elérhetősége

A projekt az alábbi linken elérhető a TinkerCad-ban

https://www.tinkercad.com/things/52x77gU6iEI-opendoorsystemelectronic

### Alkatrész készlet

| Név                                                   | Mennyiség (db)  | Öszetevő                                                                        |
|-------------------------------------------------------|-----------------|---------------------------------------------------------------------------------|
| U1                                                    | 1               | Arduino Uno R3                                                                  |
| U2                                                    | 1               | PCF8574-alapú, 39 (0x27) LCD 16 x 2 (I2C)                                       |
| U3                                                    | 1               | PCF8574-alapú, 38 (0x26) LCD 16 x 2 (I2C)                                       |
| Meter1                                                | 1               | Feszültség Multiméter                                                           |
| T2                                                    | 1               | nMOS tranzisztor (MOSFET)                                                       |
| R1                                                    | 1               | 10 kΩ Ellenállás                                                                |
| R3                                                    | 1               | 4 kΩ Ellenállás                                                                 |
| SERVO1                                                | 1               | Pozicionális Mikroszervo                                                        |
| PIEZO1                                                | 1               | Piezo                                                                           |
| KEYPAD1                                               | 1               | 4x4-es billentyűzet                                                             |


### A teljes Forráskód:
```			
// Fejlécállományok betöltése

#include <Keypad.h>
#include <LiquidCrystal_I2C.h>
#include <Servo.h>
#define DECODE_NEC

const int nmosPin = 10;

// --- Szervó definiálása ---

int SzervoPin = 11;
Servo szervo;

// Piezó Hangszóró definiálása

int piezoPin = A2;

//LCD kijelző aktiválása

LiquidCrystal_I2C lcd_bal(0x26, 16, 2);
LiquidCrystal_I2C lcd_jobb(0x27, 16, 2);  


const String helyesKod = "1234";
String bevitel = "";

// Keypad beállítás

const byte sorok = 4;  
const byte oszlopok = 4;

// Keypad gombkiosztás

char gombok[sorok][oszlopok] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

// Keypad bekötése

byte sorPin[sorok] = {9, 8, 7, 6};     // 1–4 vezeték
byte oszlopPin[oszlopok] = {5, 4, 3, 2}; // 5–8 vezeték

Keypad billentyuzet = Keypad(makeKeymap(gombok), sorPin, oszlopPin, sorok, oszlopok);


void setup()
{
	Wire.begin();        	// I2C indul
	lcd_bal.init();			// Bal LCD inicializálása
    lcd_jobb.init();		// Jobb LCD inicializálása
  
	lcd_bal.backlight();	// Bal LCD fény adása
	lcd_jobb.backlight();	// Jobb LCD fény adása
	
	delay(50); // I2C stabilizálás

	szervo.attach(SzervoPin);
  
	pinMode(nmosPin, OUTPUT);

	digitalWrite(nmosPin, LOW); // Alaphelyzetben zárva
		
	lcd_jobb.setCursor(0,0);
	lcd_jobb.print("PIN kod:");
	lcd_bal.setCursor(0,0);
	lcd_bal.print("Kerlek add meg a ");	
	lcd_bal.setCursor(0,1);
	lcd_bal.print("belepesi kododat");

}

void loop()
{
	char key = billentyuzet.getKey();

	if (key) 
	{
		if (key == '#') 
		{     
			// A  # jel az ENTER
			lcd_jobb.clear();
			if (bevitel == helyesKod) 
			{
				lcd_jobb.print("Helyes kod!");
				lcd_bal.clear();
				lcd_bal.print("Az ajto nyilik!");
				digitalWrite(nmosPin, HIGH); 
				szervo.write(90);      // Ajto nyit
				tone(piezoPin, 1000);  // 1000 Hz sípolás
			
				delay(5000); // 5 masodpercig tartja nyitva
		
				szervo.write(0);      // Ajto zár
				noTone(piezoPin);
				lcd_bal.clear();
				lcd_bal.print("Az ajto zarodik!");
				delay(1000);
				digitalWrite(nmosPin, LOW);
				
				lcd_bal.clear();
				lcd_bal.setCursor(0,0);
				lcd_bal.print("Kerlek add meg a ");	
				lcd_bal.setCursor(0,1);
				lcd_bal.print("belepesi kododat");
			} 
			else 
			{
				lcd_jobb.print("Hibas kod!");
			}
			
			bevitel = "";
			delay(2000);
			lcd_jobb.clear();
			lcd_jobb.print("PIN kod:");
		} 
		else if (key == 'C') 
		{  
			// Torles
			bevitel = "";
			lcd_jobb.clear();
			lcd_jobb.print("PIN kod:");
		}
		else 
		{  
			bevitel += key;
			lcd_jobb.setCursor(0, 1);
			lcd_jobb.print(bevitel);
		}
	}
}	
```
