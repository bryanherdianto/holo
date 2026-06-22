# HOLO (Horizon-Oriented Laser Output)

Projek Embedded System ini dibuat oleh Bryan Herdianto, Filaga Tifira Muthi, Jesie Tenardi, dan Muhammad Riyan Satrio Wibowo.

## Introduction to the Problem and the Solution

Saat ini, sistem penunjukan arah menjadi bagian yang penting dalam berbagai bidang, seperti astronomi, sistem kendali, hingga teknologi informasi. Sistem ini biasanya memanfaatkan sistem koordinat Azimuth (horizontal) dan Altitude (vertikal) sebagai acuan untuk menunjuk arah secara akurat. Akan tetapi, penunjukan arah secara manual dengan tenaga manusia tidak efisien dan rawan terhadap human error, dimana penunjukan arah seringkali tidak akurat.

Sebagai solusi dari masalah yang telah disebutkan, kami mengimplementasikan Arduino untuk melakukan otomasi proses penunjukan arah. Terdapat dua Arduino yang kami gunakan, yaitu Arduino master yang berfungsi untuk menerima input pengguna dan Arduino slave yang berfungsi untuk menggerakkan servo. Kami menggunakan keypad untuk menerima input derajat azimuth dan altitude dari pengguna. Nilai input dari pengguna ditampilkan dengan serial display.

Data input dari pengguna dikirimkan dari master ke slave untuk mengendalikan servo yang terhubung dengan PWM. Kalibrasi dilakukan untuk memastikan sudut perputaran servo sesuai dengan input dari pengguna. Kami juga mengintegrasikan reset button untuk mengembalikan servo ke posisi awal. Implementasi seluruh fungsionalitas ini diharapkan dapat menghasilkan sistem penunjukan arah yang akurat dan efisien, serta minim intervensi.

## Hardware Design and Implementation Details

Dalam membuat sistem penunjukan arah “HOLO,” dibutuhkan sejumlah komponen yang perlu diselaraskan. Terdapat 2 buah Arduino yang digunakan dimana satu Arduino berperan sebagai master dan Arduino lainnya berperan sebagai slave. Pada Arduino master, terdapat sebuah keypad yang berfungsi untuk menerima input derajat dari pengguna. Keypad tersebut sekaligus berfungsi untuk mengganti mode (azimuth/altitude) dan men-trigger pergerakan servo. Input tersebut ditampilkan dengan serial display MAX7219 yang dihubungkan dengan protokol Serial Peripheral Interface (SPI) dengan format “Azm/Alt (spasi) Derajat.”

Pada Arduino slave, terdapat 2 servo 180° untuk azimuth dan altitude, serta reset button untuk mengembalikan posisi servo. Untuk menghubungkan kedua Arduino, digunakan protokol Inter-Integrated Circuit (I2C). Arduino master akan mengirimkan input pengguna kepada Arduino slave setiap pengguna menekan tombol ❉ atau #. Arduino slave kemudian mengolah input pengguna untuk menggerakkan servo sesuai derajat yang diinput oleh pengguna.

## Software Implementation Details

### Arduino Master

Program utama menginisialisasi SPI, I2C, dan konfigurasi input/output serta membaca input dari keypad. Mode tampilan (azimuth atau altitude) diatur dan ditampilkan, kemudian program menunggu input dari pengguna.

### Arduino Slave

Arduino slave mengatur input/output untuk servo dan interrupt, serta menginisialisasi komunikasi I2C sebagai slave. Program menunggu data dari master untuk kemudian memproses dan menggerakkan servo.

### Keypad Input

Keypad dikonfigurasi dengan baris sebagai output dan kolom sebagai input, dibaca menggunakan metode scanning dan debouncing. Input numerik disusun sebagai bilangan multidigit, sedangkan tombol khusus digunakan untuk mengubah mode atau mengirim data melalui I2C.

### Display MAX7219

Display diatur melalui protokol SPI, termasuk pengaturan kecerahan, decoding, dan scan limit. Data azimuth atau altitude ditampilkan sesuai mode menggunakan subroutine `send_bytes`.

### Kontrol Servo

