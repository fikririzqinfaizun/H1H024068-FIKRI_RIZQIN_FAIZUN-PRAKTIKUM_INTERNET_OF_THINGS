## 1. Penjelasan code dan setiap fungsi
### Fungsi Utama Program Arduino

- `setup()`: Bagian program yang hanya diproses satu kali ketika ESP32 mulai dijalankan. Fungsi ini digunakan untuk melakukan persiapan awal, seperti membuka komunikasi Serial, menentukan konfigurasi pin, mengatur mode WiFi, serta menjalankan proses koneksi atau mengaktifkan Access Point.

- `loop()`: Bagian program yang akan dieksekusi secara berulang selama ESP32 dalam keadaan aktif. Pada mode STA, bagian ini digunakan untuk mengecek kondisi koneksi WiFi secara berkala. Sementara pada mode AP, `loop()` digunakan untuk mengetahui jumlah perangkat yang sedang terhubung ke Access Point dengan interval pemeriksaan setiap 5 detik.

### Fungsi pada Library WiFi ESP32

- `WiFi.mode()`: Digunakan untuk menentukan mode kerja WiFi pada ESP32. Mode `WIFI_STA` digunakan ketika ESP32 berperan sebagai client, sedangkan `WIFI_AP` digunakan ketika ESP32 menyediakan jaringan WiFi sendiri.

- `WiFi.begin(ssid, password)`: Menjalankan proses penyambungan ESP32 ke jaringan WiFi yang tersedia dengan menggunakan SSID dan password yang telah ditentukan.

- `WiFi.softAP(ap_ssid, ap_password)`: Membuat dan mengaktifkan jaringan WiFi pada ESP32 dalam mode Access Point berdasarkan SSID dan password yang diberikan.

- `WiFi.status()`: Digunakan untuk mengetahui keadaan koneksi WiFi ESP32. Salah satu kondisi yang diperiksa adalah `WL_CONNECTED`, yang menunjukkan bahwa perangkat telah berhasil terhubung ke jaringan.

- `WiFi.localIP()`: Mengambil alamat IP yang diperoleh ESP32 ketika terhubung sebagai Station pada jaringan WiFi yang tersedia.

- `WiFi.softAPIP()`: Digunakan untuk mendapatkan alamat IP yang digunakan oleh Access Point ESP32. Pada konfigurasi yang digunakan dalam praktikum, alamat tersebut adalah `192.168.4.1`.

- `WiFi.macAddress()`: Mengambil alamat MAC dari interface WiFi ESP32 yang digunakan sebagai identitas perangkat dalam jaringan.

- `WiFi.RSSI()`: Digunakan untuk membaca tingkat kekuatan sinyal WiFi yang diterima ESP32. Hasil pengukuran dinyatakan dalam satuan dBm.

- `WiFi.softAPgetStationNum()`: Digunakan untuk mengetahui jumlah perangkat atau client yang sedang terhubung ke Access Point yang dibuat oleh ESP32.

## 2. Penjelasan percabangan/conditional 
Pada Percobaan 2A (Mode STA), logika kondisi digunakan untuk mengatur proses koneksi serta memantau keadaan jaringan Wifi ESP32 selama program berjalan.

- while (WiFi.status() != WL_CONNECTED): Digunakan untuk melakukan pengulangan selama ESP32 belum berhasil terhubung ke jaringan WiFi. Pada setiap pengulangan, program berhenti sementara selama 500 milidetik menggunakan delay(500), kemudian menampilkan karakter titik (.) pada Serial Monitor sebagai tanda bahwa proses koneksi masih berlangsung.
- if (WiFi.status() == WL_CONNECTED) { ... } else { ... }: Digunakan untuk menentukan kondisi koneksi WiFi pada fungsi loop(). Apabila status menunjukkan WL_CONNECTED, program menampilkan informasi "Status: Terhubung". Sebaliknya, jika kondisi tersebut tidak terpenuhi, program menampilkan "Status: Terputus" dan mengatur LED indikator menjadi mati menggunakan digitalWrite(ledPin, LOW).

## 3. Library atau dependencies yang diperlukan
- <ESP8266WiFi.h>

