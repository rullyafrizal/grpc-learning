# gRPC


## RPC (Remote Procedure Call)
- RPC atau kepanjangan dari Remote Procedure Call adalah sebuah protokol komunikasi atau mekanisme komunikasi yang digunakan oleh suatu program untuk melakukan request ke suatu service yang terletak di server lain tanpa harus mengetahui detail dari server yang akan diakses.
- RPC menggunakan model client-server. Program yang mengirim request adalah client dan yang mengirim response adalah server.
- RPC adalah operasi yang bersifat synchronous yang memerlukan client untuk menunggu sampai server selesai mengirim data. Akan tetapi, juga ada penggunaan threading untuk mengirim request secara concurrent

### Interface Definition Language (IDL)
- IDL adalah sebuah bahasa pemrograman untuk definisi protokol komunikasi.
- IDL menyediakan jembatan bagi kedua program untuk berkomunikasi, meskipun kedua program tersebut katakanlah berjalan dalam OS yang berbeda.

### What does RPC do?
- Ketika kita compile suatu aplikasi yang menggunakan RPC framework, sebuah **stub** akan dimasukkan ke dalam aplikasi yang nantinya akan berfungsi sebagai representasi dari remote procedure code.
- Ketika program atau aplikasi dijalankan, dan procedure call dijalankan, **stub** menerima request dan men-forward ke client runtime.

### Cara Kerja RPC
![Diagram RPC](/img/operating-system-remote-call-procedure-working.png)

- Client memanggil client stub, ini statusnya local procedure call dengan parameter yang dibawa ke stack seperti biasa
- Client stub tadi membawa parameter procedurenya ke dalam **message** dan membuat system call untuk mengirim message ke server. Proses membawa message yang berisi parameter procedure disebut sebagai **marshalling**
- OS local mengirim message ke remote server machine.
- OS dari remote server menerima message dan langsung diantar ke server stub.
- Di dalam server stub, message tadi di-unboxing parameternya (ini disebut **unmarshalling**)
- Remote Server langsung proses request
- Ketika udah selesai, server langsung angkut datanya ke server stub, yang nantinya di-marshal lagi ke message. Kalo udah di-marshal, langsung diserahin ke transport layer.
- Transport layer langsung deh kirim message tadi balik ke client, yang nantinya message bakalan di-unboxing atau di-unmarshal sama si client dan diproses lebih lanjut di client.


### Beberapa Jenis Konfigurasi RPC
- Client bikin call dan ga akan lanjut sampe server bales (ini standardnya kaya gini)
- Client bikin call, dan lanjut process hal lain, server gak harus langsung reply dan client gak harus nunggu
- Client bisa kirim beberapa non-blocking call dalam satu paket atau batch
- Client bisa punya broadcast facility i.e., mereka bisa kirim message ke banyak server terus terima semua resultnya
- Client bikin non-blocking client/server call. Habis itu server kirim sinyal ke client dengan cara call ke prosedur yang terasosiasi dengan client.


### Keuntungan Menggunakan RPC
- Membantu client berkomunikasi dengan server via prosedur yang tradisional pake high-level language
- Sangat cocok digunakan di distributed environment (microservice).
- Support process-oriented dan thread-oriented model
- Bisa menyembunyikan internal message yang dikirimkan ke server dari user
- Effortnya tidak terlalu besar untuk rewrite dan redevelop code-nya.
- Menyediakan abstraksi.
- Get rid atau omit atau menyingkirkan beberapa protocol layer, ini bisa berdampak pada peningkatan performa.

### Downside dari RPC
- Tidak cocok digunalan untuk transfer data yang gede parah, karena client dan server menggunakan lingkungan eksekusi yang beda, makanya itu penggunaan resource bisa beda2 dan kompleks.
- Highly vulnerable to failure, karena menyangkut dua sistem yang saling berkomunikasi.
- Belum ada standardisasi untuk RPC, implementasinya bisa beda-beda tiap developer yang make.