Data sudut yang diterima dari master diubah menjadi nilai PWM menggunakan interpolasi linear melalui subroutine `angle_to_timer`. Nilai PWM ini digunakan untuk menghasilkan sinyal 1–2 ms dengan periode 20 ms yang sesuai standar kontrol servo.

### Interrupt Handling

Tombol eksternal pada pin INT0 dikonfigurasi untuk mendeteksi falling edge sebagai pemicu interrupt. Saat ditekan, interrupt handler akan mengeksekusi proses toggling pada pin PB2 untuk menyalakan/mematikan Laser Module KY-008.

### Komunikasi Master-Slave

Komunikasi dilakukan melalui protokol I2C dengan kecepatan 400 kHz dan urutan standar START → ADDRESS → DATA → STOP. Program mencakup pemeriksaan flag untuk memastikan keberhasilan setiap tahap komunikasi.

## Test Results and Performance Evaluation

### Testing in Proteus

Sebelumnya, perlu dipahami bahwa dalam simulasi ini, servo motor yang digunakan adalah positional servo motor 180 derajat. Komponen servo pada Proteus menampilkan -90 derajat sebagai posisi 0 derajat. Lalu, 90 derajat menunjukkan posisi 180 derajat.

Berikut adalah hasil run awal-awal dari sistem secara simulasi di Proteus:
![picture 0](https://i.imgur.com/Y6PgssB.png)  
Dari gambar di atas, dapat dilihat bahwa sistem menunjukkan mode azimuth terlebih dulu dengan tampilan 0 derajat. Untuk memasukkan derajat yang diinginkan, user dapat menekan tombol 1-9 pada keypad. Setelah itu, user dapat menekan tombol ❉ untuk mengirimkan data ke Arduino slave agar menggerakkan servo sesuai dengan derajat yang diinginkan.

Berikut adalah hasil memasukkan derajat azimuth untuk menggerakkan servo:

| Input                                         | Output                                        |
| --------------------------------------------- | --------------------------------------------- |
| ![picture 1](https://i.imgur.com/FAnzoEm.png) | ![picture 2](https://i.imgur.com/KH8ZWkX.png) |

Dari tabel di atas, kita bisa melihat bahwa user telah memasukkan derajat azimuth sebesar 165 derajat. Display MAX7219 menampilkan mode azimuth dengan sudut 165 derajat. Setelah itu, user menekan tombol ❉ untuk mengirimkan data ke Arduino slave. Setelah itu, SERVO1 bergerak ke sudut 165 derajat (-90 + 165 = 75). SERVO1 menunjukkan sudut +75.8 derajat yang tidak benar-benar pas dengan sudut 165 derajat. Hal ini karena ketidakakuratan dari kalibrasi PWM yang dibuat. Meskipun begitu, hal ini masih dalam batas toleransi yang dapat diterima.

Berikut adalah hasil memasukkan derajat altitude untuk menggerakkan servo:

| Input                                         | Output                                        |
| --------------------------------------------- | --------------------------------------------- |
| ![picture 3](https://i.imgur.com/WUXQdE7.png) | ![picture 4](https://i.imgur.com/GrFwWBg.png) |

Dari tabel di atas, kita bisa melihat bahwa user telah memasukkan derajat altitude sebesar 84 derajat. Display MAX7219 menampilkan mode altitude dengan sudut 84 derajat. Setelah itu, user menekan tombol ❉ untuk mengirimkan data ke Arduino slave. Setelah itu, SERVO2 bergerak ke sudut 84 derajat (-90 + 84 = -6). Sama seperti sebelumnya, kalibrasi belum sepenuhnya akurat, tetapi masih dalam batas toleransi yang dapat diterima.

Berikut adalah hasil menekan pushbutton menyalakan Laser Module KY-008:
![picture 5](https://i.imgur.com/rZmFtPN.png)

Karena menggunakan interrupt, maka ketika tombol ditekan, Laser Module KY-008 akan menyala secara real-time. Ketika tombol dilepas, Laser Module KY-008 akan mati secara real-time. Interrupt ini memungkinkan sistem untuk mengontrol Laser Module KY-008 tanpa ada delay.

Mode akan berubah dari azimuth ke altitude dan balik ke azimuth lagi. Maka, setelah melakukan input pada mode altitude, user dapat kembali ke mode azimuth. Berikut adalah hasilnya:

| Input                                         | Output                                        |
| --------------------------------------------- | --------------------------------------------- |
| ![picture 6](https://i.imgur.com/bClcpQW.png) | ![picture 7](https://i.imgur.com/o2Cp99c.png) |

Dari tabel di atas, kita bisa melihat bahwa user telah memasukkan derajat azimuth sebesar 48 derajat. Display MAX7219 menampilkan mode azimuth dengan sudut 48 derajat. Setelah itu, user menekan tombol ❉ untuk mengirimkan data ke Arduino slave. Setelah itu, SERVO1 bergerak ke sudut 48 derajat (-90 + 48 = -42). Sama seperti sebelumnya, kalibrasi belum sepenuhnya akurat, tetapi masih dalam batas toleransi yang dapat diterima.

### Testing in Physical Circuit

Kalibrasi yang dilakukan pada sistem ini adalah kalibrasi manual. Servo motor yang digunakan pada proyek ini ada dua, yaitu servo motor fisik dan servo motor simulasi. Kalibrasi yang dilakukan berbeda untuk kedua servo itu. Oleh karena itu, terdapat dua kode dengan kalibrasi yang berbeda (silakan melakukan comment / uncomment pada bagian kalibrasi sesuai dengan servo yang digunakan). Selain itu, terdapat juga perbedaan sedikit pada rangkaian fisik. Rangkaian fisik menggunakan resistor pull-up 400 ohm pada pin SDA dan SCL serta resistor pull-up 10k ohm pada pin CS untuk display MAX7219. Kedua resistor berfungsi agar sistem dapat berjalan dengan baik dan lancar. Berikut adalah foto rangkaian fisik dan resistor pull-up yang digunakan:

| Rangkaian Keseluruhan                          | Resistor Pull-Up                               |
| ---------------------------------------------- | ---------------------------------------------- |
| ![picture 8](https://i.imgur.com/LnR2SIc.jpeg) | ![picture 9](https://i.imgur.com/tO0lgpl.jpeg) |

| Rangkaian Keseluruhan 2.0                                                                         |
| ------------------------------------------------------------------------------------------------- |
| ![gambar_fisik8](https://github.com/user-attachments/assets/7de682b5-f1fe-4e28-a9e6-c50397319f6c) |

| Pan Tilt Servo POV #1                                                                             | Pan Tilt Servo POV #2                                                                             |
| ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| ![gambar_fisik7](https://github.com/user-attachments/assets/f333ce52-dae9-4303-add2-3cb79d2e29e8) | ![gambar_fisik6](https://github.com/user-attachments/assets/15e22923-bb4e-40bd-8492-1a02572071a9) |

## Conclusion and Future Work

Proyek ini telah berhasil melaksanakan fungsi-fungsi yang sesuai pada rangkaian fisik dan juga pada simulasi di Proteus. Berikut fitur-fitur yang berhasil dimiliki oleh sistem:

- Mampu mengintegrasikan komunikasi I2C antara Arduino master dan slave.
- Mampu mengontrol sudut servo secara PWM.
- Arduino dapat memproses input dari keypad dan menampilkan hasil input tersebut ke tampilan MAX7219.
- Servo digerakkan secara presisi dengan sinyal PWM hasil interpolasi sudut
- Sistem juga dilengkapi fitur interrupt untuk mengontrol Laser Module KY-008 secara real-time.

Meskipun demikian, terdapat beberapa hal yang perlu diperbaiki dan ditingkatkan pada sistem ini, antara lain:

- Selain menggunakan Laser Module KY-008, sistem bisa menggunakan sebuah kamera agar bisa melakukan targeting visual secara tiga dimensi.
- Motor servo dapat dikontrol secara lebih akurat lagi dengan menggunakan kode yang dapat memproses PWM secara desimal.
- Motor servo yang digunakan untuk sumbu azimuth dapat berupa motor servo 360° agar dapat berputar penuh.
- Kode dapat memproses data lebih besar dari 255 derajat untuk azimuth, baik secara komunikasi I2C maupun saat menggerakkan servo.
