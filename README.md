#include <WiFi.h>
#include <HTTPClient.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// ========================================
// KONFIGURASI LCD I2C
// ========================================
LiquidCrystal_I2C lcd(0x27, 16, 2);

// ========================================
// KONFIGURASI SENSOR LDR
// ========================================
const int ldrPin = 34;

// ========================================
// KONFIGURASI WIFI
// ========================================
const char* ssid = "Jendral Wito";
const char* password = "jemputjamtigapagi";

// ========================================
// KONFIGURASI THINGSPEAK
// ========================================
unsigned long myChannelNumber = 3568782;

const char* myWriteAPIKey = "KEPMDYIB1BLUPYQX";

String serverName = "http://api.thingspeak.com/update";

// ========================================
// THRESHOLD SENSOR LDR
// ========================================
// ADC 0 - 1000    = Pencahayaan Tinggi
// ADC 1001 - 3000 = Pencahayaan Sedang
// ADC 3001 - 4095 = Pencahayaan Rendah

int batasTerik = 1000;
int batasMendung = 3000;

// ========================================
// INTERVAL PENGIRIMAN
// ========================================
// 30000 ms = 30 detik

const unsigned long interval = 30000;
unsigned long waktuSebelumnya = 0;

// ========================================
// MEMBACA RATA-RATA SENSOR LDR
// ========================================
int bacaSensorLDR() {

  long total = 0;

  for (int i = 0; i < 10; i++) {

    total += analogRead(ldrPin);
    delay(20);
  }

  return total / 10;
}

// ========================================
// KONEKSI WIFI
// ========================================
void koneksiWiFi() {

  if (WiFi.status() == WL_CONNECTED) {
    return;
  }

  Serial.println("Menghubungkan ke WiFi...");

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Connecting WiFi");

  WiFi.begin(ssid, password);

  unsigned long mulai = millis();

  while (WiFi.status() != WL_CONNECTED &&
         millis() - mulai < 15000) {

    delay(500);
    Serial.print(".");
  }

  lcd.clear();

  if (WiFi.status() == WL_CONNECTED) {

    Serial.println();
    Serial.println("WiFi Berhasil Terhubung!");

    Serial.print("IP Address: ");
    Serial.println(WiFi.localIP());

    lcd.setCursor(0, 0);
    lcd.print("WiFi Connected");

    delay(1500);
  }

  else {

    Serial.println();
    Serial.println("WiFi Gagal Terhubung!");

    lcd.setCursor(0, 0);
    lcd.print("WiFi Failed");

    delay(1000);
  }

  lcd.clear();
}

// ========================================
// PENGIRIMAN DATA KE THINGSPEAK
// ========================================
void kirimThingSpeak(String parameterField) {

  if (WiFi.status() != WL_CONNECTED) {

    Serial.println("WiFi Terputus!");
    return;
  }

  HTTPClient http;

  String serverPath =
    serverName +
    "?api_key=" +
    String(myWriteAPIKey) +
    parameterField;

  http.begin(serverPath);

  http.setTimeout(10000);

  int httpResponseCode = http.GET();

  if (httpResponseCode == 200) {

    String response = http.getString();

    if (response.toInt() > 0) {

      Serial.println("Pengiriman ThingSpeak Berhasil!");

      Serial.print("Entry ID: ");
      Serial.println(response);
    }

    else {

      Serial.println("ThingSpeak Menolak Data!");
    }
  }

  else {

    Serial.print("Pengiriman Gagal! HTTP: ");
    Serial.println(httpResponseCode);
  }

  http.end();
}

// ========================================
// SETUP
// ========================================
void setup() {

  Serial.begin(115200);

  analogReadResolution(12);

  // Inisialisasi LCD I2C
  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();
  lcd.clear();

  // Pesan awal
  lcd.setCursor(0, 0);
  lcd.print("MONITORING LDR");

  lcd.setCursor(0, 1);
  lcd.print("TANAMAN HIAS");

  delay(2000);

  // Menghubungkan WiFi
  koneksiWiFi();

  // Mulai pengukuran langsung
  waktuSebelumnya = millis() - interval;
}

// ========================================
// LOOP UTAMA
// ========================================
void loop() {

  // Periksa koneksi WiFi
  koneksiWiFi();

  // Pengiriman setiap 30 detik
  if (millis() - waktuSebelumnya >= interval) {

    waktuSebelumnya = millis();

    // ====================================
    // MEMBACA SENSOR LDR
    // ====================================
    int nilaiCahaya = bacaSensorLDR();

    String statusCuaca = "";
    String statusLcd = "";
    String parameterField = "";

    // ====================================
    // KLASIFIKASI PENCAHAYAAN
    // ====================================

    if (nilaiCahaya <= batasTerik) {

      statusCuaca =
        "MATAHARI TERIK (PENCAHAYAAN TINGGI)";

      statusLcd = "TINGGI";

      parameterField =
        "&field1=" + String(nilaiCahaya);
    }

    else if (nilaiCahaya <= batasMendung) {

      statusCuaca =
        "MENDUNG/BERAWAN (PENCAHAYAAN SEDANG)";

      statusLcd = "SEDANG";

      parameterField =
        "&field2=" + String(nilaiCahaya);
    }

    else {

      statusCuaca =
        "GELAP/MALAM (PENCAHAYAAN RENDAH)";

      statusLcd = "RENDAH";

      parameterField =
        "&field3=" + String(nilaiCahaya);
    }

    // ====================================
    // SERIAL MONITOR
    // ====================================
    Serial.println();
    Serial.println("================================");

    Serial.println("MONITORING INTENSITAS CAHAYA");

    Serial.print("Nilai ADC: ");
    Serial.println(nilaiCahaya);

    Serial.print("Kategori: ");
    Serial.println(statusCuaca);

    Serial.print("RSSI WiFi: ");

    if (WiFi.status() == WL_CONNECTED) {
      Serial.print(WiFi.RSSI());
      Serial.println(" dBm");
    }

    else {
      Serial.println("Tidak terhubung");
    }

    // ====================================
    // UPDATE LCD I2C
    // ====================================
    lcd.clear();

    lcd.setCursor(0, 0);
    lcd.print("LDR ADC: ");
    lcd.print(nilaiCahaya);

    lcd.setCursor(0, 1);
    lcd.print("CAHAYA: ");
    lcd.print(statusLcd);

    // ====================================
    // KIRIM DATA KE THINGSPEAK
    // ====================================
    kirimThingSpeak(parameterField);

    Serial.println("================================");
  }
}
