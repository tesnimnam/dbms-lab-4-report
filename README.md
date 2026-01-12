# Deney Sonu Teslimatı

Sistem Programlama ve Veri Yapıları bakış açısıyla veri tabanlarındaki performansı öne çıkaran hususlar nelerdir?

Aşağıda kutucuk (checkbox) ile gösterilen maddelerden en az birini seçtiğiniz açık kaynak kodlu bir VT kaynak kodları üzerinde göstererek açıklayınız. Açıklama bölümüne kısaca metninizi yazıp, kod üzerinde gösterim videonuzun linkini en altta belirtilen kutucuğa yerleştiriniz.

- [X]  Seçtiğiniz konu/konuları bu şekilde işaretleyiniz. **!**
    
---

# 1. Sistem Perspektifi (Operating System, Disk, Input/Output)

### Disk Erişimi

- [X]  **Blok bazlı disk erişimi** → block_id + offset
- [ ]  Rastgele erişim

### VT için Page (Sayfa) Anlamı

- [X]  VT hangisini kullanır? **Satır/ Sayfa** okuması

---

### Buffer Pool

- [X]  Veritabanları, Sık kullanılan sayfaları bellekte (RAM) kopyalar mı (caching) ?

- [ ]  LRU / CLOCK gibi algoritmaları
- [ ]  Diske yapılan I/O nasıl minimize ederler?

# 2. Veri Yapıları Perspektifi

- [X]  B+ Tree Veri Yapıları VT' lerde nasıl kullanılır?
- [ ]  VT' lerde hangi veri yapıları hangi amaçlarla kullanılır?
- [ ]  Clustered vs Non-Clustered Index Kavramı
- [ ]  InnoDB satırı diskte nasıl durur?
- [ ]  LSM-tree (LevelDB, RocksDB) farkı
- [ ]  PostgreSQL heap + index ayrımı

DB diske yazarken:

- [ ]  WAL (Write Ahead Log) İlkesi
- [ ]  Log disk (fsync vs write) sistem çağrıları farkı

---

# Özet Tablo

| Kavram      | Bellek          | Disk / DB      |
| ----------- | --------------- | -------------- |
| Adresleme   | Pointer         | Page + Offset  |
| Hız         | O(1)            | Page IO        |
| PK          | Yok             | Index anahtarı |
| Veri yapısı | Array / Pointer | B+Tree         |
| Cache       | CPU cache       | Buffer Pool    |

---

# Video [Linki]([https://www.youtube.com/watch?v=Nw1OvCtKPII&t=2635s](https://youtu.be/pA4OUjKG96A) 
Ekran kaydı. 2-3 dk. açık kaynak V.T. kodu üzerinde konunun gösterimi. Video kendini tanıtma ile başlamalıdır (Numara, İsim, Soyisim, Teknik İlgi Alanları). 

---

# Açıklama (Ort. 600 kelime)
Disk Erişimi, Buffer Yönetimi ve B+ Tree Yapılarının PostgreSQL Üzerinden İncelenmesi

Veritabanı yönetim sistemlerinde (VTYS), performansı doğrudan etkileyen en önemli faktörlerden biri disk erişimidir. Disk, RAM’e kıyasla çok daha yavaş olduğu için modern veritabanları disk erişimini en aza indirecek şekilde tasarlanır. Bu çalışmada PostgreSQL açık kaynaklı veritabanı sistemi üzerinden; blok bazlı disk erişimi, buffer (önbellek) yönetimi ve B+ Tree veri yapılarının kullanımı incelenmiştir.

## Blok Bazlı Disk Erişimi (block_id + offset)

PostgreSQL, verileri diskte bloklar (pages) halinde saklar. Varsayılan olarak bir blok boyutu 8192 byte (8 KB)’dır. Bu değer kaynak kodda BLCKSZ sabiti ile tanımlanır. Disk üzerinde bir veriye erişirken, tek tek baytlar yerine bloklar okunur. Bu yaklaşım, disk I/O maliyetini azaltır ve erişimi daha verimli hale getirir.
Blok bazlı erişimde iki temel kavram vardır:

-block_id: Okunacak bloğun disk üzerindeki sıra numarası

-offset: Bloğun içindeki veri konumu

Yani bir satıra erişmek için önce ilgili bloğa gidilir, ardından bloğun içindeki offset yardımıyla istenen veri bulunur. PostgreSQL’in disk erişim mekanizması işletim sistemi (OS) ile birlikte çalışır ve bu yapı OS seviyesindeki dosya sistemleriyle uyumludur.

## Buffer Pool ve Caching Mekanizması

Disk erişimi pahalı olduğu için PostgreSQL sık kullanılan verileri bellekte (RAM) tutar. Bu yapı buffer pool (shared buffers) olarak adlandırılır. Bir sorgu çalıştırıldığında PostgreSQL önce verinin buffer pool içinde olup olmadığını kontrol eder. Eğer veri RAM’de bulunuyorsa bu duruma cache hit, bulunmuyorsa cache miss denir.

Cache miss durumunda ilgili blok diskten okunur ve buffer pool’a kopyalanır. Sonraki erişimlerde disk yerine RAM kullanılır. Bu mekanizma PostgreSQL kaynak kodunda ReadBuffer, ReadBufferExtended ve ReadBuffer_common gibi fonksiyonlarla yönetilir. Bu fonksiyonlar, bir bloğun diskten okunması, buffer’a yerleştirilmesi ve pinlenmesi (kullanımda olduğunun işaretlenmesi) işlemlerini gerçekleştirir.

Ayrıca PostgreSQL, prefetch mekanizması ile henüz ihtiyaç duyulmadan bazı blokları önceden belleğe alabilir. Bu sayede ardışık okuma işlemlerinde performans artırılır.

## Disk I/O Kavramı

Disk I/O (Input / Output), verinin diskten okunması veya diske yazılması işlemlerini ifade eder. Disk I/O, RAM erişimine göre çok daha yavaş olduğu için veritabanı sistemleri bu işlemleri minimize etmeye çalışır. Buffer pool, caching ve prefetch gibi mekanizmalar bu amaca hizmet eder.

PostgreSQL’de buffer yönetimi sayesinde her sorgu için doğrudan disk erişimi yapılmaz. Bunun yerine, mümkün olduğunca RAM üzerinden işlem yapılır. Bu yaklaşım, özellikle büyük veri setleriyle çalışan sistemlerde ciddi performans kazanımı sağlar.

## B+ Tree Veri Yapılarının Kullanımı

PostgreSQL’de varsayılan indeks türü B+ Tree’dir (okunuşu: B plus tree). B+ Tree, dengeli bir ağaç yapısıdır ve veritabanlarında arama, sıralama ve aralık sorguları için son derece uygundur.

B+ Tree’nin temel özellikleri şunlardır:

-Veriler sadece yaprak düğümlerde (leaf nodes) tutulur

-İç düğümler yalnızca yönlendirme bilgisi içerir

-Yaprak düğümler birbirine bağlıdır (linked list)

Bu yapı sayesinde PostgreSQL, WHERE, ORDER BY, BETWEEN gibi sorguları çok hızlı bir şekilde gerçekleştirebilir. Kaynak kodda B+ Tree implementasyonu src/backend/access/nbtree dizini altında yer alır. Özellikle nbtree.c ve nbtsearch.c dosyaları, indeks arama ve gezinme işlemlerinin nasıl yapıldığını gösterir.

## VT Üzerinde Gösterilen Kaynak Kodları

Bu çalışmada PostgreSQL tercih edilmiştir çünkü PostgreSQL
tamamen açık kaynaklı bir veritabanı yönetim sistemidir ve
disk erişimi, sayfa yönetimi, buffer cache ve B+ Tree indeksleme
gibi temel mekanizmaları kaynak kodu seviyesinde incelemeye
olanak sağlamaktadır [Linki](https://github.com/postgres/postgres.git) .


