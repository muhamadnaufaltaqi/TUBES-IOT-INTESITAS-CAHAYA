### Sistem Monitoring Intesitas cahaya pada Tanaman

### Anggota Kelompok
1.  Muhammad Naufal Taqi	(2311101080)	Perakitan perangkat dan pengumpulan dataset monitoring LDR.
2.  Mokhammad Arief F	(2311101100)	Konfigurasi protokol, eksperimen HTTP/MQTT, dan dokumentasi Wireshark. 
3.  Ramadan Aditia	(2311101097)	Pengolahan data, analisis hasil, dan penyusunan laporan.

### Deskripsi Singkat
Intensitas cahaya merupakan salah satu faktor lingkungan yang berperan dalam proses fotosintesis dan pertumbuhan tanaman hias. Kondisi pencahayaan di sekitar tanaman dapat berubah mengikuti waktu, cuaca, serta kondisi lingkungan. Pemantauan secara manual memiliki keterbatasan karena pengukuran dan pencatatan harus dilakukan secara langsung. Oleh sebab itu, sistem monitoring berbasis Internet of Things (IoT) dikembangkan untuk merekam perubahan tingkat pencahayaan secara berkala.
Sistem yang dibuat menggunakan sensor Light Dependent Resistor (LDR) dan mikrokontroler ESP32. Perubahan resistansi LDR menghasilkan perubahan tegangan pada rangkaian pembagi tegangan. Tegangan tersebut dibaca oleh ADC ESP32 melalui GPIO 34, diproses menjadi kategori pencahayaan, ditampilkan pada LCD, lalu dikirimkan ke ThingSpeak melalui jaringan Wi-Fi menggunakan HTTP GET 
Tujuan kegiatan ini adalah merancang sistem pembacaan intensitas cahaya, menampilkan hasil pada LCD, mengirim data ke ThingSpeak, mengumpulkan dataset lebih dari tujuh hari, serta membandingkan kinerja pengiriman HTTPS POST dan MQTT pada enam skenario eksperimen. Sistem digunakan sebagai alat bantu pengamatan. Penentuan kebutuhan cahaya tanaman tertentu tetap perlu mempertimbangkan jenis tanaman dan acuan agronomis yang sesuai.

### Pembandingan HTTP vs MQTT
Eksperimen membandingkan pengiriman data menggunakan HTTPS POST dan MQTT. Variabel yang diubah adalah protokol komunikasi, ukuran payload, serta tingkat QoS pada MQTT. Parameter yang diamati meliputi keberhasilan pengiriman, Packet Delivery Ratio (PDR), Round Trip Time (RTT), Received Signal Strength Indicator (RSSI), serta estimasi overhead komunikasi.
Berdasarkan kode eksperimen, perangkat ESP32 dijalankan pada simulator Wokwi dan menggunakan jaringan Wi-Fi Wokwi-GUEST. HTTPS POST dikirim ke https://httpbin.org/post melalui port 443. MQTT menggunakan broker test.mosquitto.org pada port 1883 dengan topik wokwi/ldr/exp4827. Pengujian protokol ini terpisah dari pengumpulan dataset LDR melalui perangkat fisik dan ThingSpeak.


### Struktur Direktori
`#include <WiFi.h>
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
// Isi dengan kredensial WiFi kamu
// ========================================
const char* ssid = "Jendral Wito";
const char* password = "jemputjamtigapagi";

// ========================================
// KONFIGURASI THINGSPEAK
// ========================================
const char* myWriteAPIKey = "KEPMDYIB1BLUPYQX";

String serverName = "http://api.thingspeak.com/update";

// ========================================
// BATAS ADC SENSOR LDR
// ========================================
const int batasTerik = 1000;
const int batasMendung = 3000;

// ========================================
// INTERVAL PENGIRIMAN
// ========================================
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
  } else {
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
    } else {
      Serial.println("ThingSpeak Menolak Data!");
    }
  } else {
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

  lcd.setCursor(0, 0);
  lcd.print("MONITORING LDR");

  lcd.setCursor(0, 1);
  lcd.print("TANAMAN HIAS");

  delay(2000);

  koneksiWiFi();

  // Pengukuran pertama langsung dilakukan
  waktuSebelumnya = millis() - interval;
}

// ========================================
// LOOP UTAMA
// ========================================
void loop() {
  koneksiWiFi();

  if (millis() - waktuSebelumnya >= interval) {
    waktuSebelumnya = millis();

    // Membaca sensor
    int nilaiCahaya = bacaSensorLDR();

    String statusLCD = "";
    String statusKategori = "";
    String parameterField = "";

    // ====================================
    // KLASIFIKASI PENCAHAYAAN
    // ====================================

    if (nilaiCahaya <= batasTerik) {

      statusKategori = "MATAHARI TERIK";
      statusLCD = "MATAHARI TERIK";

      parameterField =
        "&field1=" + String(nilaiCahaya);
    }

    else if (nilaiCahaya <= batasMendung) {

      statusKategori = "MENDUNG/BERAWAN";
      statusLCD = "MENDUNG/BERAWAN";

      parameterField =
        "&field2=" + String(nilaiCahaya);
    }

    else {

      statusKategori = "GELAP/MALAM";
      statusLCD = "GELAP/MALAM";

      parameterField =
        "&field3=" + String(nilaiCahaya);
    }

    // ====================================
    // SERIAL MONITOR
    // ====================================
    Serial.println();
    Serial.println("==============================");
    Serial.println("MONITORING INTENSITAS CAHAYA");

    Serial.print("Nilai ADC: ");
    Serial.println(nilaiCahaya);

    Serial.print("Kategori: ");
    Serial.println(statusKategori);

    Serial.print("RSSI WiFi: ");

    if (WiFi.status() == WL_CONNECTED) {
      Serial.print(WiFi.RSSI());
      Serial.println(" dBm");
    } else {
      Serial.println("Tidak terhubung");
    }

    // ====================================
    // TAMPILAN LCD I2C
    // ====================================
    lcd.clear();

    lcd.setCursor(0, 0);
    lcd.print("LDR ADC: ");
    lcd.print(nilaiCahaya);

    lcd.setCursor(0, 1);
    lcd.print(statusLCD);

    // ====================================
    // KIRIM DATA KE THINGSPEAK
    // ====================================
    kirimThingSpeak(parameterField);

    Serial.println("==============================");
  }
}`
