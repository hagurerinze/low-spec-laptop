# Intel iGPU, Shared Memory, dan Batas Upgrade RAM

## Checkpoint eksperimen: Acer Aspire ES1-432 vs Dell Inspiron 3138

-   **Tanggal checkpoint:** 2026-10-20
-   **Status:** temuan benchmark sudah dicatat; batas upgrade RAM kedua
    laptop masih perlu diverifikasi.
-   **Tujuan:** mendokumentasikan mengapa Dell melaporkan memori grafis
    lebih besar, sementara Acer menang di benchmark, sekaligus
    memisahkan fakta dari hipotesis upgrade RAM.
-   **Ruang lingkup:** hanya perbandingan GPU, pelaporan video memory,
    hasil benchmark, dan kemungkinan upgrade RAM kedua laptop.

------------------------------------------------------------------------

## 1. Ringkasan utama

Hasil yang sudah diperoleh:

-   **Acer menang dalam pengujian `glmark2`: 566 vs 278**, sekitar 2,03
    kali skor Dell pada pengujian tersebut.
-   **Dell melaporkan angka `Video memory` lebih besar:** 1536 MB,
    dibanding Acer 900 MB.
-   Kedua GPU melaporkan `Unified memory: yes`. Artinya, keduanya
    menggunakan arsitektur memori terpadu/shared system memory, bukan
    VRAM khusus seperti kartu grafis diskret.
-   Angka `Video memory` dari Mesa bukan bukti bahwa seluruh kapasitas
    itu sedang digunakan GPU atau dicadangkan permanen.
-   RAM fisik Dell 4 GB, sedangkan Acer 2 GB. RAM Dell yang lebih besar
    mungkin berpengaruh pada nilai yang dilaporkan, tetapi data saat ini
    **belum membuktikan mekanisme pasti** yang menghasilkan 900 MB vs
    1536 MB.
-   Menambah RAM Acer mungkin meningkatkan kelancaran sistem dan
    mengurangi tekanan swap. Namun, **belum terbukti bahwa upgrade RAM
    akan meningkatkan angka `Video memory` Mesa**.
-   Batas upgrade maksimum kedua laptop masih perlu dipastikan sebelum
    membeli modul RAM.

## 2. Identitas perangkat

### Acer Aspire ES1-432

-   OS: Debian GNU/Linux 13 (trixie).
-   CPU: Intel Celeron N3350.
-   GPU: Intel HD Graphics 500 (Apollo Lake / APL 2).
-   PCI ID GPU: `8086:5a85`.
-   Kernel driver: `i915`.
-   Mesa: `25.0.7-2+deb13u1`.
-   RAM terpasang: 2 GB DDR3-1600, menurut `dmidecode`.
-   Resolusi layar yang tercatat sebelumnya: 1366×768, 60 Hz.
-   `glxinfo` melaporkan OpenGL core/compatibility 4.6 dan OpenGL ES
    3.2.

### Dell Inspiron 3138

-   CPU: Intel Celeron N2815.
-   GPU: Intel HD Graphics (Bay Trail / BYT).
-   PCI ID GPU: `8086:0f31`.
-   Nama PCI yang muncul:
    `Intel Corporation Atom Processor Z36xxx/Z37xxx Series Graphics & Display`.
-   Kernel driver: `i915`.
-   Mesa: `25.0.7-2+deb13u1`.
-   RAM terpasang: 4 GB DDR3L/SO-DIMM menurut data yang diperoleh.
-   Resolusi layar yang tercatat sebelumnya: 1366×768, 60 Hz.
-   `glxinfo` melaporkan OpenGL core/compatibility 4.2 dan OpenGL ES
    3.0.

**Catatan penamaan:** nama PCI Dell yang mengandung "Atom Processor
Z36xxx/Z37xxx" adalah label perangkat/platform, bukan nama CPU yang
menggantikan identifikasi CPU Celeron N2815. Nama Mesa `iris` yang
pernah terlihat pada informasi Acer adalah nama driver Mesa, bukan
berarti GPU tersebut Intel Iris Xe.

## 3. Hasil benchmark grafis

### Hasil akhir `glmark2`

