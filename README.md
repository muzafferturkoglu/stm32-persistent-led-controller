# STM32 Persistent LED Controller

STM32F103 üzerinde çalışan, LED'in yanıp sönme sayısını butonla değiştiren ve seçilen değeri dahili Flash bellekte saklayan bir gömülü sistem uygulaması. Zamanlama, ana döngüde gecikme saymak yerine TIM2 kesmesi üzerinden yürütülür.

## Özellikler

- PC13 üzerinden LED kontrolü.
- PA0 butonuyla **4 → 5 → 6 → 7 → 4** şeklinde değişen yanıp sönme sayısı.
- Güç kesilse de korunan Flash bellek ayarı.
- Açılışta butonu en az üç saniye basılı tutup bırakarak varsayılan değere dönüş.
- Bir saniyelik TIM2 kesmeleriyle LED durum değişimi ve diziler arasında bekleme.
- Buton girişinde yazılımsal debounce kontrolü.

## Donanım ve bağlantılar

| Bileşen | Bağlantı / yapılandırma |
| --- | --- |
| Mikrodenetleyici | STM32F103C8, Cortex-M3 |
| LED | PC13, aktif düşük seviye varsayımı |
| Buton | PA0 ile GND arasına; girişte dahili pull-up |
| Programlayıcı | SWD bağlantılı ST-LINK veya uyumlu programlayıcı |
| Sistem saati | Dahili HSI, 8 MHz |
| Zamanlayıcı | TIM2, prescaler 7999, period 999 |

## Çalışma akışı

1. Başlangıçta `0x0800FC00` adresindeki ayar okunur. Kayıt 4-7 aralığında değilse 4 değeri yazılır.
2. TIM2 kesmesi her saniye LED çıkışını değiştirir. Açma ve kapama birlikte bir yanıp sönme olarak sayılır.
3. Seçilen sayı tamamlandığında LED kapalı tutulur ve beş zamanlayıcı adımı beklenir. Sonraki kesmede yeni dizinin ilk durum değişimi gerçekleşir.
4. Butona basıldığında sayı artırılır, Flash'a kaydedilir ve dizi yeniden başlatılır.

## Derleme ve yükleme

Gereksinimler: **GNU Arm Embedded araç zinciri** (`arm-none-eabi-gcc`), **GNU Make** ve bir STM32 programlama aracı. Linker betiğindeki `READONLY` tanımları için GCC 11 veya üzeri gerekir.

```bash
git clone https://github.com/muzafferturkoglu/stm32-persistent-led-controller.git
cd stm32-persistent-led-controller
make
```

Araç zinciri PATH üzerinde değilse `make GCC_PATH=/arac/zinciri/bin` kullanın. `build/` altında üretilen `.elf` veya `.hex` dosyasını programlayıcıyla yükleyin. `.bin` kullanıldığında başlangıç adresi `0x08000000`'dır.

Pin ve saat ayarlarını incelemek için kök klasördeki `.ioc` dosyası STM32CubeMX ile açılabilir.

## Dosyalar

| Konum | İşlev |
| --- | --- |
| `Core/Src/main.c` | Buton, Flash okuma/yazma ve LED durum mantığı |
| `Core/Src/stm32f1xx_it.c` | Kesme işleyicileri |
| `Core/Src/stm32f1xx_hal_msp.c` | Çevre birimi altyapısı |
| `Drivers/` | STM32 HAL ve CMSIS bileşenleri |
| `Makefile` | Derleme tanımları |
| `STM32F103XX_FLASH.ld` | Bellek yerleşimi |

## Uygulama notları

Ayarlar için son 1 KB Flash sayfası kullanılır; mevcut linker betiği uygulamaya 64 KB'nin tamamını ayırır. Kod büyütülürken ayar sayfasının programla çakışmaması için bellek yerleşimi düzenlenmelidir. Mevcut sürüm her ayar değişiminde Flash sayfasını silip yeniden yazar.

**Geliştirici:** [Muzaffer Türkoğlu](https://github.com/muzafferturkoglu)
