## 1. Penjelasan code
Percobaan 1 :

- #include <DHT.h> → Memanggil library sensor DHT.
- #define DHTPIN 4 → Menentukan GPIO 4 sebagai pin data sensor.
- #define DHTTYPE DHT22 → Menentukan jenis sensor yang digunakan, yaitu DHT22.
- DHT dht(DHTPIN, DHTTYPE); → Membuat objek sensor DHT22.
- void setup() → Menjalankan proses inisialisasi satu kali.
- Serial.begin(115200); → Mengatur komunikasi Serial dengan baud rate 115200.
- dht.begin(); → Menginisialisasi sensor DHT22.
- Serial.println(...) → Menampilkan pesan awal pada Serial Monitor.
- void loop() → Menjalankan program secara berulang.
- dht.readHumidity(); → Membaca nilai kelembaban.
- dht.readTemperature(); → Membaca nilai suhu.
- isnan(...) → Memeriksa apakah data sensor valid atau tidak.
- if (...) → Menentukan kondisi berdasarkan hasil pembacaan sensor.
- Serial.print(...) → Menampilkan data tanpa pindah baris.
- Serial.println(...) → Menampilkan data kemudian pindah ke baris berikutnya.
- delay(2000); → Memberikan jeda pembacaan selama 2 detik.

Percobaan 2 :
- #include <DHT.h> → Memanggil library sensor DHT.
- #define DHTPIN 4 → Menentukan GPIO 4 sebagai pin data DHT22.
- #define DHTTYPE DHT22 → Menentukan jenis sensor yang digunakan, yaitu DHT22.
- #define RELAYPIN 26 → Menentukan GPIO 26 sebagai pin kendali relay/LED.
- DHT dht(DHTPIN, DHTTYPE); → Membuat objek sensor DHT22.
- const float suhuThreshold = 30.0; → Menentukan ambang batas suhu sebesar 30°C.
- void setup() → Menjalankan proses inisialisasi satu kali.
- Serial.begin(115200); → Mengatur komunikasi Serial dengan baud rate 115200.
- dht.begin(); → Menginisialisasi sensor DHT22.
- pinMode(RELAYPIN, OUTPUT); → Mengatur GPIO 26 sebagai output.
- digitalWrite(RELAYPIN, LOW); → Memastikan relay/LED dalam kondisi mati saat awal.
- void loop() → Menjalankan program secara berulang.
- dht.readTemperature(); → Membaca nilai suhu dari sensor DHT22.
- isnan(suhu) → Memeriksa apakah data suhu valid atau tidak.
- if (isnan(suhu)) → Menentukan kondisi ketika pembacaan suhu gagal.
- Serial.println("Gagal membaca data sensor!"); → Menampilkan pesan kesalahan pada Serial Monitor.
- Serial.print("Suhu: "); → Menampilkan tulisan "Suhu:".
- Serial.print(suhu); → Menampilkan nilai suhu yang terbaca.
- Serial.print(" °C -> "); → Menampilkan satuan suhu Celsius.
- if (suhu > suhuThreshold) → Memeriksa apakah suhu melebihi 30°C.
- digitalWrite(RELAYPIN, HIGH); → Mengaktifkan relay/LED.
- Serial.println("Aktuator: ON"); → Menampilkan status aktuator menyala.
- else → Menjalankan perintah jika suhu tidak melebihi batas.
- digitalWrite(RELAYPIN, LOW); → Mematikan relay/LED.
- Serial.println("Aktuator: OFF"); → Menampilkan status aktuator mati.
  
---
## 2. Penjelasan setiap fungsi
a. Percobaan 1 : Percobaan ini dilakukan untuk memperoleh informasi mengenai kondisi suhu dan kelembaban lingkungan dengan memanfaatkan sensor DHT11. Nilai yang diperoleh dari sensor akan diproses oleh mikrokontroler ESP32, kemudian hasil pembacaannya ditampilkan pada Serial Monitor melalui laptop.

b. Percobaan 2 : Percobaan kedua merupakan pengembangan dari percobaan pertama dengan menambahkan komponen Relay/LED sebagai aktuator. Program pada mikrokontroler dirancang agar dapat menentukan kondisi aktuator berdasarkan hasil pengukuran suhu. Aktuator akan dikendalikan secara otomatis dengan membandingkan suhu yang terukur terhadap nilai batas (threshold) yang telah ditentukan, yaitu 30°C.

---
## 3. Penjelasan percabangan/conditional
a. Percobaan 1 :
```
if (isnan(kelembaban) || isnan(suhu)) {
  Serial.println("Gagal membaca data dari sensor DHT22!");
} else {
  // Cetak nilai suhu dan kelembaban
}
```
Struktur `if-else` digunakan untuk memeriksa validitas data hasil pembacaan sensor. Apabila nilai suhu atau kelembaban menghasilkan `NaN` (*Not a Number*), program akan menampilkan pesan **"Gagal"**. Sebaliknya, apabila data yang diperoleh valid atau tidak bernilai `NaN`, program akan menjalankan blok `else` untuk menampilkan nilai suhu dan kelembaban pada layar.

b. Percobaan 2 :
```
if (suhu > suhuThreshold) {
  digitalWrite(RELAYPIN, HIGH);
  Serial.println("Aktuator: ON");
} else {
  digitalWrite(RELAYPIN, LOW);
  Serial.println("Aktuator: OFF");
}
```
Di dalam blok else (jika pembacaan sensor berhasil), terdapat percabangan sekunder. Jika nilai suhu lebih besar dari 30 derajat celsius, maka mikrokontroler mengirimkan sinyal HIGH untuk menyalakan Relay/LED. Jika kondisi tersebut tidak terpenuhi (suhu <= 30 derajat celsius), maka mikrokontroler mengirimkan sinyal LOW untuk mematikan aktuator.

