<h1 align="center">منهجية صيد الثغرات (Bug Bounty) 2026 - نسخة محدّثة</h1>

<div align="center">

[![Author](https://img.shields.io/badge/Author-Amr%20Elsagaei-green)](#)
[![YouTube](https://img.shields.io/badge/AmrSecOfficial-c00?logo=youtube&logoColor=white)](https://www.youtube.com/@AmrSecOfficial)
[![Twitter](https://img.shields.io/badge/amrelsagaei-black?logo=X)](https://twitter.com/amrelsagaei)
[![Telegram](https://img.shields.io/badge/AmrSecOfficial-0088CC?logo=telegram&logoColor=white)](https://t.me/AmrSecOfficial)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-amrelsagaei-0073e3?logo=linkedin&labelColor=white)](https://www.linkedin.com/in/amrelsagaei)

</div>

<br>
<div style="text-align: center;">
    <a href="https://youtu.be/bAHNM8IInwE" target="_blank">
        <img src="https://amrelsagaei.com/storage/code/posts/bug-bounty-metho-2025.webp" alt="Bug Bounty Methodology" style="width: 100%;">
    </a>
</div>

<div align="right" dir="rtl">
أهلاً بيك في <strong>منهجية صيد الثغرات (Bug Bounty) - النسخة المحدّثة</strong>! المنهجية دي عبارة عن دليل أساسي يساعدك تبدأ رحلتك في مجال صيد الثغرات. بتوضح الخطوات الأساسية اللي تحتاجها عشان تتعامل مع الهدف بشكل فعّال، لكن التحدي الحقيقي بيفضل في إنك تكتشف الثغرات عالية التأثير بمهاراتك وإبداعك الشخصي. المنهجية دي هيتم تحديثها بشكل دوري كل ما تظهر تقنيات جديدة ومثيرة تساعد في تطوير عملية الاختبار.
</div>

<br>

## 📜 جدول المحتويات

| القسم | الوصف |
|---------|-------------|
| 1. [الاستطلاع](#1-الاستطلاع-وحصر-النطاقات-الفرعية) | حصر النطاقات الفرعية والفحص الأولي |
| 2. [الاكتشاف](#2-الاكتشاف-والفحص) | فحص HTTP واكتشاف الأصول |
| 3. [التعداد](#3-تقنيات-التعداد-المتقدمة) | تقنيات متقدمة واكتشاف البارامترات |
| 4. [الاختبار](#4-اختبار-الثغرات) | تقييم الثغرات |
| 5. [أسلوب العينين](#5-أسلوب-العينين-the-two-eye-approach) | إيه ده؟ |
| 6. [إثبات المفهوم (POC)](#6-إنشاء-إثبات-المفهوم-poc) | التوثيق والأدلة |
| 7. [كتابة التقرير](#7-كتابة-التقرير) | التوثيق النهائي |

---

<br>

## **1. الاستطلاع وحصر النطاقات الفرعية**

### **1.1 الحصر السلبي للنطاقات الفرعية (Passive)**
**🛠️الأدوات:** [Subfinder](https://github.com/projectdiscovery/subfinder), [Amass](https://github.com/OWASP/Amass), [CRTSH](https://crt.sh/), [Github-Search](https://github.com/gwen001/github-search), [bbot](https://github.com/blacklanternsecurity/bbot) *(جديدة)*

> **ملاحظة تحديث:** أداة **bbot** بقت من أقوى أدوات الاستطلاع لأنها بتدمج عشرات المصادر (passive + OSINT + crawling) في موديول واحد recursive، وبقت بديل شامل لسلسلة من الأدوات المنفصلة.

**Subfinder**
```bash
subfinder -d target.com -silent -all -recursive -o subfinder_subs.txt
```

**Amass (Passive Mode)**
```bash
amass enum -passive -d target.com -o amass_passive_subs.txt
```

**bbot (موديول الاستطلاع الشامل)**
```bash
bbot -t target.com -p subdomain-enum -o bbot_results/
```

**CRT.sh Query**
```bash
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | anew crtsh_subs.txt
```

**Github Dorking**
```bash
github-subdomains -d target.com -t YOUR_GITHUB_TOKEN -o github_subs.txt
```

**دمج النتائج**
```bash
cat *_subs.txt | sort -u | anew all_subs.txt
```

### **1.2 الحصر النشط للنطاقات الفرعية (Active)**

**🛠️الأدوات:** [MassDNS](https://github.com/blechschmidt/massdns), [PureDNS](https://github.com/d3mondev/puredns) *(جديدة)*, [Shuffledns](https://github.com/projectdiscovery/shuffledns), [DNSX](https://github.com/projectdiscovery/dnsx), [FFuF](https://github.com/ffuf/ffuf), [alterx](https://github.com/projectdiscovery/alterx) *(جديدة)*

> **ملاحظة تحديث:** **PureDNS** بقت الخيار المفضل لمعظم الصيادين لأنها بتجمع سرعة MassDNS مع فلترة ذكية للـ wildcard. وأداة **alterx** بقت تستخدم لتوليد تباديل (permutations) ذكية لأسماء النطاقات بدل الـ wordlists العادية.

**MassDNS**
```bash
massdns -r resolvers.txt -t A -o S -w massdns_results.txt wordlist.txt
```

**PureDNS (حل أدق مع فلترة wildcard)**
```bash
puredns resolve all_subs.txt -r resolvers.txt --wildcard-tests 20 -o resolved_subs.txt
```

**alterx (توليد تباديل للنطاقات)**
```bash
alterx -l resolved_subs.txt -o permutations.txt
puredns resolve permutations.txt -r resolvers.txt -o new_resolved_subs.txt
```

**Shuffledns**
```bash
shuffledns -d target.com -list all_subs.txt -r resolvers.txt -o active_subs.txt
```

**DNSX Resolution**
```bash
dnsx -l active_subs.txt -resp -o resolved_subs.txt
```

**FFuF Subdomain**
```bash
ffuf -u https://FUZZ.target.com -w wordlist.txt -t 50 -mc 200,403 -o ffuf_subs.txt
```

### **1.3 التعامل مع الأهداف المحددة (Non-Wildcard)**

**🛠️الأدوات:** [GAU](https://github.com/lc/gau), [Waybackurls](https://github.com/tomnomnom/waybackurls), [Katana](https://github.com/projectdiscovery/katana), [Hakrawler](https://github.com/hakluke/hakrawler)

**GAU**
```bash
gau target.example.com | anew gau_results.txt
```

**Waybackurls**
```bash
waybackurls target.example.com | anew wayback_results.txt
```

**Katana**
```bash
katana -u target.example.com -silent -jc -o katana_results.txt
```

**Hakrawler**
```bash
echo "https://target.example.com" | hakrawler -depth 2 -plain -js -out hakrawler_results.txt
```

### **تقنيات متقدمة إضافية**

**🛠️الأدوات:** [CloudEnum](https://github.com/initstring/cloud_enum), [AWSBucketDump](https://github.com/jordanpotti/AWSBucketDump), [S3Scanner](https://github.com/sa7mon/S3Scanner)

**Reverse DNS**
```bash
dnsx -ptr -l resolved_subs.txt -resp-only -o reverse_dns.txt
```

**ASN Enumeration**
```bash
amass intel -asn <ASN_NUMBER> -o asn_results.txt
```

**Cloud Asset Enumeration**
```bash
cloud_enum -k target.com
```

**التحقق من النتائج**
```bash
cat all_subs.txt | httpx -silent -title -o live_subdomains.txt
```

---

<br>

## **2. الاكتشاف والفحص**

### **2.1 فحص HTTP**

**🛠️الأدوات:** [httpx](https://github.com/projectdiscovery/httpx), [httprobe](https://github.com/tomnomnom/httprobe)

**HTTPX Probing**
```bash
httpx -l resolved_subs.txt -p 80,443,8080,8443 -silent -title -sc -ip -tech-detect -o live_websites.txt
```

> **ملاحظة تحديث:** أضيف خيار `-tech-detect` لأنه بقى أساسي في أي فحص حديث عشان يحدد التقنيات المستخدمة (CMS, Frameworks, WAF) من أول خطوة.

**Custom Filtering**
```bash
cat live_websites.txt | grep -i "login\|admin" | tee login_endpoints.txt
```

### **2.2 تحليل ملفات JavaScript**

**🛠️الأدوات:** [LinkFinder](https://github.com/GerbenJavado/LinkFinder), [subjs](https://github.com/lc/subjs), [JSFinder](https://github.com/Threezh1/JSFinder), [GF](https://github.com/tomnomnom/gf), [uro](https://github.com/s0md3v/uro) *(جديدة)*

> **ملاحظة تحديث:** أداة **uro** بقت ضرورية بعد تجميع آلاف الروابط من GAU/Wayback عشان تشيل التكرارات والـ noise وتسيب بس الروابط المهمة.

**JS Extraction**
```bash
cat live_websites.txt | waybackurls | grep "\.js" | anew js_files.txt
```

**تنقية الروابط باستخدام uro**
```bash
cat js_files.txt | uro -o cleaned_js_files.txt
```

**LinkFinder Analysis**
```bash
python3 linkfinder.py -i js_files.txt -o js_endpoints.txt
```

**البحث عن أنماط حساسة**
```bash
cat js_files.txt | gf aws-keys | tee aws_keys.txt
cat js_files.txt | gf urls | tee sensitive_urls.txt
```

**التحقق من صلاحية الـ API Keys**
```bash
curl -X GET "https://api.example.com/resource" -H "Authorization: Bearer <extracted_key>"
```

### **2.3 Google Dorking المتقدم**

**🛠️الأدوات:** [GitDorker](https://github.com/obheda12/GitDorker)

**Automated Dorking**
```bash
python3 GitDorker.py -tf <github_token.txt> -q target.com -d dorks.txt -o git_dorks_output.txt
```

**ملفات الأدمن/تسجيل الدخول**
```bash
site:*.example.com inurl:"*admin | login" | inurl:.php | .asp
```

**ملفات الإعدادات**
```bash
site:*.example.com ext:env | ext:yaml | ext:ini
```

**المفاتيح العامة**
```bash
site:*.example.com inurl:"id_rsa.pub" | inurl:".pem"
```

### **2.4 اكتشاف الروابط (URL Discovery)**

**🛠️الأدوات:** [Katana](https://github.com/projectdiscovery/katana), [Gospider](https://github.com/jaeles-project/gospider), [Hakrawler](https://github.com/hakluke/hakrawler)

**Katana Crawling**
```bash
katana -list live_websites.txt -jc -o katana_urls.txt
```

**Gospider**
```bash
gospider -s "https://target.com" -d 2 -o gospider_output/
```

**Hakrawler**
```bash
echo "https://target.com" | hakrawler -depth 3 -plain -out hakrawler_results.txt
```

### **2.5 تعداد الأرشيف (Archive Enumeration)**

**🛠️الأدوات:** [GAU](https://github.com/lc/gau), [Waybackurls](https://github.com/tomnomnom/waybackurls), [ParamSpider](https://github.com/devanshbatham/ParamSpider)

**Archive URL Collection**
```bash
gau --subs target.com | anew archived_urls.txt
waybackurls target.com | anew wayback_urls.txt
```

**استخراج البارامترات**
```bash
cat archived_urls.txt | grep "=" | anew parameters.txt
```

---

<br>

## **3. تقنيات التعداد المتقدمة**

### **3.1 اكتشاف البارامترات**

**🛠️الأدوات:** [Arjun](https://github.com/s0md3v/Arjun), [ParamSpider](https://github.com/devanshbatham/ParamSpider), [FFuF](https://github.com/ffuf/ffuf)

**Arjun Parameter Discovery**
```bash
arjun -u "https://target.example.com" -m GET,POST --stable -o params.json
```

**ParamSpider Web Parameters**
```bash
python3 paramspider.py --domain target.com --exclude woff,css,js --output paramspider_output.txt
```

**FFuF Parameter Bruteforce**
```bash
ffuf -u https://target.com/page.php?FUZZ=test -w /usr/share/wordlists/params.txt -o parameter_results.txt
```

### **3.2 تعداد أصول الـ Cloud**

**🛠️الأدوات:** [CloudEnum](https://github.com/initstring/cloud_enum), [AWSBucketDump](https://github.com/jordanpotti/AWSBucketDump), [S3Scanner](https://github.com/sa7mon/S3Scanner)

**Cloud Bucket Enumeration**
```bash
cloud_enum -k target.com -b buckets.txt -o cloud_enum_results.txt
```

**S3 Bucket Access Test**
```bash
aws s3 ls s3://<bucket_name> --no-sign-request
```

**S3 Bucket Content Dump**
```bash
python3 AWSBucketDump.py -b target-bucket -o dumped_data/
```

### **3.3 اكتشاف المحتوى (Content Discovery)**

**🛠️الأدوات:** [Feroxbuster](https://github.com/epi052/feroxbuster), [FFuF](https://github.com/ffuf/ffuf), [Dirsearch](https://github.com/maurosoria/dirsearch)

**Feroxbuster**
```bash
feroxbuster -u https://target.com -w /usr/share/wordlists/common.txt -r -t 20 -o recursive_results.txt
```

**Dirsearch**
```bash
dirsearch -u https://target.com -w /usr/share/wordlists/content_discovery.txt -e php,html,js,json -x 404 -o dirsearch_results.txt
```

**FFuF Recursive**
```bash
ffuf -u https://target.com/FUZZ -w /usr/share/wordlists/content_discovery.txt -mc 200,403 -recursion -recursion-depth 3 -o ffuf_results.txt
```

### **3.4 تعداد الـ API**

**🛠️الأدوات:** [Kiterunner](https://github.com/assetnote/kiterunner), [Postman](https://www.postman.com/), [Burp Suite](https://portswigger.net/burp)

**Kiterunner**
```bash
kr scan https://api.target.com -w /usr/share/kiterunner/routes-large.kite -o api_routes.txt
```

### **3.5 ربط ASN**

**🛠️الأدوات:** [Amass](https://github.com/OWASP/Amass), [Shodan](https://www.shodan.io/), [Censys](https://censys.io/)

**ASN Lookup**
```bash
amass intel -asn <ASN_Number> -o asn_ips.txt
```

**Shodan Enumeration**
```bash
shodan search "net:<ip_range>" --fields ip_str,port --limit 100
```

**Censys Asset Search**
```bash
censys search "autonomous_system.asn:<ASN_Number>" -o censys_assets.txt
```

### **3.6 فحص لقطات الشاشة البصري (جديد)**

**🛠️الأدوات:** [gowitness](https://github.com/sensepost/gowitness)

> **ملاحظة تحديث:** بقت خطوة شبه إلزامية قبل الاختبار اليدوي — بتدّيك لمحة بصرية سريعة عن آلاف الـ hosts عشان تحدد الأهداف المهمة (لوحات تحكم، صفحات لوجين، أدوات داخلية) بدون ما تفتح كل رابط يدويًا.

```bash
gowitness scan file -f live_websites.txt --screenshot-path screenshots/
```

---

<br>

## **4. اختبار الثغرات**

### **4.0 الفحص الآلي بالقوالب (Nuclei) — خطوة جديدة أساسية**

**🛠️الأداة:** [Nuclei](https://github.com/projectdiscovery/nuclei)

> **ملاحظة تحديث مهمة:** النسخة الأصلية من المنهجية ماكنتش فيها Nuclei، وده بقى غريب لأنها بقت الأداة الأساسية للفحص الآلي المبني على القوالب (CVEs، misconfigurations، exposures، takeovers) في أي منهجية حديثة. لازم تحدّث القوالب بانتظام.

```bash
nuclei -update-templates
nuclei -l live_websites.txt -severity critical,high,medium -o nuclei_results.txt
```

**فحص الـ Subdomain Takeover بالـ Nuclei**
```bash
nuclei -l resolved_subs.txt -t takeovers/ -o takeover_results.txt
```

### **4.1 الثغرات ذات الأولوية العالية**

**🐞اختبار CSRF**
```bash
cat live_websites.txt | gf csrf | tee csrf_endpoints.txt
```

**🐞اختبار LFI**
```bash
cat live_websites.txt | gf lfi | qsreplace "/etc/passwd" | xargs -I@ curl -s @ | grep "root:x:" > lfi_results.txt
```

**🐞اختبار RCE**
```bash
curl -X POST -F "file=@exploit.php" https://target.com/upload
```

**🐞اختبار SQLi**
```bash
ghauri -u "https://target.com?id=1" --dbs --batch
```

**🐞البحث عن بيانات حساسة**
```bash
cat js_files.txt | grep -Ei "key|token|auth|password" > sensitive_data.txt
```

**🐞اختبار Open Redirect**
```bash
cat urls.txt | grep "=http" | qsreplace "https://evil.com" | xargs -I@ curl -I -s @ | grep "evil.com"
```

---

<br>

## **5. أسلوب العينين (The "Two-Eye" Approach) 👀**
1. **العين الأولى:** ركّز على اختبار كل نطاق فرعي أو endpoint أو parameter تم جمعه بحثًا عن الثغرات الشائعة.
2. **العين الثانية:** حدد النتائج "المثيرة للاهتمام" زي بيانات اعتماد مكشوفة، نطاقات فرعية منسية، أو لوحات تحكم إدارية.

### **خطوات عملية:**
- لو تم اكتشاف ثغرة، اعمل إثبات مفهوم (POC) واختبر تأثيرها.
- لو مفيش ثغرات، انتقل لاختبار أعمق على نطاقات فرعية أو endpoints فريدة.

---

<br>

## **6. إنشاء إثبات المفهوم (POC)**

### **🎥 إثبات بالفيديو**

وضّح الثغرات عمليًا باستخدام أدوات تسجيل الشاشة زي Greenshot أو OBS Studio.

### **📸 إثبات بالصور**

التقط لقطات شاشة واضحة مع تعليقات توضيحية لكل خطوة.

- **🛠️الأداة:** Greenshot.

---

<br>

## **7. كتابة التقرير**

### **📝 هيكل التقرير**

1. **الملخص التنفيذي**
   - نطاق الهدف
   - الجدول الزمني للاختبار
   - ملخص النتائج الرئيسية
   - تصنيف المخاطر

2. **التفاصيل الفنية**
   - عنوان الثغرة
   - تصنيف الخطورة
   - المكونات المتأثرة
   - الوصف الفني
   - خطوات إعادة الإنتاج
   - تحليل التأثير
   - الأدلة الداعمة (POC)

3. **الحلول المقترحة**
   - توصيات تفصيلية
   - خطوات التخفيف
   - ضوابط أمنية إضافية
   - مراجع ومصادر

4. **المواد الداعمة**
   - عروض الفيديو
   - لقطات الشاشة والتعليقات
   - سجلات طلبات/استجابات HTTP
   - مقتطفات الكود
   - الجدول الزمني للاكتشاف

### **أفضل الممارسات**

- اكتب وصفًا واضحًا ومختصرًا
- اذكر خطوات إعادة إنتاج تفصيلية
- قدّم نصائح علاجية قابلة للتنفيذ
- ادعم النتائج بالأدلة
- استخدم تنسيقًا احترافيًا
- أبرز التأثير على الأعمال
- أضف خطوات التحقق

### **قالب التقرير**

```markdown
# تقرير ثغرة: [العنوان]

## نظرة عامة
- الخطورة: [حرجة/عالية/متوسطة/منخفضة]
- درجة CVSS: [الدرجة]
- المكوّن المتأثر: [المكوّن]

## الوصف
[وصف فني تفصيلي]

## خطوات إعادة الإنتاج
1. [الخطوة 1]
2. [الخطوة 2]
3. [الخطوة n...]

## التأثير
[التأثير التقني وعلى الأعمال]

## إثبات المفهوم
[لقطات شاشة، فيديوهات، كود]

## التوصيات
[توصيات تفصيلية للإصلاح]

## المراجع
[CVE, CWE، مصادر ذات صلة]
```

---

<br>

## 📄 الترخيص
جميع الحقوق محفوظة © 2026 **عمرو الصغاعي (AmrSec)**.