## 4. Jawaban pertanyaan praktikum yang berkaitan dengan code
### 1. Modifikasi Percobaan 1: Fitur Auto-Reconnect pada Mode Station
**Kode Modifikasi (Ditambahkan pada blok `loop()`):**
```cpp
void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("Status: Terhubung");
    digitalWrite(ledPin, HIGH); // Pastikan LED menyala
  } else {
    Serial.println("Status: Terputus. Menghubungkan ulang...");
    digitalWrite(ledPin, LOW);  // Matikan LED
    
    // Logika percobaan menghubungkan ulang
    WiFi.disconnect(); 
    WiFi.begin(ssid, password);
    
    // Tahan dan tunggu proses reconnect selesai
    while (WiFi.status() != WL_CONNECTED) {
      delay(500);
      Serial.print(".");
    }
    Serial.println("\nWiFi berhasil terhubung kembali!");
  }
  delay(5000);
}
```
Penjelasan Kode :
- WiFi.disconnect(); : Digunakan untuk memutus koneksi WiFi yang sedang aktif sebelum ESP32 mencoba melakukan koneksi kembali.
- WiFi.begin(ssid, password); : Menjalankan kembali proses koneksi ESP32 ke jaringan WiFi dengan menggunakan SSID dan password yang telah ditentukan.
- while (WiFi.status() != WL_CONNECTED) { : Membuat perulangan yang akan terus dilakukan selama ESP32 belum berhasil terhubung ke jaringan WiFi.
- delay(500); : Memberikan jeda selama 500 milidetik pada setiap proses percobaan koneksi agar pengecekan tidak dilakukan secara terus-menerus.
- Serial.print("."); : Menampilkan karakter titik pada Serial Monitor sebagai tanda bahwa ESP32 masih melakukan proses koneksi ulang.
- } : Menandai akhir dari blok perintah yang terdapat di dalam perulangan while.
- Serial.println("\nWiFi berhasil terhubung kembali!"); : Menampilkan pesan bahwa ESP32 telah berhasil terhubung kembali ke jaringan WiFi. Simbol \n digunakan untuk membuat tampilan pesan dimulai dari baris baru.

### 2. Modifikasi Percobaan 2: Konfigurasi Mode AP + STA (Gabungan)
**Kode Modifikasi Keseluruhan:**
```cpp
#include <ESP8266WiFi.h>

// Kredensial jaringan untuk mode STA (Sebagai Klien/Terhubung ke internet rumah)
const char* ssid = "ESP32_RASTA";
const char* password = "12345678";

// Kredensial jaringan untuk mode AP (Sebagai Penyedia/Hotspot)
const char* ap_ssid = "ESP32_RASTA";
const char* ap_password = "12345678";

void setup() {
  Serial.begin(115200);
  
  // 1. Set mode gabungan AP dan STA
  WiFi.mode(WIFI_AP_STA);
  
  // 2. Inisialisasi proses koneksi Station (STA)
  WiFi.begin(ssid, password);
  Serial.print("Menghubungkan ke WiFi rumah...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi STA terhubung!");
  Serial.print("IP Address STA: ");
  Serial.println(WiFi.localIP());

  // 3. Inisialisasi pemancaran Access Point (AP)
  WiFi.softAP(ap_ssid, ap_password);
  Serial.println("Access Point aktif!");
  Serial.print("IP Address AP : ");
  Serial.println(WiFi.softAPIP());
}

void loop() {
  // Mengecek jumlah perangkat yang numpang (terhubung) ke jaringan AP lokal
  int jumlahClient = WiFi.softAPgetStationNum();
  Serial.print("Jumlah klien di AP: ");
  Serial.println(jumlahClient);
  
  delay(5000);
}
```
Penjelasan Kode :
- WiFi.mode(WIFI_AP_STA); : Mengatur ESP32 agar bekerja menggunakan dua mode WiFi sekaligus, yaitu sebagai Station yang terhubung ke jaringan utama dan sebagai Access Point yang menyediakan jaringan untuk perangkat lain.
- WiFi.begin(ssid, password); : Memulai proses ESP32 untuk bergabung ke jaringan WiFi yang tersedia dengan menggunakan SSID dan password yang telah ditentukan.
- WiFi.localIP() (digunakan dalam Serial.println) : Digunakan untuk menampilkan alamat IP yang diperoleh ESP32 setelah berhasil terhubung ke jaringan dalam mode Station.
- WiFi.softAP(ap_ssid, ap_password); : Mengaktifkan ESP32 sebagai Access Point dengan membuat jaringan WiFi baru berdasarkan nama SSID dan password yang telah ditentukan.
- WiFi.softAPIP() (digunakan dalam Serial.println) : Digunakan untuk menampilkan alamat IP yang dimiliki ESP32 pada jaringan Access Point, sehingga dapat diketahui alamat jaringan yang digunakan oleh perangkat yang terhubung.

## 5. Foto proses praktikum/perangkaian
<img width="1200" height="1600" alt="image" src="https://github.com/user-attachments/assets/07d99f19-5d37-4e8a-a6ed-753de1391ded" />