## gRPC
![gRPC Diagram](/img/grpc.svg)
### Apa itu gRPC?
- Prinsipnya sama seperti RPC, kita define service, tentuin methodnya, dan kita define parameternya.
- Di sisi server, server sendiri mengimplementasikan interface dan menjalankan gRPC server guna menghandle request atau call dari client
- Di sisi client, client punya stub yang menyediakan method yang sama dengan yang ada di sisi server

### Sekilas Tentang Protocol Buffer
- Protocol Buffers adalah sebuah mekanisme untuk serialisasi data terstruktur yang bersifat language dan platform neutral.
- Protocol Buffers mirip dengan konsep XML tetapi lebih kecil dalam hal ukuran, lebih cepat, dan lebih sederhana
- Secara default, gRPC menggunakan **protocol buffers** sebagai serializer, meski kita bisa menggunakan JSON.

#### Contoh Sederhana Protocol Buffers
- Data dari protocol buffers terstruktur dalam bentuk message
```
message Person {
  string name = 1;
  int32 id = 2;
  bool has_ponycopter = 3;
}
```
- Representasi dari message akan berbeda tiap bahasa pemrograman, misal di Go akan menjadi struct, di C++ akan menjadi Class, dsb.
- Define gRPC services dengan method RPC parameter dan return value sebagai message
```
// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

### Mendefinisikan Service
- Seperti kebanyakan sistem RPC, ide utama gRPC adalah mendefinisikan service, menentukan method dengan paramtere dan return type-nya. Secara default, gRPC menggunakan Protocol Buffers sebagai Interface Definition Language (IDL). IDL di sini beruna untuk mendefinisikan service interface serta struktur data atau message.

```
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

### Jenis Service Method
- **Unary RPC**: Method yang hanya mengirimkan satu request ke server dan mendapatkan satu repsonse, seperti function pada umumnya.
- **Server Streaming RPC**: Method yang mengirimkan sebuah request ke server dan mendapatkan return value dari server berbentuk stream data yang artinya data dikirimkan secara sequential (tidak langsung semua, dipisah-pisah dan diurutkan). Client akan terus membaca data stream hingga sudah tidak ada data untuk dibaca. gRPC menjamin urutan data yang dikirimkan pada tiap-tiap RPC call.
- **Client Streaming RPC**: Method yang mengirimkan request atau message secara sequential dengan menggunakan stream yang sudah disediakan. Setelah selesai mengirim message, client akan menunggu server untuk membaca data dan mengembalikan response. Lagi-lagi gRPC menjamin urutan data yang dikirimkan pada tiap-tiap RPC call.
- **Bidirectional Streaming RPC**: Method yang keduanya (server dan client) sama-sama mengirimkan data berbentuk read-write stream secara sequential. Stream antara server dan client ini beroperasi secara independen, sehingga client dan sevrer bisa read dan write data tidak harus secara urut, atau bisa juga dikasih SOP seperti read message dahulu, lalu write message. Urutan dari message akan tetap diperhatikan oleh gRPC.

### Using the API
Dimulai dari service definition dalam sebuah `.proto` file, gRPC menyediakan compiler untuk protocol buffer yang akan men-generate client dan server code. Biasanya pengguna gRPC memanggil API ini di sisi client dan mengimplementasikannya di sisi server.
- Di sisi server, server mengimplementasikan method yang sudah dideklarasikan oleh service dan menjalankan gRPC server untuk menghandle call dari client. gRPC akan men-decode incoming request, mengeksekusi service method, dan meng-encode service response yang nantinya dikembalikan ke client.
- Di sisi client, client memiliki objek lokal dinamakan `stub` yang mengimplementasikan method yang sama dengan service. Client akan memanggil method dari objeck stub tadi, membungkus parameter-nya untuk keperluan call ke server dengan cara yang benar sesuai dengan protocol buffer message type, yang nantinya akan mudah dicari dan ditemukan oleh gRPC setelah client kirim call ke server dan menerima balasan

