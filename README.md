# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

Menurut pemahaman saya, untuk kasus BambangShop kita tidak wajib membuat interface/trait `Subscriber` seperti di diagram klasik Observer, karena saat ini perilaku subscriber kita masih sangat spesifik: menyimpan `url` dan mengirim notifikasi HTTP ke endpoint tertentu. Satu `struct Subscriber` sudah cukup untuk kebutuhan sekarang. Namun, trait tetap berguna kalau ke depannya kita ingin ada banyak tipe observer dengan perilaku berbeda (misalnya subscriber HTTP, subscriber message queue, atau mock subscriber untuk testing). Jadi, kesimpulan saya: saat ini struct tunggal cukup, tapi trait lebih baik untuk extensibility jangka panjang dan agar kode lebih sesuai Open/Closed Principle.

Untuk keunikan `id` pada Program dan `url` pada Subscriber, menurut saya `Vec` sebenarnya bisa dipakai kalau datanya kecil, tetapi efisiensinya kurang bagus karena perlu pencarian linear $O(n)$ untuk cek duplikasi, update, atau delete. Sementara struktur map seperti `DashMap` (atau `HashMap` pada konteks single-thread) memberi akses berbasis key rata-rata $O(1)$, sehingga lebih cocok untuk data yang punya constraint unik. Selain performa, map juga lebih "natural" secara model data karena key unik langsung dipetakan ke value. Jadi untuk kasus ini, map lebih tepat dibanding list biasa.

Terkait thread safety, `Singleton` dan `DashMap` menurut saya menyelesaikan masalah yang berbeda. Singleton hanya mengatur bahwa instance globalnya satu, tapi tidak otomatis membuat akses datanya aman saat multithread. Sebaliknya, `DashMap` fokus pada concurrent access yang aman dan efisien. Di Rust, karena server Rocket bisa melayani request secara paralel, kita tetap butuh struktur data thread-safe untuk state global seperti daftar subscriber. Artinya, kita bisa saja menerapkan pola singleton untuk "satu sumber data", tetapi isi penyimpanannya tetap perlu mekanisme sinkronisasi (`DashMap`, `RwLock<HashMap<...>>`, dll). Jadi singleton saja tidak cukup menggantikan kebutuhan `DashMap`.

#### Reflection Publisher-2

Menurut saya, pemisahan `Service` dan `Repository` dari `Model` diperlukan supaya tanggung jawab tiap komponen jelas (Single Responsibility Principle). `Repository` fokus ke urusan akses data (simpan, ambil, hapus), sedangkan `Service` fokus ke business logic/use case aplikasi. Dengan pemisahan ini, model tetap menjadi representasi data/domain object yang relatif bersih. Dampaknya cukup terasa: kode lebih mudah diuji (service bisa di-test dengan mock repository), lebih mudah di-maintain, dan perubahan di layer data (misalnya pindah dari in-memory ke database sungguhan) tidak terlalu merusak layer bisnis/controller.

Kalau hanya pakai Model, kemungkinan besar setiap model akan jadi "gemuk" karena mencampur data, business rule, sekaligus detail penyimpanan. Dalam konteks `Program`, `Subscriber`, dan `Notification`, interaksinya bisa saling tarik-menarik: `Program` harus tahu cara ambil subscriber dan kirim notifikasi, `Subscriber` mungkin ikut tahu detail format notifikasi, `Notification` bisa ikut menangani penyimpanan/lookup juga. Akhirnya coupling antarmodel naik, dependency jadi berputar, dan perubahan kecil di satu model bisa memicu perubahan berantai di model lain. Menurut saya ini bikin kompleksitas meningkat cepat, terutama saat fitur bertambah.

Saya sudah eksplor Postman lebih jauh, dan tool ini sangat membantu untuk ngetes endpoint secara cepat tanpa bikin client manual. Untuk pekerjaan sekarang, Postman memudahkan saya cek skenario subscribe, unsubscribe, create/delete product, lalu verifikasi apakah response status/body sudah sesuai ekspektasi. Fitur yang menurut saya paling kepakai dan menarik untuk project ke depan:
1. Collection untuk mengelompokkan endpoint per modul/fitur.
2. Environment variables supaya base URL, token, atau parameter bisa diganti tanpa edit request satu-satu.
3. Tests tab (script) untuk assertion otomatis setelah request, jadi bisa dipakai sebagai regression check ringan.
4. Collection Runner untuk menjalankan banyak request berurutan.
5. Export/Share collection agar satu tim punya standar pengujian API yang sama.

Menurut saya, kombinasi fitur-fitur ini cukup powerful untuk bantu workflow QA dasar di group project, bahkan sebelum punya automated integration test yang lebih formal di pipeline CI/CD.

#### Reflection Publisher-3

Pada tutorial ini, menurut saya kita memakai variasi **Push model**. Alasannya, publisher (BambangShop) langsung mengirim data notifikasi ke subscriber lewat HTTP POST saat event terjadi (misalnya create/delete product). Jadi subscriber tidak perlu request dulu untuk ambil update; informasi didorong dari publisher ke subscriber.

Kalau dibayangkan kita memakai **Pull model**, ada beberapa kelebihan dan kekurangan.
Kelebihan:
1. Payload notifikasi awal bisa lebih ringan, karena publisher cukup kirim sinyal "ada update" lalu subscriber yang mengambil detail jika perlu.
2. Subscriber punya kontrol kapan dan data apa yang mau diambil, jadi lebih fleksibel untuk kebutuhan masing-masing client.

Kekurangan:
1. Kompleksitas sistem naik karena harus menyediakan endpoint tambahan untuk "fetch detail update".
2. Potensi latency bertambah karena jadi dua langkah (dapat sinyal dulu, lalu fetch data), bukan satu kali kirim selesai.
3. Traffic request bisa lebih banyak saat subscriber banyak dan sering polling/pull.
4. Konsistensi data bisa lebih tricky kalau data berubah lagi sebelum subscriber selesai pull.

Menurut saya, untuk kasus tutorial yang fokusnya notifikasi sederhana berbasis event, Push lebih straightforward dan mudah dipahami.

Jika kita tidak memakai multi-threading di proses notifikasi, program tetap bisa jalan, tapi performanya kemungkinan menurun. Pengiriman notifikasi ke subscriber akan cenderung serial (satu per satu), sehingga total response time endpoint yang memicu notify bisa ikut lebih lama, apalagi kalau ada subscriber yang lambat atau timeout. Efeknya throughput turun dan user bisa merasakan API lebih "lemot" saat traffic naik. Selain itu, satu subscriber bermasalah dapat menjadi bottleneck untuk alur notifikasi lainnya. Jadi, tanpa multi-threading implementasinya memang lebih sederhana, tetapi kurang scalable untuk skenario real-world yang concurrent.