Kedua hasil berasal dari pengujian `glmark2` versi 2023.01 dalam mode
windowed 800×600 dengan Mesa 25.0.7-2+deb13u1.

  -----------------------------------------------------------------------
  Parameter                        Acer ES1-432        Dell Inspiron 3138
  ------------------- ------------------------- -------------------------
  Renderer               Mesa Intel HD Graphics    Mesa Intel HD Graphics
                                    500 (APL 2)                     (BYT)

  Skor `glmark2`                        **566**                       278

  Perbandingan skor        **sekitar 2,03× skor   sekitar 49,1% skor Acer
                                         Dell** 

  Pemenang tes ini                     **Acer**                       ---
  -----------------------------------------------------------------------

Per-scene FPS yang dicatat:

  Scene                                           Acer               Dell
  --------------------------------- ------------------ ------------------
  build vbo=false                                  774                344
  build vbo=true                                   948                371
  texture nearest                                  806                353
  texture linear                                   777                353
  texture mipmap                                   803                362
  shading gouraud                                  646                323
  shading blinn-phong-inf                          642                324
  shading phong                                    637                314
  shading cel                                      636                304
  bump high-poly                                   548                237
  bump normals                                     886                372
  bump height                                      859                366
  effect2d kernel (hasil pertama)                  621                288
  effect2d kernel (hasil kedua)                    423                188
  pulsar                                           673                339
  desktop blur                                     323                156
  desktop shadow                                   485                217
  buffer map                                       261                140
  buffer subdata                                   298                178
  buffer interleave map                            312                171
  ideas                                            591                309
  jellyfish                                        352                197
  terrain                                           83                 36
  shadow                                           540                186
  refract                                           87                 49
  conditionals                        sekitar 588--597   sekitar 324--347
  function                            sekitar 587--592   sekitar 346--348
  loop                                sekitar 587--593   sekitar 324--349

Semua scene yang dicatat lebih tinggi pada Acer dalam run ini.

### Batas interpretasi benchmark

-   Hasil ini menunjukkan Acer lebih cepat pada rangkaian tes `glmark2`
    tersebut.
-   Skor tidak menjamin Acer akan dua kali lebih cepat pada semua game
    atau aplikasi.
-   Benchmark dapat dipengaruhi CPU, clock, suhu, daya, driver, beban
    desktop, dan kondisi pengujian.
-   `glxgears` dengan VSync menghasilkan sekitar 60 FPS pada kedua
    laptop, sehingga angka tersebut tidak berguna sebagai perbandingan
    performa GPU.
-   Dengan `vblank_mode=0`, `glxgears` menghasilkan sekitar 1557 FPS
    pada Dell dan 670--679 FPS pada Acer, tetapi `glxgears` tetap
    merupakan tes sederhana dan tidak menggantikan benchmark grafis yang
    lebih luas.
-   Ada kekeliruan pemetaan hasil pada diskusi awal. Pemetaan yang benar
    dan sudah diverifikasi lewat PCI ID serta renderer adalah **Acer =
    skor 566; Dell = skor 278**.

## 4. Data `glxinfo -B`

### Acer

``` text
Vendor: Intel (0x8086)
Device: Mesa Intel(R) HD Graphics 500 (APL 2) (0x5a85)
Version: 25.0.7
Accelerated: yes
Video memory: 900MB
Unified memory: yes
Preferred profile: core (0x1)
Max core profile version: 4.6
Max compat profile version: 4.6
Max GLES1 profile version: 1.1
Max GLES[23] profile version: 3.2
OpenGL vendor string: Intel
OpenGL renderer string: Mesa Intel(R) HD Graphics 500 (APL 2)
OpenGL core profile version string: 4.6 (Core Profile) Mesa 25.0.7-2+deb13u1
OpenGL version string: 4.6 (Compatibility Profile) Mesa 25.0.7-2+deb13u1
OpenGL ES profile version string: OpenGL ES 3.2 Mesa 25.0.7-2+deb13u1
```

### Dell

``` text
Vendor: Intel (0x8086)
Device: Mesa Intel(R) HD Graphics (BYT) (0xf31)
Version: 25.0.7
Accelerated: yes
Video memory: 1536MB
Unified memory: yes
Preferred profile: core (0x1)
Max core profile version: 4.2
Max compat profile version: 4.2
Max GLES1 profile version: 1.1
Max GLES[23] profile version: 3.0
OpenGL vendor string: Intel
OpenGL renderer string: Mesa Intel(R) HD Graphics (BYT)
OpenGL core profile version string: 4.2 (Core Profile) Mesa 25.0.7-2+deb13u1
OpenGL version string: 4.2 (Compatibility Profile) Mesa 25.0.7-2+deb13u1
OpenGL ES profile version string: OpenGL ES 3.0 Mesa 25.0.7-2+deb13u1
```

