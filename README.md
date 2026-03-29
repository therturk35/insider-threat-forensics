# Merhaba bu taskte senaryo şirket içindeki ajanı yakalamak ve onun bilgisayarında (windows/MAC farketmeksizin) adli bilişim analizi yapmak.
# 1. Aşama: Sistem Bilgisi Toplama
# 2. Aşama: Kayıt Defteri Analizi
# 3. Aşama: Ağ Durumu Kontrolü
# 4. Aşama: Raporlama
# 5. Aşama: Kalıcılık Tespiti
# 6. Aşama: C2 Port Kontrolü
# 7. Aşama: Sonuçların Kaydedilmesi ve Hashlenmesi


import platform
import os
import getpass
import winreg
import socket
import json
import hashlib
from datetime import datetime
import subprocess



#aşama 1: sistem bilgisi toplama
def sistem_ozeti():
    return {
        "isletim_sistemi": f"{platform.system()} {platform.release()}",
        "kullanici_adi": getpass.getuser(),
        "bilgisayar_adi": os.environ.get('COMPUTERNAME', 'Bilinmiyor'),
        "sistem_versiyonu": platform.version()
    }



#aşama 2: kayıt defteri analizi
def registry_analysis():
    reg_path = r"Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced"
    try:
        key = winreg.OpenKey(winreg.HKEY_CURRENT_USER, reg_path)
        hidden_value, _ = winreg.QueryValueEx(key, "Hidden")
        hidefileext_value, _ = winreg.QueryValueEx(key, "HideFileExt")
        winreg.CloseKey(key)
        return {
            "Hidden_Status": "Gizli Dosyalar Gorunur" if hidden_value == 1 else "Gizli Dosyalar Sakli",
            "HideFileExt_Status": "Uzantilar Sakli" if hidefileext_value == 1 else "Uzantilar Gorunur"
        }
    except PermissionError:
        return {"Hata": "Yetki Yetersiz"}
    except:
        return {"Hata": "Registry Anahtarı Bulunamadı"}



#aşama 3: ağ durumu kontrolü
def internet_durumu():
    try:
        socket.create_connection(("www.google.com", 80), timeout=2)
        return "Baglanti Aktif (HTTP OK)"
    except OSError:
        return "Baglanti Yok/Kisitli (HTTP Fail)"



#aşama 4: raporlama
def rapor_yaz(sistem, registry, network):
    rapor_icerik = {
        "analiz_zamani": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        "sistem_bilgileri": sistem,
        "registry_bulgulari": registry,
        "ag_durumu": network
    }
    file_name = "evidence_report.json"
    with open(file_name, "w", encoding="utf-8") as f:
        json.dump(rapor_icerik, f, indent=4, ensure_ascii=False)
    return file_name



