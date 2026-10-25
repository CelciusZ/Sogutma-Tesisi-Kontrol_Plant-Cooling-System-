# Soğutma Tesisi: C# Arayüzlü Sıcaklık Kontrolü

[English](README.md)

Bir soğutma tesisinin odaları için sıcaklık kontrol sistemi. Dört PIC16F877A mikrodenetleyici (bir master, üç slave) RS-485 hattı üzerinden haberleşir. C# masaüstü arayüzünden her oda için istenen sıcaklık girilir ve ölçülen değerler izlenir.

![Arayüz](GUI.PNG)

## Genel bakış

| | |
|---|---|
| Mikrodenetleyiciler | 4 × PIC16F877A (1 master, 3 slave) |
| Sensör | Her slave'de LM35 (−55 °C ile +150 °C arası) |
| Gösterge | Her slave'de BCD 7 segment display (sıcaklık ve CRC) |
| MCU ↔ MCU haberleşmesi | RS-485 half duplex, MAX487 |
| Arayüz ↔ master haberleşmesi | Full duplex, paralel bağlı iki MAX487 |
| Gömülü yazılım | C, CCS C derleyicisi |
| Arayüz | C# WinForms (.NET Framework) |
| Simülasyon | Proteus, sanal COM portlar için VSPE |

## Nasıl çalışır

### Paket yapısı

Her mesaj 6 byte'lık bir pakettir:

| Byte | Alan |
|---|---|
| 1 | Gönderici adresi |
| 2 | Alıcı adresi |
| 3 | ACK |
| 4 | Veri uzunluğu |
| 5 | Veri |
| 6 | CRC |

Alıcı veriyi kullanmadan önce CRC'yi kontrol eder. Bozuk paket işlenmez, atılır.

### Master

- Her odanın istenen sıcaklığını arayüzden alır.
- Gelen byte'ları RDA (UART alma) kesmesinde işler.
- `make_packet` ile slave'ler için paket oluşturup gönderir.
- Slave paketi onayladığında (ACK) LED yakar.

### Slave'ler

- Oda sıcaklığını LM35'ten okur.
- Master'dan gelen hedef sıcaklıkla karşılaştırıp soğutucuyu açar veya kapatır.
- Sıcaklığı ve CRC'yi BCD display'de gösterir.
- Ölçülen sıcaklığı `send_gui` ile arayüze gönderir.
- Paket kontrolden geçmezse hata LED'ini yakar.

### Arayüz

- **COM port ayarları**: port, baud rate, parity ve data bit.
- **Transmitter**: hedef sıcaklıkları ve ACK'leri gönderir.
- **Receiver**: gelen ham veriyi gösterir (test için).
- **Sistem ekranı**: oda sıcaklıkları ve soğutucuların durumu.

## Repo yapısı

```
Circuit/Devre Şeması.pdsprj          Proteus şeması ve simülasyonu
Embedded Code Master and Slave/code/ CCS C kaynak kodları (CRC_MASTER.c, CRC_SLAVE_*.c) ve .hex dosyaları
Form1.cs, Form1.Designer.cs          C# arayüz
*.sln, GUI.csproj                    Visual Studio çözümü
```

`CRC_SLAVE_3` repoda sadece `.hex` dosyası olarak var.

## Çalıştırma

1. `Circuit/Devre Şeması.pdsprj` dosyasını Proteus'ta açın. `.hex` dosyaları hazır. Yeniden derlemek için `.c` dosyalarını CCS C ile derleyin.
2. VSPE ile bir çift sanal COM port oluşturun ve birini Proteus şemasındaki sanal seri porta bağlayın.
3. `.sln` dosyasını Visual Studio'da açıp arayüzü çalıştırın.
4. Arayüzde diğer COM portu seçin, 9600 baud, parity yok, 8 data bit, 1 stop bit seçin (gömülü yazılımdaki ayarlar) ve bağlanın.
5. Bir oda için hedef sıcaklık girip gönderin. Slave'in göstergesi ve arayüzdeki sistem ekranı güncellenmelidir.

## Lisans

MIT, bkz. [LICENSE.txt](LICENSE.txt).