Kedua output menyatakan `Accelerated: yes` dan `Unified memory: yes`.

## 5. RAM dan tekanan memori saat output dikumpulkan

### Acer

``` text
Mem: 1.8Gi total, 1.4Gi used, 95Mi free, 124Mi shared,
     569Mi buff/cache, 386Mi available
Swap: 5.3Gi total, 900Mi used, 4.4Gi free
```

### Dell

``` text
Mem: 3.7Gi total, 836Mi used, 686Mi free, 60Mi shared,
     2.5Gi buff/cache, 2.9Gi available
Swap: 4.0Gi total, 12Ki used, 4.0Gi free
```

Angka ini adalah snapshot saat perintah dijalankan, bukan pengukuran
yang dijamin diambil pada kondisi idle atau beban yang identik. Namun,
snapshot Acer menunjukkan tekanan memori lebih tinggi dan penggunaan
swap lebih banyak. Ini bisa memengaruhi kelancaran pemakaian
sehari-hari.

## 6. Apa arti `Video memory` 900 MB vs 1536 MB?

Xfce menampilkan:

-   Acer: `Mesa Intel HD Graphics 500 (900.0 MiB)`.
-   Dell: `Mesa Intel HD Graphics (1.5 GiB)`.

Angka itu cocok dengan nilai yang ditampilkan `glxinfo`:

-   Acer: `Video memory: 900MB`.
-   Dell: `Video memory: 1536MB`.

Dengan demikian, angka Xfce berasal dari informasi yang sama atau
sejalan dengan laporan Mesa.

**Jangan menafsirkan angka tersebut sebagai VRAM khusus atau memori yang
pasti dicadangkan permanen.** Keduanya adalah GPU terintegrasi dengan
`Unified memory: yes`, sehingga GPU menggunakan RAM sistem bersama.
Nilai yang dilaporkan Mesa tidak sama dengan jumlah RAM yang sedang
dipakai GPU dan tidak otomatis merupakan batas mutlak alokasi GPU.

RAM Dell yang lebih besar mungkin ikut memengaruhi nilai pelaporan,
tetapi output yang tersedia belum cukup untuk memastikan algoritme atau
batas spesifik yang menghasilkan angka 900 MB dan 1536 MB. Kita juga
belum membuktikan bahwa menambah RAM Acer akan menaikkan angka
`Video memory`.

## 7. Hasil `dmidecode -t memory`

### Acer

``` text
SMBIOS 3.0.0 present.

Physical Memory Array
    Location: System Board Or Motherboard
    Use: System Memory
    Error Correction Type: None
    Maximum Capacity: 32 GB
    Number Of Devices: 2

Memory Device
    Total Width: 8 bits
    Data Width: 8 bits
    Size: 2 GB
    Form Factor: DIMM
    Locator: ChannelA-DIMM1
    Bank Locator: BANK 1
    Type: DDR3
    Type Detail: Synchronous
    Speed: 1600 MT/s
    Manufacturer: Samsung
    Serial Number: 00000000
    Configured Memory Speed: 1600 MT/s
```

Interpretasi berhati-hati:

-   Firmware melaporkan satu entri memori 2 GB DDR3-1600 dan
    `Number Of Devices: 2`.
-   `Maximum Capacity: 32 GB` adalah nilai SMBIOS; **belum terverifikasi
    sebagai kapasitas upgrade yang benar-benar didukung** oleh
    motherboard/BIOS dan CPU.
-   `ChannelA-DIMM1` dan `Form Factor: DIMM` tidak membuktikan RAM pasti
    tersolder atau pasti berupa SO-DIMM yang bisa dilepas.
-   `lshw` kemudian menampilkan satu bank 2 GiB pada `ChannelA-DIMM1`,
    tetapi hasil itu juga belum membuktikan topologi fisik secara
    definitif.
-   BIOS yang dilaporkan: Insyde Corp. V1.08, tanggal 2016-10-21.

### Dell

