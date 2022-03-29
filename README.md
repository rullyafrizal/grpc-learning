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

## Sekilas Tentang Protocol Buffer
## Core Concepts, Architecture, and Lifecycle