### Synchronous vs Asynchronous
- Dengan **Synchronous** RPC call, otomatis akan block sampai response dari server diterima.
- Dengan **Asynchronous**, tidak akan blocking.

## RPC Life Cycle

### Unary RPC
Unary RPC adalah tipe paling simpel dari jenis RPC Method.
- Client akan memanggil method dari objek lokal (stub), server kemudian diberitahu dambil dikasih data dari client bahwa RPC telah dipanggil, data-data itu dalah nama method, dan deadline
- Server bisa langsung kirim metadata dulu (yang biasanya memang harus dikirim sebelum kirim response), atau menunggu request message dari client dulu.
- Setelah server menerima request message dari client, server mulai mengeksekusi pekerjaan apapun yang diperlukan untuk mengirimkan response. Setelah sukses response akan dikirimkan bersamaan dengan detail status (code dan status message) dan metadata (optional)
- Jika reponse status OK, maka client akan mendapat response, yang akan mngakhiri proses RPC call.

### Server Streaming RPC
Method ini mirip dengan Unary, namun perbedaanya response yang dikirim dari serveradalah dalam bentuk stream message. Setelah semua reponse dari server sudah dikirim, message, server status, dan metadata ke client, dan client menerima, server akan mengakhiri proses RPC call di sisi server.

### Client Streaming RPC
Method ini mirip dengan Unary, namun perbedaanya request yang dikirim dari client adalah dalam bentuk stream message. Server mengirim response kembali ke client dalam bentuk satu data message langsung.

### Bidirectional Streaming RPC
- Dalam Method ini, call dimulai ketika client memamnggil method dan server menerima metadata, nama method, dan deadline dari client
- Server bisa memilih untuk nunggu client kirim message secara streamed atau langsung kirim initial metadata dulu ke client.
- Client dan server stream processing is application specific yang artinya independen satu sama lain. Sehingga client dan server bisa read dan write message dalam urutan seenaknya.
Contoh:
-  Server bisa menunggu client selesai kirim streamed messagenya sebelum nulis message buat responsenya.
-  Server bisa bermain `ping-pong` dengan client, yang artinya setiap ada request dari client, server langsung kirim balik response-nya, begitu seterusnya.

### Istilah-istilah

#### Deadline/Timeout
- Client bisa menentukan seberapa lama mereka harus menunggu untuk mendapatkan reponse dari server sebelum RPC call dihentikan dengan error `DEADLINE_EXCEEDED`. Di sisi server, server bisa melihat apakah sebuah RPC memiliki time out, atau berapa sisa waktu untuk menyelesaikan sebauh RPC call.

#### RPC Termination
- Dalam gRPC, client dan server bisa membuat definisi bagaimana sebuah call dianggap sukses, definisi ini bisa berbeda antara client dan server.

#### Membatalkan RPC Call
- Client maupun server keduanta bisa membatalkan RPC sewaktu-waktu. Pembatalan ini akan langsung menghentikan RPC sehingga process apapun yang sedang berjalan akan dihentikan.
- Perlu diperhatikan semua perubahan yang dilakukan sebelum pembatalan tidak akan di-rollback,

#### Metadata
- Metadata adalah bagian dari RPC call yang bisa disediakan oleh client maupun sever daalam bentuk pasangan key-value. Cara mengakses metadata ini akan berbeda di tiap-tiap bahasa pemrograman.

#### Channels
- Sebuah gRPC channel menyediakan koneksi ke gRPC server dengan host dan port tertentu. Channel berguna ketika membuat client stub. Di sini client dapat menentukan parameter channel guna memodifikasi perilaku default dari gRPC, seperti mengganti message compression.
- Sebuah channel memiliki state yaitu `connected` dan `idle`
- Bagaimana gRPC menghandle sebuah client juga berbeda-beda tiap bahasa pemrograman.

