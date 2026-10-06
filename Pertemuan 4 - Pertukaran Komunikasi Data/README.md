# Modul 4: Komunikasi dan Pertukaran Data pada IoT

## Library dan Dependencies
Untuk menjalankan program ini, diperlukan beberapa *library* berikut yang harus diinstal pada Arduino IDE:
1. **`ESP8266WiFi.h`**
2. **`PubSubClient.h`**
3. **`ArduinoJson.h`**
4. **`DHT.h`** (Untuk Percobaan 4B)

---

## Percobaan 4A: Subscribe dan Deserialisasi Data JSON
### Detail singkat Percobaan
Pada percobaan 4A ini, membahas penerapan komunikasi MQTT dengan mekanisme subscribe untuk menerima perintah dari broker. Pada percobaan ini, ESP8266 terhubung ke broker MQTT dan melakukan subscribe pada topic yang telah ditentukan. Ketika terdapat pesan masuk, fungsi callback() akan menerima dan membaca data tersebut, kemudian mengubah payload menjadi bentuk String dan melakukan proses deserialisasi JSON. Jika format JSON valid, program mengambil nilai pada bagian perintah. Perintah "ON" digunakan untuk menyalakan LED, sedangkan perintah "OFF" digunakan untuk mematikan LED. Hasil pesan yang diterima dan kondisi aktuator juga ditampilkan melalui Serial Monitor. 

### Penjelasan Fungsi Utama
* `hubungkanWiFi()`: Menghubungkan ESP8266 ke jaringan Wi-Fi agar perangkat dapat melakukan komunikasi MQTT.

* `hubungkanMQTT()`: Menghubungkan perangkat ke broker MQTT dan melakukan `subscribe` pada topic perintah setelah koneksi berhasil.

* `callback(char* topic, byte* payload, unsigned int length)`: Menangani pesan MQTT yang diterima, membaca payload, mengubahnya menjadi String, menampilkan pesan pada Serial Monitor, serta memproses data JSON.

* `deserializeJson()`: Melakukan parsing atau deserialisasi pesan JSON yang diterima dan memeriksa apakah format JSON valid.

* `doc["perintah"]`: Mengambil nilai `perintah` dari pesan JSON untuk menentukan tindakan yang dilakukan oleh LED.

* `digitalWrite()`: Mengatur kondisi LED berdasarkan perintah yang diterima. Perintah `"ON"` menyalakan LED dengan `HIGH`, sedangkan `"OFF"` mematikan LED dengan `LOW`.

* `setup()`: Melakukan inisialisasi awal program, seperti mengatur pin LED, menghubungkan Wi-Fi dan MQTT, serta mendaftarkan fungsi `callback()`.

* `loop()`: Menjalankan proses utama secara berulang dan memanggil `client.loop()` agar komunikasi MQTT tetap berjalan serta pesan yang masuk dapat diproses.

### Penjelasan Percabangan (Conditional)

* `if (error)`: Memeriksa apakah terjadi kesalahan saat proses deserialisasi JSON. Jika terjadi kesalahan, program menampilkan pesan `Gagal parsing JSON` dan menghentikan proses dengan `return`.

* `if (String(perintah) == "ON")`: Memeriksa apakah nilai `perintah` yang diterima adalah `"ON"`. Jika benar, LED dinyalakan dengan `digitalWrite(ledPin, HIGH)` dan Serial Monitor menampilkan `Aktuator: ON`.

* `else if (String(perintah) == "OFF")`: Memeriksa apakah nilai `perintah` adalah `"OFF"`. Jika benar, LED dimatikan dengan `digitalWrite(ledPin, LOW)` dan Serial Monitor menampilkan `Aktuator: OFF`.

* `else`: Pada program Percobaan 4A tidak terdapat percabangan `else` setelah pengecekan `"OFF"`, sehingga apabila nilai `perintah` bukan `"ON"` atau `"OFF"`, tidak ada perubahan kondisi pada LED.

### Jawaban Pertanyaan Praktikum 4A (Modifikasi PWM)
Untuk menambahkan fitur pengatur intensitas kecerahan LED melalui PWM berdasarkan pesan JSON seperti `{"perintah": "ON", "intensitas": 200}`, berikut adalah modifikasi kode pada fungsi `callback`:

```cpp
// ... kode bagian atas fungsi callback tetap sama ...
  
  // Deserialisasi data JSON yang diterima
  JsonDocument doc;
  DeserializationError error = deserializeJson(doc, pesan);
  if (error) {
    Serial.print("Gagal parsing JSON: ");
    Serial.println(error.c_str());
    return;
  }

  const char* perintah = doc["perintah"];
  int intensitas = doc["intensitas"]; // TAMBAHAN: Mengambil nilai "intensitas" berformat integer dari objek JSON

  if (String(perintah) == "ON") {
    analogWrite(ledPin, intensitas); // TAMBAHAN: Menggunakan analogWrite (PWM) untuk mengatur kecerahan LED (0-1023 untuk ESP8266)
    Serial.print("Aktuator: ON, Intensitas: ");
    Serial.println(intensitas); // TAMBAHAN: Mencetak nilai intensitas ke Serial Monitor
  } else if (String(perintah) == "OFF") {
    analogWrite(ledPin, 0); // TAMBAHAN: Mematikan LED dengan mengatur duty cycle PWM ke 0
    Serial.println("Aktuator: OFF");
  }
}
```

---

## Percobaan 4B: Pertukaran Data Dua Arah (Full Duplex)