``` text
SMBIOS 2.7 present.

Memory Module Information
    Socket Designation: DIMM0
    Current Speed: Unknown
    Type: None
    Installed Size: 4096 MB (Single-bank Connection)
    Enabled Size: 4096 MB (Single-bank Connection)

Physical Memory Array
    Location: System Board Or Motherboard
    Use: System Memory
    Error Correction Type: None
    Maximum Capacity: 4 GB
    Number Of Devices: 2

Memory Device
    Total Width: 64 bits
    Data Width: 64 bits
    Size: 4 GB
    Form Factor: SODIMM
    Locator: DIMM0
    Bank Locator: BANK 0
    Type: DDR3
    Type Detail: Synchronous
    Speed: 1066 MT/s
    Manufacturer: Micron
    Part Number: 8KTF51264HZ-1G6E1
    Configured Memory Speed: 1066 MT/s
```

Interpretasi berhati-hati:

-   Firmware melaporkan modul 4 GB dengan form factor `SODIMM`, sehingga
    data ini mendukung kesimpulan bahwa RAM terpasang adalah modul,
    bukan seluruhnya tersolder.
-   `Maximum Capacity: 4 GB` adalah batas yang dilaporkan SMBIOS dan
    sejalan dengan batas resmi yang ditemukan, tetapi tidak dengan
    sendirinya membuktikan apakah modul 8 GB mustahil bekerja.
-   Perbedaan antara `Number Of Devices: 2` dan satu entri memori 4 GB
    menunjukkan bahwa data firmware tidak boleh dibaca secara terlalu
    literal untuk menyimpulkan jumlah slot fisik.

## 8. Batas upgrade RAM: status bukti

### Acer Aspire ES1-432

Informasi yang sudah dibahas:

-   RAM sekarang: 2 GB DDR3-1600.
-   CPU: Celeron N3350.
-   Dokumentasi servis yang ditemukan sebelumnya menyebut konfigurasi
    berbeda: 2 GB onboard + SO-DIMM hingga total 6 GB, atau varian tanpa
    onboard RAM dengan SO-DIMM hingga 8 GB.
-   Klaim komunitas mengenai konfigurasi 10 GB/Flex Mode belum
    terverifikasi untuk unit ini.
-   Celeron N3350 tercatat mendukung maksimum memori sistem 8 GB pada
    spesifikasi Intel. Batas CPU ini bukan jaminan bahwa setiap
    motherboard/BIOS ES1-432 mendukung 8 GB.
-   Belum diketahui dengan pasti apakah RAM 2 GB di unit ini onboard
    atau merupakan modul yang bisa dilepas.
-   `lshw` tersedia setelah dipasang, tetapi hasil yang dikirim belum
    menyelesaikan pertanyaan topologi RAM.

**Status:** jangan menyimpulkan 32 GB dapat dipasang hanya dari SMBIOS.
Jangan membeli modul sebelum konfigurasi fisik dan varian motherboard
dipastikan.

### Dell Inspiron 3138

Informasi yang sudah dibahas:

-   RAM sekarang: 4 GB, dilaporkan sebagai SO-DIMM.
-   Dokumentasi Dell yang ditemukan sebelumnya menyebut satu konektor
    SO-DIMM dan maksimum resmi 4 GB.
-   CPU Celeron N2815 memiliki batas memori sistem yang dilaporkan Intel
    hingga 8 GB, tetapi spesifikasi CPU tidak membuktikan bahwa
    BIOS/motherboard 3138 menerima modul 8 GB.
-   Ada laporan komunitas tentang Inspiron 3137 yang berhasil memakai 8
    GB, tetapi 3137 adalah model berbeda. Bukti itu tidak boleh dipakai
    sebagai konfirmasi kompatibilitas 3138.

**Status:** modul RAM Dell dapat dilepas, tetapi keberhasilan upgrade ke
8 GB pada Inspiron 3138 dengan Celeron N2815 belum terverifikasi secara
spesifik.

### Catatan Flex Mode

Flex Mode mengatur pemakaian kanal memori saat kapasitas memori
antarkanal tidak seimbang. Flex Mode bukan mekanisme yang dengan
sendirinya menaikkan batas maksimum RAM yang didukung CPU, motherboard,
atau BIOS. Klaim konfigurasi 10 GB harus dibuktikan dengan identifikasi
varian yang tepat dan laporan pengujian yang dapat dipercaya.

## 9. Rencana eksperimen setelah upgrade Acer

**Status: DRAFT --- belum dilakukan.**

Setelah kompatibilitas RAM Acer dipastikan dan RAM benar-benar
di-upgrade, ulangi pemeriksaan berikut.

### A. Catat kondisi RAM

``` bash
free -h
```