#aşama 5: kalıcılık tespiti
def kalicilik_tespiti(rapor_dosyasi_adi):
    persistence_found = False
    
    # Raporlama fonksiyonunu kullanabilmek için genel bir raporlama yapalım
    def rapor_yaz_persistence(mesaj):
        with open(rapor_dosyasi_adi, "a", encoding="utf-8") as f:
            f.write(f"{datetime.now().strftime('%Y-%m-%d %H:%M:%S')} - [PERSISTENCE] {mesaj}\n")
        print(f"[PERSISTENCE] {mesaj}")

    rapor_yaz_persistence("\n[Persistence Analysis]")

    # 1. WINDOWS KONTROLÜ
    if SYS_PLATFORM == "Windows":
        rapor_yaz_persistence("Windows (HKCU\\Run) anahtarı taranıyor...")
        try:
            key = winreg.OpenKey(winreg.HKEY_CURRENT_USER, r"Software\Microsoft\Windows\CurrentVersion\Run")
            i = 0
            while True:
                try:
                    value_name, value_data, _ = winreg.EnumValue(key, i)
                    if value_data:
                        rapor_yaz_persistence(f"Kalıcılık tespit edildi (HKCU\\Run): {value_name} -> {value_data}")
                        persistence_found = True
                    i += 1
                except OSError:
                    break
            winreg.CloseKey(key)
            if not persistence_found:
                rapor_yaz_persistence("HKCU\\Run anahtarında kayıtlı başlangıç öğesi bulunamadı.")

        except Exception as e:
            rapor_yaz_persistence(f"Windows persistence analiz hatası: {e}")
            print(f"[!] Windows persistence analiz hatası: {e}")

    # 2. MACOS KONTROLÜ
    elif SYS_PLATFORM == "Darwin":  # macOS
        rapor_yaz_persistence("macOS'ta LaunchAgents ve LaunchDaemons taranıyor...")

        paths_to_check = [
            Path.home() / "Library" / "LaunchAgents",
            Path("/Library/LaunchAgents"),
            Path("/Library/LaunchDaemons")
        ]

        for folder in paths_to_check:
            if folder.exists():
                for plist_file in folder.glob("*.plist"):
                    # Apple'ın kendi dosyalarını filtreleme
                    if not plist_file.name.startswith("com.apple."):
                        rapor_yaz_persistence(f"Kalıcılık tespit edildi (LaunchItem): {plist_file}")
                        persistence_found = True
        
        # Login Items (Giriş Öğeleri) basit kontrolü
        try:
            result = subprocess.run(
                ["osascript", "-e", 'tell application "System Events" to get every login item'],
                capture_output=True, text=True, timeout=5, check=True
            )
            if result.stdout.strip():
                rapor_yaz_persistence(f"Login Items bulundu: {result.stdout.strip()}")
                persistence_found = True
        except subprocess.CalledProcessError:
             rapor_yaz_persistence("Login Item kontrolü başarısız oldu (osascript hatası).")
        except Exception:
            pass

        if not persistence_found:
            rapor_yaz_persistence("macOS'ta bilinen persistence mekanizmalarında şüpheli öğe bulunamadı.")

    # 3. LİNUX KONTROLÜ (EKLEME)
    elif SYS_PLATFORM == "Linux":
        rapor_yaz_persistence("Linux platformu taranıyor (Cron ve Autostart kontrolü)...")
        
        # Cron Jobs Kontrolü (Basit bir başlangıç)
        try:
            # Kullanıcının crontab'ını kontrol et
            cron_output = subprocess.check_output(["crontab", "-l"], text=True, stderr=subprocess.DEVNULL)
            if cron_output.strip():
                rapor_yaz_persistence(f"Kullanıcı Cron Job'ları bulundu:\n{cron_output.strip()}")
                persistence_found = True
        except subprocess.CalledProcessError:
            rapor_yaz_persistence("Kullanıcı Cron Job'u yok veya okunamadı.")
        except Exception as e:
            rapor_yaz_persistence(f"Cron kontrol hatası: {e}")

        # Basit Autostart kontrolü (XDG standartına göre)
        autostart_path = Path.home() / ".config" / "autostart"
        if autostart_path.exists():
            desktop_files = list(autostart_path.glob("*.desktop"))
            if desktop_files:
                rapor_yaz_persistence(f"Autostart dizininde {len(desktop_files)} adet dosya bulundu.")
                persistence_found = True

        if not persistence_found:
            rapor_yaz_persistence("Linux'ta bilinen persistence mekanizmalarında şüpheli öğe bulunamadı.")

    else:
        rapor_yaz_persistence(f"Desteklenmeyen platform: {SYS_PLATFORM}")

            



#aşama 6: C2 port kontrolü
def network_kontrol():

    rapor_yaz("\n[NETWORK ANALYSIS]")

    try:
        output = subprocess.check_output(["netstat", "-an"]).decode()

        if "4444" in output:
            rapor_yaz("⚠️ Port 4444 üzerinde aktif bağlantı tespit edildi")
            print("[!] C2 bağlantısı olabilir")
        else:
            rapor_yaz("Port 4444 bağlantısı bulunamadı")
            print("[✓] C2 bağlantısı yok")

    except Exception as e:
        rapor_yaz(f"Network analiz hatası: {e}")



#aşama 7: sonuçların kaydedilmesi ve hashlenmesi
def hash_hesapla(dosya_yolu):
    sha256_hash = hashlib.sha256()
    with open(dosya_yolu, "rb") as f:
        for byte_block in iter(lambda: f.read(4096), b""):
            sha256_hash.update(byte_block)
    return sha256_hash.hexdigest()
