### Sistem Monitoring Intesitas cahaya pada Tanaman Hias

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
