# Insider Threat Forensics Analyzer

**Şirket içi ajan (Insider Threat) tespiti için Python tabanlı adli bilişim analiz aracı**

Bu araç, şüpheli bir kullanıcının bilgisayarında (**Windows 7+** ve **macOS**) hızlı ve adli bilişim standartlarına uygun **live forensics** analizi yapar.  
Amaç: Kalıcılık mekanizmalarını, C2 iletişimini ve sistem anomalilerini tespit ederek delil toplamaktır.

---

## 🎯 Senaryo

Şirket içinde bir **ajan** tespit edildi.  
Görev: Ajanın bilgisayarına erişim sağlandıktan sonra adli analiz yaparak kanıtları toplamak ve profesyonel rapor üretmek.

---

## 📋 Analiz Aşamaları

### 1. Aşama: Sistem Bilgisi Toplama
- İşletim sistemi, hostname, kullanıcı, domain ve uptime bilgileri
- CPU, RAM, disk kullanımı gibi donanım detayları

### 2. Aşama: Kayıt Defteri Analizi (Windows)
- Run, RunOnce ve diğer persistence anahtarları
- Şüpheli registry girdilerinin tespiti

### 3. Aşama: Ağ Durumu Kontrolü
- Aktif bağlantılar ve süreç eşleştirmeleri
- Açık portlar, dinleyen servisler, DNS cache

### 4. Aşama: Raporlama
- JSON, TXT ve HTML formatında detaylı rapor üretimi
- Zaman damgası ve sistem bilgileriyle zenginleştirilmiş çıktı

### 5. Aşama: Kalıcılık Tespiti
- Scheduled Tasks, Servisler, Startup klasörleri
- Registry Run keys ve diğer kalıcılık teknikleri

### 6. Aşama: C2 Port Kontrolü
- Bilinen C2 portları taraması (4444, 8080, 443, 1337 vb.)
- Şüpheli IP ve portlara yapılan sürekli bağlantıların tespiti
- Process ile bağlantı ilişkisi analizi

### 7. Aşama: Sonuçların Kaydedilmesi ve Hashlenmesi
- Tüm bulgular `forensic_report_[tarih]` klasörüne kaydedilir
- Kanıt bütünlüğü için **SHA-256 hash** değerleri hesaplanır
- Hashler `HASHES.txt` dosyasına kaydedilir

---

## 🚀 Kullanım

```bash
# Gereksinimleri yükleyin
pip install -r requirements.txt

# Aracı çalıştırın
python forensics_analyzer.py --output ./forensic_report --verbose

PROJE YAPISI
insider-threat-forensics/
├── forensics_analyzer.py
├── modules/                  # system_info.py, registry_analysis.py vb.
├── reports/                  # Raporların kaydedileceği klasör
├── requirements.txt
├── README.md
├── LICENSE
└── .gitignore