### Detail singkat Percobaan 4B
Pada percobaan 4B menerapkan komunikasi **full duplex** menggunakan protokol MQTT, yaitu ESP8266 melakukan proses `publish` data suhu dari sensor DHT11 sekaligus `subscribe` perintah untuk mengendalikan LED. Data suhu dibaca dan dikirim ke broker MQTT setiap 5 detik menggunakan fungsi `millis()` sebagai mekanisme **non-blocking**. Sementara itu, fungsi `client.loop()` digunakan untuk menjaga komunikasi MQTT dan menerima pesan perintah secara berkala. Pesan yang diterima diproses melalui fungsi `callback()` dan dilakukan deserialisasi JSON. Perintah `"ON"` digunakan untuk menyalakan LED, sedangkan `"OFF"` digunakan untuk mematikannya. Dengan mekanisme tersebut, proses pengiriman data sensor dan penerimaan perintah dapat berjalan secara bersamaan tanpa saling menghambat. Penggunaan `client.loop()` secara berkala sangat penting karena fungsi tersebut menangani komunikasi MQTT dan memungkinkan perangkat menerima pesan dari broker secara responsif. Oleh karena itu, penggunaan `delay()` dengan durasi yang panjang dihindari agar proses penerimaan perintah tidak terhambat selama perangkat melakukan pembacaan dan pengiriman data suhu.

### Penjelasan Percabangan Khusus (Non-Blocking & Ternary)
*   `if (millis() - waktuTerakhirPublish > intervalPublish)`: Merupakan logika *non-blocking delay*. Program mengecek selisih waktu saat ini dengan waktu terakhir data dikirim. Jika selisihnya lebih dari 5000 ms, maka sensor dibaca dan data di-publish. Ini memastikan `client.loop()` tidak pernah berhenti.
*   `if (!isnan(suhu))`: Percabangan untuk validasi. Memastikan angka suhu yang dibaca bukanlah *Not a Number* (NaN) sebelum dikirimkan.
*   `String(perintah) == "ON" ? HIGH : LOW`: Ini adalah *Ternary Operator* (versi singkat dari IF-ELSE). Artinya: "Jika perintah adalah ON, kembalikan nilai HIGH, selain itu kembalikan LOW". Hasilnya langsung dimasukkan ke parameter `digitalWrite()`.

### Jawaban Pertanyaan Praktikum 4B (Modifikasi Tambah Topik Aktuator Kedua)
Untuk menambahkan kontrol aktuator kedua (misal: Buzzer) dengan topik MQTT terpisah, kita harus memodifikasi deklarasi variabel, pendaftaran subscribe, dan mengecek variabel `topic` pada fungsi `callback`.

**Modifikasi Kode:**
```cpp
// 1. TAMBAHAN DEKLARASI GLOBAL (di bagian atas program)
const char* topicBuzzer = "unsoed/tk245004/kelompok4/buzzer"; // Definisi topik khusus untuk buzzer
const int buzzerPin = 12; // (misal GPIO12/D6) Deklarasi pin untuk aktuator kedua

// 2. TAMBAHAN DI DALAM FUNGSI setup()
pinMode(buzzerPin, OUTPUT); // Mengatur pin buzzer sebagai output

// 3. TAMBAHAN DI DALAM FUNGSI hubungkanMQTT()
if (client.connect(clientId.c_str())) {
  client.subscribe(topicPerintah); // (Sudah ada) Subscribe LED
  client.subscribe(topicBuzzer);   // TAMBAHAN: Subscribe ke topik Buzzer setelah koneksi MQTT terjalin
  Serial.println("Terhubung dan subscribe topik LED dan Buzzer");
}

// 4. MODIFIKASI FUNGSI callback()
void callback(char* topic, byte* payload, unsigned int length) {
  String pesan;
  for (unsigned int i = 0; i < length; i++) pesan += (char) payload[i];
  
  JsonDocument doc;
  if (deserializeJson(doc, pesan)) return;

  const char* perintah = doc["perintah"];

  // TAMBAHAN: Membedakan aksi berdasarkan 'topic' dari pesan yang masuk
  if (String(topic) == String(topicPerintah)) {
    // Blok dieksekusi HANYA jika pesan berasal dari topik LED
    digitalWrite(ledPin, String(perintah) == "ON" ? HIGH : LOW);
    Serial.print("Perintah LED diterima: ");
    Serial.println(perintah);
  } 
  else if (String(topic) == String(topicBuzzer)) {
    // TAMBAHAN: Blok dieksekusi HANYA jika pesan berasal dari topik Buzzer
    digitalWrite(buzzerPin, String(perintah) == "ON" ? HIGH : LOW);
    Serial.print("Perintah Buzzer diterima: ");
    Serial.println(perintah);
  }
}
```
*Penjelasan Modifikasi:* Pada fungsi `callback`, pesan MQTT yang diterima terlebih dahulu diperiksa berdasarkan nilai `topic`. Program kemudian menggunakan percabangan `if-else if` untuk menentukan apakah pesan berasal dari `topicPerintah` atau `topicBuzzer`. Dengan cara tersebut, setiap pesan hanya akan diproses oleh bagian program yang sesuai dengan topic-nya, sehingga perintah pada topic buzzer tidak akan memengaruhi kondisi LED.

foto rangkaian saat praktikum :

percobaan 4a :

<img width="411" height="501" alt="image" src="https://github.com/user-attachments/assets/328f8b48-432a-41de-8b83-99fc39b54077" />

percobaan 4b :

<img width="340" height="547" alt="image" src="https://github.com/user-attachments/assets/8f227e8a-3b69-496d-8c97-aa453702d54f" />