Catat `total`, `available`, penggunaan swap, dan kondisi desktop saat
pengukuran.

### B. Catat laporan memori grafis

``` bash
glxinfo -B
```

Catat:

-   `Device`
-   `Version`
-   `Accelerated`
-   `Video memory`
-   `Unified memory`
-   versi OpenGL dan OpenGL ES maksimum.

Pertanyaan yang diuji: apakah angka `Video memory` berubah setelah
upgrade? Hasil tidak boleh diasumsikan terlebih dahulu.

### C. Ulangi benchmark

Jalankan `glmark2` dengan kondisi yang sedekat mungkin dengan pengujian
awal:

-   resolusi/window size sama (800×600);
-   versi benchmark dan Mesa dicatat;
-   desktop dan beban latar belakang dibuat serupa;
-   catat suhu/daya bila tersedia;
-   simpan skor akhir dan per-scene FPS.

Pertanyaan yang diuji: apakah performa benchmark berubah secara
konsisten? RAM lebih besar dapat membantu ketika sistem mengalami
tekanan memori, tetapi tidak otomatis meningkatkan kemampuan komputasi
GPU.

### D. Bandingkan sebelum dan sesudah

Bandingkan setidaknya tiga hal secara terpisah:

1.  **Kapasitas RAM sistem dan `available`** --- apakah tekanan memori
    dan swap berkurang?
2.  **`Video memory` dari Mesa** --- apakah nilai yang dilaporkan
    berubah?
3.  **Skor `glmark2`** --- apakah performa grafis terukur berubah?

Perubahan pada satu kategori tidak boleh dianggap membuktikan perubahan
pada kategori lainnya.

## 10. Kesimpulan saat checkpoint dibuat

1.  **Pemenang benchmark grafis yang telah dilakukan: Acer**, dengan
    `glmark2` 566 vs Dell 278.
2.  **Dell melaporkan memori video lebih besar:** 1536 MB vs Acer 900
    MB. Ini adalah nilai laporan Mesa pada GPU dengan unified memory,
    bukan bukti bahwa Dell memiliki GPU lebih kencang.
3.  **Shared RAM adalah konteks yang relevan**, tetapi belum terbukti
    sebagai satu-satunya penyebab angka yang berbeda.
4.  **Upgrade RAM Acer berpotensi membantu kelancaran sistem**,
    mengingat snapshot sebelumnya menunjukkan RAM tersedia rendah dan
    swap sudah terpakai. Efeknya pada angka video memory Mesa belum
    diketahui.
5.  **Batas RAM maksimum kedua laptop belum sepenuhnya dipastikan untuk
    unit spesifik ini.** Data SMBIOS, batas CPU, dokumentasi model, dan
    laporan komunitas adalah jenis bukti berbeda dan tidak boleh
    dicampur.
6.  Jangan membeli modul 8 GB untuk salah satu laptop hanya berdasarkan
    `Maximum Capacity` dari `dmidecode` atau keberhasilan upgrade model
    tetangga.
7.  Langkah lanjutan yang masih diperlukan: identifikasi konfigurasi
    fisik RAM/motherboard Acer dan cari bukti kompatibilitas yang secara
    spesifik berlaku untuk Dell Inspiron 3138 dengan Celeron N2815.

------------------------------------------------------------------------

## Referensi yang perlu diverifikasi ulang sebelum pembelian

Referensi yang pernah dibahas dalam sesi ini:

-   Acer Aspire ES1-432 Service Guide:
    https://www.manualzz.com/doc/33583656/acer-aspire-es1-432-laptop-service-guide
-   Dell Inspiron 11-3138 Reference Guide (PDF):
    https://downloads.dell.com/manuals/all-products/esuprt_laptop/esuprt_inspiron_laptop/inspiron-11-3138_reference%20guide_en-us.pdf
-   Spesifikasi prosesor Intel Celeron N3350 dan N2815: periksa halaman
    spesifikasi resmi Intel untuk masing-masing SKU.
-   Laporan komunitas tentang RAM 10 GB pada ES1-432 dan 8 GB pada
    Inspiron 3137: perlakukan sebagai petunjuk, bukan jaminan
    kompatibilitas.

**Catatan kontribusi:** ini adalah checkpoint eksperimen dan rangkuman
hasil pengujian yang dilakukan pengguna. Klaim yang belum diuji ditandai
sebagai belum terverifikasi agar tidak berubah menjadi fakta hanya
karena diulang dalam catatan.
