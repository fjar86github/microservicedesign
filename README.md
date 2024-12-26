#Rancangan Aplikasi Microservice 

"Microservices", kata kunci yang sedang tren saat ini. Masalah apa yang dipecahkannya?

Banyak organisasi saat ini mendorong migrasi teknologi dan salah satu tantangan terbesar adalah bermigrasi dari aplikasi monolitik ke aplikasi berbasis layanan mikro.

Proyek contoh layanan mikro ini menunjukkan bagaimana beberapa layanan berjalan secara independen dengan memanfaatkan pola layanan mikro terbaik untuk memungkinkan skalabilitas, kinerja, dan ketahanan.

![image](https://github.com/user-attachments/assets/9ab55fd4-46f4-4805-9ea7-07b0ed450df2)

#Studi Kasus
Aplikasi contoh memiliki dua layanan yaitu layanan-satu dan layanan-dua. Masing-masing layanan memiliki basis data layanan-satu-db dan layanan-dua-db sendiri. Selama memulai layanan, layanan tersebut menyimpan nama layanan dan UUID yang dibuat secara otomatis dalam basis data perspektifnya dan mengirimkan data ke bursa RabbitMQ yang kemudian menyiarkan data ke semua antrean berdasarkan kunci perutean. Setiap layanan mikro mendengarkan antrean RabbitMQ-nya sendiri dan terus memperbarui basis data saat menerima data.

Berikut ini adalah layar aplikasi.

![image](https://github.com/user-attachments/assets/e7aed40b-2962-47df-bba7-be8f822eb6c4)
Dengan mengklik tab satu atau dua, data yang Anda lihat di layar didasarkan pada data yang diambil oleh layanan masing-masing dengan memanggil basis datanya.
![image](https://github.com/user-attachments/assets/1c5841fb-3122-41d2-98f3-92ee289c602e)
Perhatikan bahwa UUID yang dihasilkan untuk layanan-satu yang terletak di layanan-satu-db sinkron dengan tab layanan-dua yang dicapai oleh RabbitMQ (komunikasi asinkron antara layanan mikro).
![image](https://github.com/user-attachments/assets/d37026c6-db86-485f-a31f-ba93a0593542)

#Pendaftaran Layanan
Selama inisialisasi suatu layanan, layanan tersebut akan didaftarkan ke server penemuan dan registrasi (yang dalam contoh kita adalah Konsul Hashicorp).
![image](https://github.com/user-attachments/assets/6b742cab-154c-4b6c-8a92-afb58d072183)

#Penemuan Layanan
Ketika satu layanan (misalnya api-gateway) perlu mengakses sumber daya dari layanan lain (misalnya layanan-satu), yang harus dilakukannya hanyalah meminta server penemuan dan pendaftaran (Consul) untuk memberikan salah satu informasi instansi layanan-satu.
![image](https://github.com/user-attachments/assets/a0fa3a38-45c4-4260-9c8c-5fffd39c4c86)

#Arsitektur
Di bawah ini adalah diagram arsitektur untuk proyek layanan mikro.
![image](https://github.com/user-attachments/assets/a2ad4ab0-7e4f-4d7c-b590-2374d767c412)

#Komponen Terintegrasi & Penggunaan Alat
API Gateway
Netflix Zuul adalah server proxy terbalik yang bertindak sebagai Gateway API untuk mengakses layanan mikro di balik gateway yang mengarahkan permintaan ke layanan terkait. Layanan mikro tetap berada di balik server proxy terbalik dan perlu dikonsumsi melalui gateway api. Layanan mikro api-gateway dengan profil docker berjalan pada port 8080 dan dapat diakses melalui http://localhost:8080 .

Konfigurasi yang dilakukan di API Gateway untuk Routing:
zuul:
  ignoredServices: '*'
  routes:
    one:
      path: /service-one/**
      serviceId: Service-One
    two:
      path: /service-two/**
      serviceId: Service-Two

Pendaftaran dan penemuan layanan
Registrasi dan penemuan ditangani oleh Konsul HashiCorp. Selama permulaan layanan individual, layanan tersebut mendaftarkan detail seperti nama host, port, dll. yang dapat digunakan untuk mengakses layanan tersebut ke layanan registrasi layanan. Setelah layanan didaftarkan ke konsul, konsul memeriksa kesehatan layanan dengan mengirimkan detak jantung untuk jalur pemeriksaan kesehatan dan interval pemeriksaan kesehatan yang telah didaftarkan ke Konsul. Permintaan ke layanan mikro harus dirutekan melalui api-gateway selama layanan penemuan kontak api-gateway untuk mendapatkan informasi yang diperlukan guna mengirim permintaan ke layanan mikro yang dituju.

Konfigurasi yang dilakukan di layanan mikro untuk mendaftar ke Konsul:
management:
  contextPath: /actuator

spring:
  application.name: service-one
  cloud:
    consul:
      host: consul
      port: 8500
      discovery:
        hostName: service-one
        instanceId:${spring.application.name}:${spring.application.instance_id:${random.value}}
        healthCheckPath: ${management.contextPath}/health
        healthCheckInterval: 15s

Konsol manajemen konsul dapat diakses di http://localhost:8500/ui/
![image](https://github.com/user-attachments/assets/1e92292a-b027-43c1-967a-c417aec73b27)

#Pemantauan dan visualisasi
Pemantauan, visualisasi & pengelolaan kontainer di docker dilakukan oleh weave scope.

Konsol manajemen Weavescope dapat diakses di http://localhost:4040/
![image](https://github.com/user-attachments/assets/18284b65-40f8-4d45-a601-b0d787d32a25)

Pencatatan terpusat menggunakan ELK
Logback yang terintegrasi dengan setiap layanan mikro membuat log aplikasi dan mengirim data log ke server pencatatan (Logstash). Logstash memformat data dan mengirimkannya ke server pengindeksan (Elasticsearch). Data yang disimpan di server elasticsearch dapat divisualisasikan dengan indah menggunakan Kibana.

Mengakses alat:

Elasticsearch: http://localhost:9200/_search?pretty

Kibana: http://localhost:5601/app/kibana

![image](https://github.com/user-attachments/assets/2d36121f-ece8-4462-bb23-931a7af5b8b1)
![image](https://github.com/user-attachments/assets/711353d3-f07a-4c71-986d-eb3237b55f26)
![image](https://github.com/user-attachments/assets/0cde715c-4ffc-4c4c-a950-d650c3b60573)
![image](https://github.com/user-attachments/assets/21d90173-e74e-4be9-8a3c-4139bcb7041a)
![image](https://github.com/user-attachments/assets/3a9cade5-f547-4db9-a9fa-cb6219ec0d68)
![image](https://github.com/user-attachments/assets/99542394-554f-4a20-bb78-94fc06954f94)
![image](https://github.com/user-attachments/assets/40d5481e-e5cf-4b9e-8564-145705ffdc0e)
![image](https://github.com/user-attachments/assets/242d9092-716f-4dba-be5d-6adf6c93f744)
![image](https://github.com/user-attachments/assets/55d66943-c261-42f2-868f-fe761f3e702c)

#Komunikasi layanan mikro asinkron
Interkomunikasi antar layanan mikro terjadi secara asinkron dengan bantuan RabbitMQ.
Konsol RabbitMQ dapat diakses di http://localhost:15672/

![image](https://github.com/user-attachments/assets/70723f27-4b7f-42e3-948c-2cc310cc3e20)
![image](https://github.com/user-attachments/assets/5027815f-6289-454d-b7f9-c9c6f60b1343)
![image](https://github.com/user-attachments/assets/d7aaa38b-4c7a-4240-a2af-06852168d699)

#Teknologi
Proyek layanan mikro menggunakan sejumlah proyek sumber terbuka agar berfungsi dengan baik:

SpringBoot - Kerangka kerja aplikasi
Zuul - API Gateway (Load Balancer)
Consul - Pendaftaran dan Penemuan Layanan
Docker - Platform Kontainerisasi
RabbitMQ - Perpesanan layanan mikro asinkron.
Logstash - Pengumpul log
Elasticsearch - Pengindeks log
Kibana - Visualisasi data
Angular - HTML yang disempurnakan untuk aplikasi web!
Bootstrap - Boilerplate UI yang hebat untuk aplikasi web modern
jQuery - Traversal dan manipulasi dokumen HTML
Swagger - Dokumentasi API

Alat
Java - Pemrograman
Maven - Membangun
Git - Kontrol versi
Docker - Penerapan

#Pengembangan
Berikut adalah langkah-langkah untuk membuka lingkungan pengembangan dan memulai.
Instal Git, Java, Maven, dan Docker
Untuk proyek menggunakan https://github.com/mudigal-technologies/microservices-sample.git
Kloning fork menggunakan https://github.com/{YOUR_GIT_ID}/microservices-sample.git
Jalankan "cd /microservices-sample/build/docker/scripts/"
Untuk menyebarkan docker, jalankan "./deploy.sh docker".
Akses Aplikasi di http://localhost/

Tambahan
Mysql pada service-two-db:
$ docker exec -it service-two-db mysql -u service-two -p
Enter password: service-two

mysql> USE service-two;
Database changed

mysql> SELECT * FROM name_value;


