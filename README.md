# Türkiye İl, İlçe ve Mahalle Veritabanı

Bu veri seti, Türkiye'deki tüm iller, ilçeler ve mahalleleri PostgreSQL formatında içermektedir. Veritabanı, il, ilçe ve mahalleler arasındaki ilişkileri destekleyecek şekilde yapılandırılmıştır.

## Tablo Yapısı

Veri seti, dört ana tablo içermektedir: `city`, `district`, `neighbourhood` ve `country`. Her bir tablonun yapısı ve içerdiği bilgiler aşağıda açıklanmıştır.

### 1. City (Şehirler)
`city` tablosu, Türkiye'deki iller hakkında temel bilgileri içerir.

- **id**: (INTEGER) Şehrin benzersiz kimliği.
- **name**: (TEXT) Şehrin adı (örneğin, "İstanbul").
- **plate_number**: (INTEGER) Şehrin plaka numarası (örneğin, "34").
- **phone_code**: (INTEGER) Şehrin telefon kodu (örneğin, "212").
- **row_number**: (INTEGER) Sıra numarası (veri sıralaması için kullanılabilir).

### 2. District (İlçeler)
`district` tablosu, illere bağlı ilçeleri içerir. İlçeler, `city` tablosuna bir yabancı anahtar ile bağlanmıştır.

- **id**: (INTEGER) İlçenin benzersiz kimliği.
- **city_id**: (INTEGER) İlçenin bağlı olduğu şehrin kimliği (foreign key).
- **name**: (TEXT) İlçenin adı (örneğin, "Kadıköy").
- **city_name**: (TEXT) İlçenin bağlı olduğu şehrin adı (bilgilendirme için).

### 3. Neighbourhood (Mahalleler)
`neighbourhood` tablosu, ilçelere bağlı mahalleleri içerir. Mahalleler, `district` tablosuna bir yabancı anahtar ile bağlanmıştır.

- **id**: (INTEGER) Mahallenin benzersiz kimliği.
- **name**: (TEXT) Mahallenin adı (örneğin, "Bostancı").
- **area_name**: (TEXT) Semt Adı (örneğin, "Kadıköy").
- **postal_code**: (TEXT) Mahallenin posta kodu.
- **district_id**: (INTEGER) Mahallenin bağlı olduğu ilçenin kimliği (foreign key).

### 4. Country (Ülkeler)
Ek olarak, `country` tablosu, ülke bilgilerini içerir. Bu tablo, uluslararası projelerde kullanılabilir.

- **id**: (INTEGER) Ülkenin benzersiz kimliği.
- **iso_code2**: (TEXT) ISO 2 haneli ülke kodu (örneğin, "TR").
- **iso_code3**: (TEXT) ISO 3 haneli ülke kodu (örneğin, "TUR").
- **name**: (TEXT) Ülkenin adı (örneğin, "Türkiye").
- **phone_code**: (TEXT) Ülkenin telefon kodu (örneğin, "+90").

## Prisma Şeması

Eğer Prisma kullanarak bu veritabanını oluşturmak isterseniz, aşağıdaki Prisma şemasını kullanabilirsiniz:

```prisma
model District {
  id       Int    @id
  cityId   Int    @map("city_id") // Şehir ID
  name     String @map("name") // İlçe Adı
  cityName String @map("city_name") // Şehir Adı

  city           City            @relation(fields: [cityId], references: [id])
  neighbourhoods Neighbourhood[] // Birden-çok ilişki

  @@index([cityId], name: "FK_District_City")
  @@map("district")
}

model City {
  id          Int    @id
  name        String @map("name") // Şehir Adı
  plateNumber Int    @map("plate_number") // Plaka Numarası
  phoneCode   Int    @map("phone_code") // Telefon Kodu
  rowNumber   Int    @map("row_number") // Sıra Numarası

  districts District[] // Birden-çok ilişki

  @@map("city")
}

model Neighbourhood {
  id         Int    @id
  name       String @map("name") // Mahalle Adı
  areaName   String @map("area_name") // Semt Adı
  postalCode String @map("postal_code") // Posta Kodu
  districtId Int    @map("district_id") // İlçe ID

  district District @relation(fields: [districtId], references: [id])

  @@index([districtId], name: "FK_Neighbourhood_District")
  @@map("neighbourhood")
}

model Country {
  id        Int    @id
  isoCode2  String @map("iso_code2") // ISO 2 Kodu
  isoCode3  String @map("iso_code3") // ISO 3 Kodu
  name      String @map("name") // Ülke Adı
  phoneCode String @map("phone_code") // Telefon Kodu

  @@map("country")
}
```
Prisma Kullanım Talimatları

1.	Yukarıdaki Prisma şemasını prisma/schema.prisma dosyanıza ekleyin.
2.	Prisma ile veritabanı tablolarını oluşturmak için aşağıdaki komutları çalıştırın: 
```bash
npx prisma migrate dev --name init
```
3.	NOT: Veritabanı tabloları oluşturulduktan sonra, turkiye_konum_verileri.sql dosyasındaki INSERT komutları için olan kısmı çalıştırmadan önce CREATE TABLE bölümlerini silebilirsiniz. Ardından, veri eklemek için SQL dosyasını çalıştırabilirsiniz.

SQL Dosyasını Çalıştırma
1.	Bu depoyu klonlayın veya turkiye_konum_verileri.sql dosyasını indirin.
2.	PostgreSQL veritabanınıza dosyayı yüklemek için aşağıdaki komutu kullanın:
```bash
psql -U kullanıcı_adı -d veritabani_adi -f path/to/turkiye_konum_verileri.sql
```