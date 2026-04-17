# Meware PDKS V16.20200313 - Güvenlik Açıkları ( Bilgi İfşası + IDOR + Rate Limit (DoS) )

##  Özet
Meware PDKS (Personel Devam Kontrol Sistemi) V16.20200313 versiyonunda **3 adet  güvenlik açığı** tespit edilmiştir.

| # | Zafiyet | CWE | CVSS |
|---|---------|-----|------|
| 1 | Bilgi İfşası - Kullanıcı ID'lerinin Sızdırılması | CWE-200 | 4.3 (Medium) 
| 2 | IDOR - Yetkisiz Fazla Mesai Talebi Oluşturma | CWE-639 | 8.2 (High) 
| 3 | Rate Limit Eksikliği - Sınırsız Talep Oluşturma | CWE-770 | 7.5 (High) 


##  Etkilenen Sürüm
- Meware PDKS **V16.20200313** ve önceki tüm sürümler

##  ZAFİYET 1: Bilgi İfşası (Information Disclosure)
### Açıklama
Sistem, yetki kontrolü yapmadan tüm kullanıcıların ID, ad soyad ve kullanıcı adı bilgilerini döndürmektedir.

### (Proof of Concept)
http
GET /api/Dynamic?Name=tokenid%3D...%26siciller%3D607%26point%3Dtalep%26islemtipi%3Di HTTP/2

Host: meyerapi.tcddteknik.com.tr


<img width="786" height="329" alt="kullanıcı idleri" src="https://github.com/user-attachments/assets/c3bf34ab-7120-4a5d-b1e0-fbc0475f870c" />



Sistem, yetki kontrolü yapmadan tüm kullanıcıların ID, ad soyad ve kullanıcı adı bilgilerini döndürmektedir.


##  ZAFİYET 2: Rate Limit Eksikliği (DoS) - Sınırsız Talep Oluşturma
### Açıklama
Sistem, bir kullanıcının belirli bir zaman diliminde gönderebileceği istek sayısını sınırlamamaktadır. Bir kullanıcı sınırsız sayıda fazla mesai ve ziyaretçi talebi oluşturabilmektedir.

### (Proof of Concept)
# 100 adet ardışık talep
for i in {1..100}; do
    curl "https://meyerapi.tcddteknik.com.tr/api/Dynamic?Name=...&islemtipi=i"
done

<img width="782" height="330" alt="Ratelimit 1" src="https://github.com/user-attachments/assets/80c12577-2630-4478-abbc-752c032d313c" />


<img width="808" height="315" alt="ratelimit 2" src="https://github.com/user-attachments/assets/4065257f-12b1-432a-9637-182be1812503" />

# Tüm istekler HTTP 200 OK dönmektedir

##  ZAFİYET 3: IDOR (Insecure Direct Object Reference)
### Açıklama
/api/Dynamic endpoint'inde siciller parametresi manipüle edilerek başka bir kullanıcı adına fazla mesai talebi oluşturulabilmektedir.
### (Proof of Concept)
Kullanıcı pentestnk (ID: 2710) ile sisteme giriş yapılır.
/api/Dynamic?Name=... endpoint'ine bir istek gönderilir ve yanıtta tüm kullanıcıların ID, Ad Soyad ve LoginName bilgileri döner.

<img width="786" height="329" alt="kullanıcı idleri" src="https://github.com/user-attachments/assets/763cf865-883d-4687-afce-d7d234d17c52" />


Bu response sayesinde U*** Ç**** isimli kullanıcının ID'sinin 350 olduğu tespit edilir.

pentestnk kendi ID'si olan siciller=2710 parametresi ile fazla mesai talebi oluşturur ve kendi adına işlem yapar.

<img width="782" height="333" alt="idor 2" src="https://github.com/user-attachments/assets/d2db7b1d-b035-4c40-b2d1-b46a5d6ec58e" />


Aynı istekte siciller=2710 parametresi silinip yerine siciller=350 yazılır.

<img width="783" height="336" alt="idor 3" src="https://github.com/user-attachments/assets/f516e40c-6641-40d5-adee-7b62fec52c0d" />


İstek tekrar gönderilir.

Sunucu HTTP/2 200 OK yanıtı döner ve U*** Ç*** adına fazla mesai talebi oluşturulmuş olur.

pentestnk kendi yetkisiyle, hiçbir yetki kontrolü olmadan başka bir kullanıcı adına işlem yapabilmiştir.


<img width="888" height="396" alt="idor 5" src="https://github.com/user-attachments/assets/941ba1bc-4370-479a-81dd-0257032109e5" />

### Supply Chain Saldırı Zinciri (3 CVE)

CVE-1 (Bilgi İfşası)                                          
 Saldırgan tüm personel ID'lerini ele geçirir                  
"U*** Ç*** ID = 350"                                        
                                                               
                                                                 
 CVE-2 (IDOR)                                                  
 Saldırgan ele geçirdiği ID ile başkası adına işlem yapar      
 "siciller=350 → U*** Ç*** adına talep"                      
                                                                
CVE-3 (Rate Limit)                                            
 Saldırgan aynı işlemi 100.000+ kez tekrarlar                  
"Sistem çöker, veritabanı şişer."