---
## 4. Library atau dependencies yang diperlukan 
- DHT sensor library (oleh Adafruit)
- Board esp8266 esp32
- Driver CP20X untuk esp8266

---
## 5. Jawaban pertanyaan praktikum yang berkaitan dengan code
### A. Modifikasi Percobaan 1: Rata-rata 5 Kali Pembacaan
Untuk meningkatkan akurasi, program dimodifikasi agar mengambil 5 sampel data, menjumlahkannya, dan membaginya dengan 5 untuk mendapatkan nilai rata-rata sebelum ditampilkan.

**Penambahan/Modifikasi Kode Utama pada `void loop()`:**
```
float sumSuhu = 0;
  float sumKelembaban = 0;
  
  // Melakukan 5 kali pembacaan
  for(int i = 0; i < 5; i++) {
    sumKelembaban += dht.readHumidity();
    sumSuhu += dht.readTemperature();
    delay(2000); 
  }

  // Menghitung rata-rata
  float rataSuhu = sumSuhu / 5.0;
  float rataKelembaban = sumKelembaban / 5.0;
```
*    `float sumSuhu = 0;` : Mendeklarasikan variabel sumSuhu bertipe float dengan nilai awal 0 untuk menampung total akumulasi nilai suhu.
*    `float sumKelembaban = 0;` : Mendeklarasikan variabel sumKelembaban bertipe float dengan nilai awal 0 untuk menampung total akumulasi nilai kelembaban.
*    `for(int i = 0; i < 5; i++) {` : Memulai perulangan (looping) yang akan dieksekusi sebanyak 5 kali (indeks 0 hingga 4).
*    `sumKelembaban += dht.readHumidity();` : Membaca nilai kelembaban dari sensor saat itu, lalu menambahkannya (+=) ke dalam variabel sumKelembaban.
*    `sumSuhu += dht.readTemperature();` : Membaca nilai suhu dari sensor saat itu, lalu menambahkannya (+=) ke dalam variabel sumSuhu.
*    `delay(2000);` : Memberikan jeda 2 detik pada setiap iterasi pembacaan agar sensor DHT memiliki waktu yang cukup untuk memperbarui data hardware-nya sebelum dibaca kembali.
*    `float rataSuhu = sumSuhu / 5.0;` : Membuat variabel baru rataSuhu yang nilainya didapat dari total penjumlahan suhu (sumSuhu) dibagi 5.0.
*    `float rataKelembaban = sumKelembaban / 5.0;` : Membuat variabel baru rataKelembaban yang nilainya didapat dari total penjumlahan kelembaban (sumKelembaban) dibagi 5.0.

### B. Modifikasi Percobaan 2: Kendali Aktuator dengan Histerisis (Dua Ambang Batas)Penambahan/Modifikasi Kode Utama (menggantikan variabel suhuThreshold tunggal):
```
const float suhuBatasAtas = 30.0;
const float suhuBatasBawah = 28.0;

// Di dalam void loop(), logika kontrol diubah menjadi:
if (suhu > suhuBatasAtas) {
  digitalWrite(RELAYPIN, HIGH);
  Serial.println("Aktuator: ON (Suhu > 30°C)");
} else if (suhu < suhuBatasBawah) {
  digitalWrite(RELAYPIN, LOW);
  Serial.println("Aktuator: OFF (Suhu < 28°C)");
}
```
*   `const float suhuBatasAtas = 30.0;` : Mendefinisikan konstanta batas atas suhu di angka 30.0°C. Ini adalah titik di mana aktuator akan mulai menyala.
*   `const float suhuBatasBawah = 28.0;` : Mendefinisikan konstanta batas bawah suhu di angka 28.0°C. Ini adalah titik di mana aktuator akan dimatikan.
*   `if (suhu > suhuBatasAtas) {` : Percabangan kondisi pertama. Mengecek apakah nilai suhu yang baru saja dibaca lebih besar dari 30.0°C.
*   `digitalWrite(RELAYPIN, HIGH);` : Jika kondisi di atas benar (suhu > 30), ESP32 mengirim sinyal tegangan (HIGH) ke pin relay untuk menyalakan aktuator.
*   `Serial.println("Aktuator: ON (Suhu > 30°C)");` : Menampilkan informasi ke Serial Monitor bahwa aktuator sedang dalam kondisi ON.
*   `} else if (suhu < suhuBatasBawah) {` : Kondisi alternatif (histerisis). Jika suhu tidak lebih dari batas atas, program mengecek apakah suhu turun lebih kecil dari 28.0°C.
*   `digitalWrite(RELAYPIN, LOW);` : Jika suhu berada di bawah 28.0°C, ESP32 memutus sinyal (LOW) pada pin relay sehingga aktuator mati.
*   `Serial.println("Aktuator: OFF (Suhu < 28°C)");` : Menampilkan informasi ke Serial Monitor bahwa aktuator telah OFF.

---
## 6. Skematik/diagram rangkaian jika diperlukan
a. Percobaan 1 :

<img width="362" height="562" alt="image" src="https://github.com/user-attachments/assets/130b640d-1209-4b5a-95f4-cd88746bfc54" />

b. Percobaan 2 :

<img width="536" height="572" alt="image" src="https://github.com/user-attachments/assets/8a1a7050-6155-4bb4-956d-0291b5bfaaa7" />

---
## 7. Foto proses praktikum/perangkaian :
a. Percobaan 1 :

<img width="491" height="370" alt="image" src="https://github.com/user-attachments/assets/623f2b26-95de-471e-a08a-edd98340046c" />

b. Percobaan 2 : 

<img width="407" height="461" alt="image" src="https://github.com/user-attachments/assets/58f5fe92-b997-4f41-bae3-147b0019b324" />

