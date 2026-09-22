# Modul 3: Protokol Komunikasi IoT

## 1. Penjelasan Singkat Percobaan
- Percobaan 3A (Komunikasi HTTP): Melakukan implementasi komunikasi antara mikrokontroler ESP8266 dengan server melalui protokol HTTP. Pada percobaan ini, ESP8266 mengirimkan data sensor berupa nilai suhu dan kelembaban yang masih menggunakan data dummy ke endpoint httpbin.org/post menggunakan metode POST. Informasi sensor tersebut dikemas dalam bentuk data JSON sebelum dikirimkan ke server.
- Percobaan 3B (Komunikasi MQTT): Menerapkan komunikasi berbasis protokol MQTT dengan mekanisme publish-subscribe. ESP8266 berperan sebagai perangkat yang mengirimkan data sensor dalam format JSON melalui proses publish ke broker MQTT publik broker.hivemq.com. Data tersebut dikirim secara berkala pada topic yang telah ditentukan.

## 2. Library / Dependencies
Library yang dibutuhkan untuk menjalankan program ini antara lain:
- ESP8266WiFi.h`
- ESP8266HTTPClient.h`
- WiFiClientSecure.h`
- ArduinoJson.h`
- PubSubClient.h`

## 3. Penjelasan Code & Fungsi

### Pada percobaan 3A (HTTP) :
- setup(): Bagian program yang hanya dieksekusi satu kali ketika ESP8266 mulai bekerja. Pada bagian ini dilakukan pengaturan komunikasi Serial dengan baud rate 115200 serta proses awal untuk menghubungkan perangkat ke jaringan WiFi.
- loop(): Merupakan fungsi yang dijalankan secara berulang selama perangkat aktif. Program akan mempersiapkan koneksi HTTP/HTTPS, kemudian membuat data JSON yang berisi informasi suhu dan kelembaban. Data JSON tersebut dikonversi menjadi bentuk string menggunakan serializeJson, kemudian dikirim ke server melalui fungsi http.POST().
- client.setInsecure(): Perintah yang digunakan pada WiFiClientSecure untuk menonaktifkan pemeriksaan sertifikat SSL. Dengan demikian, ESP8266 dapat melakukan komunikasi dengan endpoint HTTPS tanpa melakukan validasi terhadap sertifikat keamanan server.

### Pada percobaan 3B (MQTT) :
- hubungkanWiFi(): Merupakan fungsi tambahan yang dibuat untuk menangani proses penyambungan ESP8266 ke jaringan WiFi berdasarkan SSID dan password yang telah ditentukan.
- hubungkanMQTT(): Fungsi yang bertugas menghubungkan ESP8266 dengan broker MQTT. Apabila koneksi belum berhasil, program akan terus melakukan percobaan koneksi ulang. Pada setiap percobaan dapat digunakan Client ID yang dibuat secara acak agar perangkat memiliki identitas yang berbeda.
- setup(): Digunakan untuk melakukan konfigurasi awal perangkat, seperti mengaktifkan komunikasi Serial, menjalankan fungsi hubungkanWiFi(), serta menentukan alamat dan port broker MQTT menggunakan client.setServer().
- loop(): Berfungsi menjalankan proses utama secara berulang. Program akan mengecek koneksi perangkat dengan broker MQTT, menjalankan client.loop() agar koneksi tetap terpelihara, kemudian membentuk data sensor dalam format JSON dan mengirimkannya melalui client.publish() dengan interval pengiriman setiap 5 detik.

### 4. Penjelasan Percabangan (Conditionals) :
- while (WiFi.status() != WL_CONNECTED): Digunakan untuk melakukan perulangan selama ESP8266 belum mendapatkan koneksi WiFi. Selama proses tersebut berlangsung, program memberikan jeda menggunakan delay() dan menampilkan tanda titik sebagai indikator bahwa perangkat masih mencoba terhubung.
- if (WiFi.status() == WL_CONNECTED): Pada Percobaan 3A, kondisi ini digunakan untuk mengecek status koneksi WiFi terlebih dahulu. Proses pengiriman data melalui HTTP hanya dilanjutkan apabila ESP8266 telah berhasil tersambung ke jaringan.
- if (httpResponseCode > 0): Digunakan untuk mengevaluasi hasil respons setelah HTTP request dikirim. Apabila nilainya lebih besar dari 0, berarti proses komunikasi dengan server mendapatkan respons dan request tidak mengalami kegagalan pada sisi koneksi jaringan.
- while (!client.connected()): Pada Percobaan 3B, perulangan ini digunakan untuk memastikan ESP8266 tetap mencoba melakukan koneksi ke broker MQTT selama status perangkat masih belum terhubung.
- if (client.connect(clientId.c_str())): Kondisi ini digunakan untuk menentukan apakah proses penyambungan ESP8266 ke broker MQTT berhasil. Jika koneksi berhasil, program akan menampilkan informasi bahwa perangkat telah tersambung. Sebaliknya, apabila koneksi gagal, bagian else akan dijalankan untuk menampilkan informasi mengenai kode kesalahan dan memberikan jeda sebelum proses koneksi dicoba kembali.

---

## 5. Jawaban Pertanyaan Praktikum (Percobaan 3A)
**Tugas:** Modifikasi program agar dapat mengirimkan data tambahan berupa waktu (dalam milidetik sejak dinyalakan menggunakan `millis()`) ke dalam JSON yang dikirim, dan berikan penjelasan di setiap baris kode yang ditambahkan.

**Modifikasi Kode:**

Untuk menambahkan data `millis()`, modifikasi dilakukan pada blok pembuatan objek JSON di dalam fungsi `loop()`:

```cpp
// Membuat objek data sensor dalam format JSON
JsonDocument doc;
doc["suhu"] = 28.5;
doc["kelembaban"] = 65.0;

// -- BARIS YANG DITAMBAHKAN --
doc["waktu"] = millis(); 
// ---------------------------

String requestBody;
serializeJson(doc, requestBody);
```

**Penjelasan Baris Kode yang Ditambahkan:**
*   `doc["waktu"] = millis();` 
    *   `doc["waktu"]` : Perintah ini menginstruksikan objek `JsonDocument` untuk membuat pasangan *key* baru dengan nama `"waktu"`.
    *   `=` : Operator *assignment* untuk mengisi nilai ke dalam *key* tersebut.
    *   `millis()` : Memanggil fungsi internal Arduino yang berfungsi menghitung dan mengembalikan jumlah waktu dalam satuan milidetik (ms) semenjak *board* mikrokontroler pertama kali dialiri listrik dan mulai menjalankan program.
    *   **Kesimpulan:** Secara keseluruhan, baris ini menambahkan *timestamp* relatif ke dalam paket data JSON, sehingga saat diterima oleh server, formatnya menjadi `{"suhu":28.5, "kelembaban":65.0, "waktu":15000}` (contoh jika data dikirim pada detik ke-15).

### 6. Foto proses saat praktikum :
Foto Proses Praktikum Pada Percobaan 3A :
<img width="1112" height="675" alt="image" src="https://github.com/user-attachments/assets/0f4d5973-3b07-4406-b4cf-c8d6bdebecfb" />

Foto Proses Praktikum Pada Percobaan 3B :
<img width="1562" height="667" alt="image" src="https://github.com/user-attachments/assets/43fc90f7-7683-491d-b066-b06f6ead7229" />

<img width="1656" height="552" alt="image" src="https://github.com/user-attachments/assets/e5862394-335c-43e5-bd20-b7f989917608" />
