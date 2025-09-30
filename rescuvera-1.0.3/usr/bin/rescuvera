#!/usr/bin/env python3

import sys
import os
import subprocess 
import re 
import json
import datetime 
from PyQt5.QtWidgets import (QApplication, QWidget, QVBoxLayout, QHBoxLayout,
                             QLabel, QComboBox, QLineEdit, QPushButton, 
                             QTabWidget, QGridLayout, QDialog, QFrame, QRadioButton,
                             QFileDialog, QMessageBox, QProgressBar, QStackedWidget, QSizePolicy) 
from PyQt5.QtCore import Qt, QProcess, QLocale, QTimer
from PyQt5.QtGui import QIcon, QPixmap, QMovie

# Qt platform plugin sorununu çözmek için ortam değişkenlerini ayarla
# Bu iki satır, programın gnome gibi ortamlarda sorunsuzca açılması için gereklidir.
os.environ['QT_QPA_PLATFORM_PLUGIN_PATH'] = ''
os.environ['QT_PLUGIN_PATH'] = ''

# --- ÇEVİRİ SÖZLÜKLERİ ---
# Rescuvera ve Language kelimeleri çeviri kapsamı dışındadır.
TRANSLATIONS = {
    'tr': {
        'title_main': "Advanced Data Rescue GUI",
        'tab_image': "Ham İmaj Al",
        'tab_ext4': "Ext4 Dosya Sistemi",
        'tab_all': "Tüm Dosya Sistemleri",
        'image_title': "İmaj Alma Ayarları",
        'image_source_label': "Kaynak Aygıt/Partisyon:",
        'image_target_type_label': "Hedef Tipi:",
        'radio_file': "İmaj Dosyası",
        'radio_device': "Başka Bir Aygıt",
        'image_target_label': "Hedef:",
        'image_browse_file_save': "Dosya Kaydet",
        'image_file_filter': "İmaj Dosyaları (*.img *.dd);;Tüm Dosyalar (*)",
        'image_target_device_warning': "DİKKAT: Veriler silinecek! Bir Hedef Disk/Partisyon Seçin.",
        'image_log_label': "Günlük Dosyası:",
        'image_log_filter': "Log Dosyaları (*.log);;Tüm Dosyalar (*)",
        'button_browse': "Gözat",
        'button_start_image': "İmaj Al",
        'status_waiting': "İşlem bekleniyor.",
        'about_button': "Hakkında",
        'info_label': "Önemli:",
        'info_button': "Bilgilendirme Metni",
        'about_version': "Sürüm:",
        'about_license': "Lisans:",
        'about_lang': "Programlama Dili:",
        'about_ui': "Arayüz:",
        'about_dev': "Geliştirici:",
        'about_text': ("Bu program, depolama birimlerinden veri kurtarmada kullanılan, "
                       "temelde ddrescue, foremost ve ext4magic yazılımlarını kullanan bir arayüzdür. " 
                       "İlk olarak Ham İmaj Al sekmesinde imaj almayı ve sonra bu imajlar üzerinde kurtarma yapmayı destekler.<br><br>"
                       "<b style='text-align: center;'>Bu program, hiçbir garanti getirmez.</b>"),
        'about_close': "Kapat",
        'about_dialog_title': "Hakkında",
        'device_scan_wait': "Aygıtlar Taranıyor...",
        'device_not_found': "Sistemde blok aygıtı bulunamadı.",
        'device_lsblk_error': "lsblk kullanılamıyor.",
        
        # Kurtarma Sekmeleri
        'rescue_selection_title': "Kurtarma Yöntemi Seçimi",
        'radio_image_rescue': "Ham İmaj Üzerinden Kurtar",
        'radio_device_rescue': "Doğrudan Aygıt Üzerinden Kurtar",
        'rescue_file_title': "Kurtarma Ayarları",
        'rescue_from': "Şuradan:",
        'rescue_to': "Şuraya:",
        'rescue_target_folder_select': "Hedef Klasör Seç",
        'button_rescue': "Kurtar",
        
        # Hata/Uyarı Mesajları (Çevirilerin büyük kısmı bu kısımda)
        'msg_err_title': "Hata",
        'msg_warn_title': "Uyarı",
        'msg_info_title': "ÖNEMLİ",
        'msg_crit_title': "KRİTİK ONAY GEREKLİ",
        
        # İmaj Alma Mesajları
        'msg_img_ok_title': "İşlem Tamamlandı",
        'msg_img_ok_text': "İmaj alma işlemi başarıyla tamamlandı. Log dosyasını kontrol ediniz.",
        'msg_img_fail_title': "İşlem Hatası",
        'msg_img_fail_status': "İşlem Başarısız Oldu veya İptal Edildi.",
        'msg_img_fail_details': ("**Çıkış Kodu: {code}.**\n\n"
                                 "**Olası Nedenler:**\n"
                                 "1. **Yetkilendirme:** Pkexec şifresi girilmedi veya yetki reddedildi.\n"
                                 "2. **Komut Yolu:** ddrescue komutu `/usr/bin/ddrescue` yolunda bulunamadı.\n"
                                 "3. **Giriş/Çıkış Hatası:** Kaynak/Hedef aygıt veya dosya erişilemez (özellikle diskten diske kopyalamada).\n"
                                 "4. **Aygıt Zaten Bağlı:** Kaynak veya Hedef aygıtı takılı (mount) ise bu hataya neden olabilir.\n\n"
                                 "**Hata Çıktısı (Son Kısım):**\n"
                                 "----------------------------------------\n"
                                 "{snippet}"),
        'msg_img_start_status': "İşlem Sürüyor...",
        'msg_img_progress_status': "Kurtarma: %{percent}, Kurtarılan: {rescued}, Hız: {rate}",
        'msg_img_invalid_source': "Lütfen geçerli bir Kaynak Aygıt seçimi yapın.",
        'msg_img_no_target_file': "Lütfen bir **Hedef İmaj Dosyası** yolu belirleyin.",
        'msg_img_no_target_device': "Lütfen bir **Hedef Aygıt** (Verilerin Kopyalanacağı Disk/Partisyon) seçimi yapın.",
        'msg_img_source_target_same': "Kaynak ve Hedef Aygıt aynı olamaz. Bu, kaynak diskinizin silinmesine neden olur!",
        'msg_img_confirm_copy': ("**UYARI: Kaynak {source} aygıtındaki/partisyonundaki tüm veriyi Hedef {target} aygıtına/partisyonuna kopyalayacaktır.**\n\n"
                                 "**Hedef aygıttaki {target} TÜM VERİ KAYBOLACAKTIR!**\n\n"
                                 "Bu işleme başlamak istediğinizden emin misiniz? (Yetki onayı için grafik pencere açılacaktır)"),
        'msg_img_no_log': "Lütfen ddrescue için bir Günlük Dosyası yolu belirleyin.",
        'msg_img_command_missing': "Gerekli komutlardan (pkexec, ddrescue) biri bulunamadı. Lütfen kurulu olduklarından emin olun.",
        'msg_img_command_failed_start': "Komut çalıştırma başlatılamadı. pkexec ve ddrescue kurulu mu? Hata: {error}",

        # Sahiplik Değiştirme Mesajları
        'msg_chown_status': "İzinler Ayarlanıyor: Klasör Sahipliği ({user}) atanıyor... (İkinci şifre ekranını bekleyin.)",
        'msg_chown_missing': "Yetkilendirme aracı **pkexec** bulunamadı. İzin değiştirilemiyor.",
        'msg_chown_ok_status': "İzinler Başarıyla Ayarlandı. İşlem Tamamlandı.",
        'msg_chown_ok_info': "Kurtarılan dosyaların bulunduğu klasörün ({folder_name}) sahipliği başarıyla normal kullanıcınıza ayarlandı. Artık dosyalarınıza erişebilirsiniz.",
        'msg_chown_fail_status': "İzin Ayarlama Hatası (Kod: {code}). Klasör Root'a ait kaldı.",
        'msg_chown_fail_crit': ("Klasör izinleri otomatik olarak ayarlanamadı (Hata Kodu: {code}).\n"
                                "Lütfen bir terminal açıp **pkexec chown -R $USER:$USER {target_folder}** komutunu kendiniz çalıştırın."),
        'msg_chown_fail_start': "İzin değiştirme komutu başlatılamadı. Hata: {error}",
        
        # ext4 Kurtarma Mesajları
        'msg_ext4_ui_missing': "Arayüz bileşenleri yüklenemedi. Program hatası.",
        'msg_ext4_command_missing': "Gerekli komutlardan (pkexec, ext4magic) biri bulunamadı. Lütfen kurulu olduklarından emin olun.",
        'msg_ext4_invalid_image': "Lütfen geçerli bir Ham İmaj Dosyası seçin.",
        'msg_ext4_invalid_device': "Lütfen kurtarma yapılacak geçerli bir Kaynak Aygıt (Disk/Partisyon) seçin.",
        'msg_ext4_mounted_warning': ("**Dikkat:** Aygıt üzerindeki kurtarma işlemi ({source}), cihazın **bağlı (mounted) OLMAMASINI** gerektirir. Lütfen işleme başlamadan önce cihazın bağlı olmadığından emin olun."),
        'msg_ext4_no_target_folder': "Lütfen kurtarılan dosyaların kaydedileceği geçerli bir Hedef Klasör seçin.",
        'msg_ext4_target_folder_fail': "Hedef Klasör ({folder}) oluşturulamıyor veya erişilemiyor. Lütfen yazma izninizi kontrol edin. Hata: {error}",
        'msg_ext4_start_status': "Ext4 Kurtarma Başlatılıyor... (Hedef Klasör: {folder_name})",
        'msg_ext4_inprogress_status': "Ext4 Kurtarma: İşleniyor...",
        'msg_ext4_ok_status': "Ext4 Kurtarma (ext4magic) İşlemi Başarıyla Tamamlandı. İzinler Ayarlanıyor...",
        'msg_ext4_ok_info': "Dosya kurtarma işlemi başarıyla tamamlandı. Kurtarılan dosyalar Hedef Klasörün altındaki **{folder_name}** dizinindedir. Klasör izinleri şimdi ayarlanacak.",
        'msg_ext4_fail_status': "Ext4 Kurtarma İşlemi Başarısız Oldu.",
        'msg_ext4_fail_crit': ("**Çıkış Kodu: {code}.**\n\n"
                               "**KRİTİK HATA:** ext4magic komutu başarısız oldu.\n"
                               "**EN OLASI NEDENLER:**\n"
                               "1. **Kaynak Bağlı:** Kurtarma yapılan aygıt/imaj **kesinlikle bağlı (mounted) olmamalıdır**.\n"
                               "2. **ext4magic Konumu:** Komut yolu (`/usr/sbin/ext4magic`) bulunamadı.\n"
                               "3. **Süper Blok Hatası:** ext4magic otomatik olarak yedek süper blokları denemiş olsa bile, dosya sistemi çok ciddi hasar görmüş olabilir.\n\n"
                               "**Hata Çıktısı (İlk 500 Karakter):**\n"
                               "----------------------------------------\n"
                               "{snippet}..."),
        'msg_ext4_command_failed_start': "Komut çalıştırma başlatılamadı. pkexec ve ext4magic kurulu mu? Hata: {error}",

        # foremost Kurtarma Mesajları
        'msg_all_inprogress_status': "Kurtarma işlemi: Sürüyor...",
        'msg_all_ok_status': "Derin Kurtarma İşlemi Başarıyla Tamamlandı. İzinler Ayarlanıyor...",
        'msg_all_ok_info': "Dosya kurtarma işlemi başarıyla tamamlandı. Kurtarılan dosyalar Hedef Klasörün içinde (**Rescuvera-Zaman Damgası** adıyla oluşturulan alt klasöre) kaydedilmiş olmalıdır. Klasör izinleri şimdi ayarlanacak.",
        'msg_all_fail_status': "Derin Kurtarma İşlemi Başarısız Oldu.",
        'msg_all_fail_crit': ("**Çıkış Kodu: {code}.**\n\n"
                              "**KRİTİK UYARI:** Yetkilendirme (Polkit) başarılı olmasına rağmen, komutun kendisi hemen başarısız oldu (Kod 1).\n"
                              "**EN OLASI NEDENLER:**\n"
                              "1. **Kaynak Bağlı:** Kurtarma yapılan aygıt bağlı (mounted) olmamalıdır.\n"
                              "2. **Kaynak Okunamaz:** Kaynak dosya/aygıt bulunamadı veya erişilemez durumda.\n"
                              "3. **Komut Yolu:** foremost komutu (`/usr/bin/foremost`) bulunamadı.\n\n"
                              "**Hata Çıktısı:**\n"
                              "----------------------------------------\n"
                              "{error}"),
        'msg_all_invalid_image': "Lütfen geçerli bir Ham İmaj Dosyası seçin.",
        'msg_all_invalid_device': "Lütfen kurtarma yapılacak geçerli bir Kaynak Aygıt (Disk/Partisyon) seçin.",
        'msg_all_command_missing': "Gerekli komutlardan (pkexec, foremost) biri bulunamadı. Lütfen kurulu olduklarından emin olun.",
        'msg_readme_not_found': "ReadMe dosyası bulunamadı:\n{pdf_path}",
        'msg_xdg_open_missing': "ReadMe dosyası açılamadı.\n'xdg-open' komutu sisteminizde bulunamadı.",
        'msg_readme_open_error': "ReadMe dosyası açılırken hata oluştu: {error}",
    },
    'en': {
        'title_main': "Advanced Data Rescue GUI",
        'tab_image': "Raw Image Acquisition",
        'tab_ext4': "Ext4 File System Rescue",
        'tab_all': "All File Systems Rescue",
        'image_title': "Image Acquisition Settings",
        'image_source_label': "Source Device/Partition:",
        'image_target_type_label': "Target Type:",
        'radio_file': "Image File",
        'radio_device': "Another Device",
        'image_target_label': "Target:",
        'image_browse_file_save': "Save File",
        'image_file_filter': "Image Files (*.img *.dd);;All Files (*)",
        'image_target_device_warning': "CAUTION: Data will be erased! Select a Target Disk/Partition.",
        'image_log_label': "Log File:",
        'image_log_filter': "Log Files (*.log);;All Files (*)",
        'button_browse': "Browse",
        'button_start_image': "Acquire Image",
        'status_waiting': "Operation pending.",
        'about_button': "About",
        'info_label': "Important:",
        'info_button': "Information Text",
        'about_version': "Version:",
        'about_license': "License:",
        'about_lang': "Programming Language:",
        'about_ui': "Interface:",
        'about_dev': "Developer:",
        'about_text': ("This program is a GUI for data recovery from storage units, "
                       "fundamentally utilizing ddrescue, foremost, and ext4magic software. " 
                       "It supports first acquiring an image via the Raw Image Acquisition tab and then performing recovery on those images.<br><br>"
                       "<b style='text-align: center;'>This program comes with no warranty.</b>"),
        'about_close': "Close",
        'about_dialog_title': "About",
        'device_scan_wait': "Scanning Devices...",
        'device_not_found': "No block device found on the system.",
        'device_lsblk_error': "lsblk is not available.",
        
        # Rescue Tabs
        'rescue_selection_title': "Recovery Method Selection",
        'radio_image_rescue': "Recover From Raw Image",
        'radio_device_rescue': "Recover Directly From Device",
        'rescue_file_title': "Recovery Settings",
        'rescue_from': "From:",
        'rescue_to': "To:",
        'rescue_target_folder_select': "Select Target Folder",
        'button_rescue': "Recover",
        
        # Error/Warning Messages (Most of the translations are here)
        'msg_err_title': "Error",
        'msg_warn_title': "Warning",
        'msg_info_title': "IMPORTANT",
        'msg_crit_title': "CRITICAL CONFIRMATION REQUIRED",
        
        # Image Acquisition Messages
        'msg_img_ok_title': "Operation Complete",
        'msg_img_ok_text': "Image acquisition completed successfully. Please check the log file.",
        'msg_img_fail_title': "Operation Error",
        'msg_img_fail_status': "Operation Failed or Cancelled.",
        'msg_img_fail_details': ("**Exit Code: {code}.**\n\n"
                                 "**Possible Causes:**\n"
                                 "1. **Authorization:** Pkexec password not entered or permission denied.\n"
                                 "2. **Command Path:** ddrescue command not found at `/usr/bin/ddrescue`.\n"
                                 "3. **I/O Error:** Source/Target device or file is inaccessible (especially in disk-to-disk copy).\n"
                                 "4. **Device Already Mounted:** If the Source or Target device is mounted, it might cause this error.\n\n"
                                 "**Error Output (Last Part):**\n"
                                 "----------------------------------------\n"
                                 "{snippet}"),
        'msg_img_start_status': "Operation in Progress...",
        'msg_img_progress_status': "Rescue: %{percent}, Rescued: {rescued}, Rate: {rate}",
        'msg_img_invalid_source': "Please select a valid Source Device.",
        'msg_img_no_target_file': "Please specify a path for the **Target Image File**.",
        'msg_img_no_target_device': "Please select a **Target Device** (Disk/Partition to be copied to).",
        'msg_img_source_target_same': "Source and Target Devices cannot be the same. This will erase your source disk!",
        'msg_img_confirm_copy': ("**WARNING: All data on the Source {source} device/partition will be copied to the Target {target} device/partition.**\n\n"
                                 "**ALL DATA ON THE TARGET DEVICE {target} WILL BE LOST!**\n\n"
                                 "Are you sure you want to start this operation? (A graphical window for authorization will open)"),
        'msg_img_no_log': "Please specify a path for the ddrescue Log File.",
        'msg_img_command_missing': "One of the required commands (pkexec, ddrescue) was not found. Please ensure they are installed.",
        'msg_img_command_failed_start': "Command execution failed to start. Are pkexec and ddrescue installed? Error: {error}",

        # Ownership Change Messages
        'msg_chown_status': "Setting Permissions: Assigning Folder Ownership to ({user})... (Wait for the second password screen.)",
        'msg_chown_missing': "Authorization tool **pkexec** was not found. Cannot change permissions.",
        'msg_chown_ok_status': "Permissions Set Successfully. Operation Complete.",
        'msg_chown_ok_info': "Ownership of the recovered files' folder ({folder_name}) has been successfully set to your normal user. You can now access your files.",
        'msg_chown_fail_status': "Permission Setting Error (Code: {code}). Folder remains owned by Root.",
        'msg_chown_fail_crit': ("Folder permissions could not be set automatically (Error Code: {code}).\n"
                                "Please open a terminal and manually run the command **pkexec chown -R $USER:$USER {target_folder}**."),
        'msg_chown_fail_start': "Permission change command could not be started. Error: {error}",
        
        # ext4 Recovery Messages
        'msg_ext4_ui_missing': "UI components could not be loaded. Program error.",
        'msg_ext4_command_missing': "One of the required commands (pkexec, ext4magic) was not found. Please ensure they are installed.",
        'msg_ext4_invalid_image': "Please select a valid Raw Image File.",
        'msg_ext4_invalid_device': "Please select a valid Source Device (Disk/Partition) for recovery.",
        'msg_ext4_mounted_warning': ("**Attention:** Recovery from a device ({source}) **MUST NOT BE MOUNTED**. Please ensure the device is unmounted before starting the operation."),
        'msg_ext4_no_target_folder': "Please select a valid Target Folder where recovered files will be saved.",
        'msg_ext4_target_folder_fail': "Target Folder ({folder}) could not be created or is inaccessible. Please check your write permissions. Error: {error}",
        'msg_ext4_start_status': "Starting Ext4 Recovery... (Target Folder: {folder_name})",
        'msg_ext4_inprogress_status': "Ext4 Recovery: Processing...",
        'msg_ext4_ok_status': "Ext4 Recovery (ext4magic) Operation Completed Successfully. Setting Permissions...",
        'msg_ext4_ok_info': "File recovery completed successfully. Recovered files are located in the **{folder_name}** directory under the Target Folder. Folder permissions will now be set.",
        'msg_ext4_fail_status': "Ext4 Recovery Operation Failed.",
        'msg_ext4_fail_crit': ("**Exit Code: {code}.**\n\n"
                               "**CRITICAL ERROR:** ext4magic command failed.\n"
                               "**MOST LIKELY CAUSES:**\n"
                               "1. **Source Mounted:** The device/image being recovered from **MUST NOT be mounted**.\n"
                               "2. **ext4magic Location:** Command path (`/usr/sbin/ext4magic`) not found.\n"
                               "3. **Superblock Error:** Even if ext4magic automatically tried backup superblocks, the file system might be severely damaged.\n\n"
                               "**Error Output (First 500 Characters):**\n"
                               "----------------------------------------\n"
                               "{snippet}..."),
        'msg_ext4_command_failed_start': "Command execution failed to start. Are pkexec and ext4magic installed? Error: {error}",

        # foremost Recovery Messages
        'msg_all_inprogress_status': "Recovery operation: In Progress...",
        'msg_all_ok_status': "Deep Recovery Operation Completed Successfully. Setting Permissions...",
        'msg_all_ok_info': "File recovery completed successfully. Recovered files should be saved inside the Target Folder (in a subfolder named **Rescuvera-Timestamp**). Folder permissions will now be set.",
        'msg_all_fail_status': "Deep Recovery Operation Failed.",
        'msg_all_fail_crit': ("**Exit Code: {code}.**\n\n"
                              "**CRITICAL WARNING:** Although Authorization (Polkit) was successful, the command itself failed immediately (Code 1).\n"
                              "**MOST LIKELY CAUSES:**\n"
                              "1. **Source Mounted:** The device being recovered from must not be mounted.\n"
                              "2. **Source Unreadable:** Source file/device not found or inaccessible.\n"
                              "3. **Command Path:** foremost command (`/usr/bin/foremost`) not found.\n\n"
                              "**Error Output:**\n"
                              "----------------------------------------\n"
                              "{error}"),
        'msg_all_invalid_image': "Please select a valid Raw Image File.",
        'msg_all_invalid_device': "Please select a valid Source Device (Disk/Partition) for recovery.",
        'msg_all_command_missing': "One of the required commands (pkexec, foremost) was not found. Please ensure they are installed.",
        'msg_readme_not_found': "ReadMe file not found:\n{pdf_path}",
        'msg_xdg_open_missing': "Could not open ReadMe file.\n'xdg-open' command not found on your system.",
        'msg_readme_open_error': "An error occurred while opening the ReadMe file: {error}",
    }
}
# --- ÇEVİRİ SÖZLÜKLERİ SONU ---

class RescuveraApp(QWidget):
    
    def __init__(self):
        super().__init__()
        self.current_lang = 'tr'  # Başlangıç dili Türkçe
        self.setWindowTitle("Rescuvera")
        
        self.setFixedSize(650, 600) 
        
        self.setWindowFlags(Qt.Window | Qt.WindowCloseButtonHint | Qt.WindowMinimizeButtonHint)
        
        icon_path = self.find_icon_path("amblem.png")
        if icon_path:
            self.setWindowIcon(QIcon(icon_path))
        
        self.input_fields = {} 
        self.progress_bar = None 
        self.progress_status_label = None 
        self.ddrescue_process = None 
        
        self.target_type = 'file' 
        self.target_combo_device = None
        
        self.loading_gif_label = None 
        
        # Kurtarma sekmelerinde hangi kaynaktan kurtarma yapılacağını tutar (image/device)
        self.rescue_source_type = {'ext4': 'image', 'all': 'image'} 
        self.rescue_process = None # extundelete/ext4magic/foremost için genel değişken
        
        self.cleanup_process = None 
        self.current_target_folder = "" 
        
        self.init_ui()
        
        if 'image_source' in self.input_fields:
             # ComboBox'ların başlangıçta doldurulması için çağırıyoruz
             self.populate_device_combobox(self.input_fields['image_source']) 

    def tr(self, key):
        """Mevcut dile göre çeviriyi döndürür."""
        return TRANSLATIONS[self.current_lang].get(key, key)
        
    def open_readme_pdf(self):
        """
        /usr/share/Rescuvera/Rescuvera Readme.pdf dosyasını 
        sistemin varsayılan okuyucusu ile açar (xdg-open kullanarak).
        """
        # Hedef dosyanın tam yolu
        pdf_path = "/usr/share/Rescuvera/Rescuvera Readme.pdf"
        
        # Dosyanın varlığını kontrol et
        if not os.path.exists(pdf_path):
            QMessageBox.critical(self, self.tr('msg_err_title'), 
                                 self.tr('msg_readme_not_found').format(pdf_path=pdf_path))
            return
        
        # xdg-open ile sistemin varsayılan PDF okuyucusunu çağırır. 
        # subprocess.Popen kullanmak, GUI'nin kilitlenmesini engeller.
        try:
            # Komutu liste olarak vermek, shell injection riskini azaltır.
            subprocess.Popen(["xdg-open", pdf_path])
            
        except FileNotFoundError:
            # xdg-open komutu bulunamazsa (çok nadir)
            QMessageBox.critical(self, self.tr('msg_err_title'), 
                                 self.tr('msg_xdg_open_missing'))
        except Exception as e:
             QMessageBox.critical(self, self.tr('msg_err_title'), 
                                 self.tr('msg_readme_open_error').format(error=e))

    def find_icon_path(self, icon_name):
        """İkon dosyasını belirli yollarda arayan yardımcı fonksiyon."""
        base_dir = os.path.dirname(os.path.abspath(__file__))
        
        paths_to_check = [
            os.path.join(base_dir, "icons", icon_name),
            os.path.join("/usr/share/Rescuvera/icons", icon_name),
            os.path.join(os.path.expanduser("~"), "Rescuvera/icons", icon_name)
        ]
        
        for path in paths_to_check:
            if os.path.exists(path):
                return path
        return None

    def populate_device_combobox(self, combo_box, exclude_device_path=None):
        """
        Sistemdeki blok aygıtlarını (disk/partisyon) lsblk komutu ile listeler ve 
        belirtilen ComboBox'ı doldurur. Hiyerarşik ve görsel bağlantılı listeleme (Basitleştirilmiş).
        """
        
        # Seçili öğeyi kaydet (varsa)
        selected_text = combo_box.currentText()
        
        combo_box.clear()
        
        try:
            # -J: JSON formatında çıktı, -o: istenen sütunlar
            result = subprocess.run(['lsblk', '-J', '-o', 'NAME,KNAME,SIZE,MODEL,TYPE,FSTYPE,MOUNTPOINT'], capture_output=True, text=True, check=True)
            
            data = json.loads(result.stdout)
            
            items = []
            
            # Yardımcı fonksiyon: cihazı ve altındaki partisyonları özyinelemeli olarak ekler
            def add_devices(devices_list, indent=""):
                for dev in devices_list:
                    dev_type = dev.get('type')
                    kname = dev.get('kname')
                    
                    if dev_type in ['disk', 'part']:
                        dev_path = f"/dev/{kname}"
                        
                        if exclude_device_path and dev_path == exclude_device_path:
                            continue
                        
                        label_parts = [f"[{dev['size']}]"] if dev.get('size') else []
                        
                        prefix = ""
                        next_indent = ""
                        
                        if dev_type == 'disk':
                            # Ana disk için girinti yok
                            next_indent = "   " # Alt öğeler için 3 boşluk
                            
                            model_name = ""
                            if dev['model']:
                                model_name = dev['model'].strip()
                                if len(model_name) > 20: 
                                    model_name = model_name[:17] + "..."
                                model_name = re.sub(r'[^\w\s\-\.]', '', model_name)
                                label_parts.append(f"Model: {model_name}")
                            
                            # Diskin kendisi için
                            item_label = f"{dev_path} {' '.join(label_parts)}"
                            items.append(item_label)

                        elif dev_type == 'part':
                            # Partisyonlar için basit ok ve indent kullan (p.py'den esinlenerek)
                            prefix = indent + "↳ " 
                            next_indent = indent + "   " # Daha derin seviyeler için 3 boşluk ekle

                            fstype = dev.get('fstype') if dev.get('fstype') else self.tr('device_lsblk_error') # Burası sabit kalsın, partisyon listesi olduğu için
                            mountpoint = dev.get('mountpoint') if dev.get('mountpoint') else self.tr('device_lsblk_error') # Burası sabit kalsın

                            label_parts.append(f"{self.tr('ext4_type_label')}: {fstype}") # Yeni çeviri anahtarı eklenebilir
                            label_parts.append(f"{self.tr('ext4_mount_label')}: {mountpoint}") # Yeni çeviri anahtarı eklenebilir

                            item_label = f"{prefix}{dev_path} {' '.join(label_parts)}"
                            items.append(item_label)
                        
                        # Alt öğeleri (partisyonları) özyinelemeli olarak eklemek için
                        if 'children' in dev and dev['children']:
                            add_devices(dev['children'], indent=next_indent)

            add_devices(data.get('blockdevices', []))

            if combo_box == self.input_fields.get('image_target_device'):
                 combo_box.addItem(self.tr('image_target_device_warning'))
                 
            if items:
                combo_box.addItems(items)
                
                # Daha önce seçili olan öğeyi geri seç (eğer hala listedeyse)
                if selected_text in combo_box.currentText(): 
                    index = combo_box.findText(selected_text)
                    if index != -1:
                        combo_box.setCurrentIndex(index)
                        
            else:
                combo_box.addItem(self.tr('device_not_found'))
                
        except (subprocess.CalledProcessError, FileNotFoundError, json.JSONDecodeError) as e:
            combo_box.addItem(self.tr('device_lsblk_error'))
            print(f"Hata: {e}")

    # Not: browse_file ve browse_directory fonksiyonları QFileDialog'u kullandığı için 
    # diyalog başlığını çevirmek yeterlidir.
    def browse_file(self, line_edit, is_save=True, filter_text="Tüm Dosyalar *", default_extension=None) -> str:
        """Gözat düğmeleri için genel dosya/dizin seçme işlevi ve otomatik uzantı ekleme."""
        options = QFileDialog.Options()
        file_path = ""
        
        dialog_title = self.tr('image_browse_file_save') if is_save else self.tr('image_browse_file_open') # image_browse_file_open yeni eklendi
        
        if is_save:
            file_path, selected_filter = QFileDialog.getSaveFileName(self, dialog_title, os.path.expanduser("~"), filter_text, options=options)
            
            if file_path and default_extension:
                if "." not in os.path.basename(file_path):
                    if default_extension in selected_filter or default_extension == ".img" or default_extension == ".log":
                        file_path += default_extension
                        
        else:
            file_path, _ = QFileDialog.getOpenFileName(self, dialog_title, os.path.expanduser("~"), filter_text, options=options)

        if file_path:
            line_edit.setText(file_path)
        return file_path
        
    def browse_directory(self, line_edit) -> str:
        """Dizin seçme işlevi."""
        options = QFileDialog.Options()
        directory_path = QFileDialog.getExistingDirectory(self, self.tr('rescue_target_folder_select'), os.path.expanduser("~"), options=options)
        
        if directory_path:
            line_edit.setText(directory_path)
        return directory_path

    def update_target_mode(self, radio_button):
        """Ham İmaj Al sekmesi için Hedef modunu (Dosya/Aygıt) ayarlar ve arayüzü günceller."""
        
        target_combo = self.input_fields.get('image_target_device')

        if radio_button.text() == self.tr('radio_file') and radio_button.isChecked():
            self.target_type = 'file'
            self.input_fields['target_stack'].setCurrentIndex(0) 
        
        elif radio_button.text() == self.tr('radio_device') and radio_button.isChecked():
            self.target_type = 'device'
            self.input_fields['target_stack'].setCurrentIndex(1) 
            
            if target_combo:
                source_text = self.input_fields['image_source'].currentText()
                # Hiyerarşik işaretleri atlatsa bile /dev/ yolunu bul
                source_match = re.search(r'(\/dev\/[a-zA-Z0-9]+)', source_text)
                exclude_path = source_match.group(1) if source_match else None
                
                self.populate_device_combobox(target_combo, exclude_device_path=exclude_path)

    def update_rescue_source_mode(self, tab_id, radio_button):
        """Kurtarma kaynağı modunu (İmaj/Aygıt) ayarlar ve arayüzü günceller."""
        
        source_stack = self.input_fields.get(f'{tab_id}_source_stack')
        source_combo = self.input_fields.get(f'{tab_id}_source_device')

        if not source_stack:
            return

        if radio_button.text() == self.tr('radio_image_rescue') and radio_button.isChecked():
            self.rescue_source_type[tab_id] = 'image'
            source_stack.setCurrentIndex(0) 
        
        elif radio_button.text() == self.tr('radio_device_rescue') and radio_button.isChecked():
            self.rescue_source_type[tab_id] = 'device'
            source_stack.setCurrentIndex(1) 
            
            if source_combo:
                # Partisyonları içeren liste doldurulur
                self.populate_device_combobox(source_combo)

    def update_progress(self):
        """ddrescue çıktısını okur ve ilerleme çubuğunu/etiketi günceller."""
        if not self.ddrescue_process:
            return
            
        output_bytes = self.ddrescue_process.readAllStandardError()
        output = str(output_bytes, 'utf-8', errors='ignore')
        
        status_info = self.tr('msg_img_start_status')

        percent_match = re.search(r'pct rescued:\s*(\d+)%', output)
        rescued_match = re.search(r'Rescued:\s*([0-9.,]+ [KMGT]B)', output)
        rate_match = re.search(r'avg rate:\s*([0-9.,]+ [KMGT]B/s)', output)
        
        if percent_match:
            if self.progress_bar.maximum() == 0:
                self.progress_bar.setRange(0, 100)
                self.progress_bar.setFormat('%p%')

            try:
                percent = int(percent_match.group(1))
                self.progress_bar.setValue(percent)
                
                # Çeviriye uygun formatlama
                rescued_val = rescued_match.group(1).replace(',', '.') if rescued_match else '0 KB'
                rate_val = rate_match.group(1).replace(',', '.') if rate_match else '0 KB/s'
                
                status_info = self.tr('msg_img_progress_status').format(
                    percent=percent, 
                    rescued=rescued_val,
                    rate=rate_val
                )
            except ValueError:
                pass
        
        self.progress_status_label.setText(status_info)


    def process_finished(self, exit_code, exit_status):
        """ddrescue işlemi bittiğinde çağrılır."""
        
        if self.loading_gif_label and self.loading_gif_label.movie():
            self.loading_gif_label.movie().stop()
            self.loading_gif_label.setVisible(False)

        error_output = str(self.ddrescue_process.readAllStandardError(), 'utf-8', errors='ignore')
        error_snippet = error_output[-500:] if len(error_output) > 500 else error_output

        self.progress_bar.setRange(0, 100)

        if exit_code == 0:
            self.progress_bar.setValue(100) 
            self.progress_status_label.setText(self.tr('msg_img_ok_title'))
            QMessageBox.information(self, self.tr('msg_img_ok_title'), 
                                    self.tr('msg_img_ok_text'))
        else:
            self.progress_bar.setValue(0) 
            self.progress_status_label.setText(self.tr('msg_img_fail_status'))
            
            error_details = self.tr('msg_img_fail_details').format(code=exit_code, snippet=error_snippet)
            
            QMessageBox.critical(self, self.tr('msg_img_fail_title'), error_details)
        
        self.ddrescue_process = None
        
    def change_ownership_and_notify(self, target_folder, tab_id):
        """Kurtarılan klasörlerin sahipliğini mevcut kullanıcıya (non-root) ayarlar. İkinci bir pkexec gerektirir."""
        
        try:
            current_user = os.getlogin() 
        except OSError:
            current_user = os.environ.get('USER', 'user') 
            
        pkexec_path = "/usr/bin/pkexec"
        
        if not os.path.isfile(pkexec_path):
             QMessageBox.critical(self, self.tr('msg_err_title'), self.tr('msg_chown_missing'))
             return
             
        command = [
            pkexec_path,  
            "/usr/bin/chown", 
            "-R",
            f"{current_user}:{current_user}", 
            target_folder 
        ]
        
        status_label = self.input_fields.get(f'{tab_id}_status_label')
        if status_label:
             status_label.setText(self.tr('msg_chown_status').format(user=current_user))

        self.cleanup_process = QProcess(self)
        
        self.cleanup_process.finished.connect(lambda exit_code, exit_status: self.cleanup_finished(exit_code, exit_status, target_folder, tab_id))
        
        try:
            self.cleanup_process.start(command[0], command[1:])
        except Exception as e:
            QMessageBox.critical(self, self.tr('msg_err_title'), self.tr('msg_chown_fail_start').format(error=e))
            self.cleanup_process = None

    def cleanup_finished(self, exit_code, exit_status, target_folder, tab_id):
        """chown işlemi bittiğinde son bildirimi gösterir."""
        
        status_label = self.input_fields.get(f'{tab_id}_status_label')
        
        if exit_code == 0:
             if status_label:
                 status_label.setText(self.tr('msg_chown_ok_status'))
             QMessageBox.information(self, self.tr('msg_chown_ok_status'), 
                                     self.tr('msg_chown_ok_info').format(folder_name=os.path.basename(target_folder)))
        else:
             error_output = str(self.cleanup_process.readAllStandardError(), 'utf-8', errors='ignore')
             if status_label:
                 status_label.setText(self.tr('msg_chown_fail_status').format(code=exit_code))
             QMessageBox.critical(self, self.tr('msg_err_title'), 
                                  self.tr('msg_chown_fail_crit').format(code=exit_code, target_folder=target_folder))
        
        self.cleanup_process = None

    def ext4_magic_process_finished(self, exit_code, exit_status):
        """ext4magic işlemi bittiğinde çağrılır. (extundelete yerine)"""
        
        progress_bar = self.input_fields.get('ext4_progress_bar')
        status_label = self.input_fields.get('ext4_status_label')
        ext4_gif_label = self.input_fields.get('ext4_gif_label')
        
        if ext4_gif_label and ext4_gif_label.movie():
            ext4_gif_label.movie().stop()
            ext4_gif_label.setVisible(False)

        if progress_bar:
            progress_bar.setRange(0, 100) 
            progress_bar.setValue(0) 

        # ext4magic çıktısı stdout'ta, hatalar stderr'da olabilir.
        error_output = str(self.rescue_process.readAllStandardError(), 'utf-8', errors='ignore')
        full_output = str(self.rescue_process.readAllStandardOutput(), 'utf-8', errors='ignore')
        
        # Eğer stderr boşsa ve exit code 0 değilse, stdout'u da kontrol et
        if not error_output and exit_code != 0:
             error_output = full_output
        
        # Hata kodu 0: Başarılı
        if exit_code == 0:
            if status_label:
                status_label.setText(self.tr('msg_ext4_ok_status'))
            QMessageBox.information(self, self.tr('msg_img_ok_title'), 
                                    self.tr('msg_ext4_ok_info').format(folder_name=os.path.basename(self.current_target_folder)))
            
            self.change_ownership_and_notify(self.current_target_folder, 'ext4')
        else:
            if status_label:
                status_label.setText(self.tr('msg_ext4_fail_status'))
            
            error_details = self.tr('msg_ext4_fail_crit').format(code=exit_code, snippet=error_output[:500])
            
            QMessageBox.critical(self, self.tr('msg_err_title'), error_details)
        
        self.rescue_process = None

    def foremost_process_finished(self, exit_code, exit_status):
        """foremost işlemi bittiğinde çağrılır."""
        
        progress_bar = self.input_fields.get('all_progress_bar')
        status_label = self.input_fields.get('all_status_label')
        all_gif_label = self.input_fields.get('all_gif_label')

        if all_gif_label and all_gif_label.movie():
            all_gif_label.movie().stop()
            all_gif_label.setVisible(False)

        if progress_bar:
            progress_bar.setRange(0, 100) 
            progress_bar.setValue(0) 

        error_output = str(self.rescue_process.readAllStandardError(), 'utf-8', errors='ignore')
        if not error_output:
             error_output = str(self.rescue_process.readAllStandardOutput(), 'utf-8', errors='ignore')

        if exit_code == 0:
            if status_label:
                status_label.setText(self.tr('msg_all_ok_status'))
            
            QMessageBox.information(self, self.tr('msg_img_ok_title'), 
                                    self.tr('msg_all_ok_info'))

            final_target_folder = self.current_target_folder 

            self.change_ownership_and_notify(final_target_folder, 'all')
        else:
            if status_label:
                status_label.setText(self.tr('msg_all_fail_status'))
            
            error_details = self.tr('msg_all_fail_crit').format(code=exit_code, error=error_output)
            
            QMessageBox.critical(self, self.tr('msg_err_title'), error_details)
        
        self.rescue_process = None

    def start_ext4_rescue(self):
        """ext4magic komutunu QProcess ile pkexec yetkisiyle çalıştıran işlev. (EXT4MAGIC ENTEGRASYONU)"""
        
        source = ""
        target_folder_root = self.input_fields['ext4_target_folder'].text() 
        
        progress_bar = self.input_fields.get('ext4_progress_bar')
        status_label = self.input_fields.get('ext4_status_label')
        ext4_gif_label = self.input_fields.get('ext4_gif_label')
        
        if not progress_bar or not status_label:
            QMessageBox.critical(self, self.tr('msg_err_title'), self.tr('msg_ext4_ui_missing'))
            return
            
        pkexec_path = "/usr/bin/pkexec"
        ext4magic_path = "/usr/sbin/ext4magic" 
        
        if not os.path.isfile(pkexec_path) or not os.path.isfile(ext4magic_path):
             QMessageBox.critical(self, self.tr('msg_err_title'), self.tr('msg_ext4_command_missing'))
             return
            
        if self.rescue_source_type['ext4'] == 'image':
            source = self.input_fields['ext4_source_file'].text()
            if not source or not os.path.isfile(source):
                QMessageBox.warning(self, self.tr('msg_warn_title'), self.tr('msg_ext4_invalid_image'))
                return
            
        elif self.rescue_source_type['ext4'] == 'device':
            source_text = self.input_fields['ext4_source_device'].currentText()
            # Hiyerarşik işaretleri atlatsa bile /dev/ yolunu bul
            source_match = re.search(r'(\/dev\/[a-zA-Z0-9]+)', source_text)
            
            if not source_match or self.tr('device_lsblk_error') in source_text:
                 QMessageBox.warning(self, self.tr('msg_warn_title'), self.tr('msg_ext4_invalid_device'))
                 return
            source = source_match.group(1) # Partisyon veya disk yolu
            
            QMessageBox.information(self, self.tr('msg_info_title'), 
                self.tr('msg_ext4_mounted_warning').format(source=source))

        if not target_folder_root:
            QMessageBox.warning(self, self.tr('msg_warn_title'), self.tr('msg_ext4_no_target_folder'))
            return
            
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d-%H-%M-%S")
        final_output_path = os.path.join(target_folder_root, f"Rescuvera-{timestamp}")
            
        try:
            if not os.path.exists(target_folder_root):
                os.makedirs(target_folder_root, exist_ok=True)
        except OSError as e:
            QMessageBox.critical(self, self.tr('msg_err_title'), self.tr('msg_ext4_target_folder_fail').format(folder=target_folder_root, error=e))
            return
            
        self.current_target_folder = final_output_path 

        # --- EXT4MAGIC KOMUTU BAŞLATILIYOR ---

        command = [
            pkexec_path,  
            ext4magic_path, 
            source,      
            "-d", 
            final_output_path, # Kurtarılan dosyaların yazılacağı klasör
            "-r" # Tamamen geri yükle (recursive)
        ]

        self.rescue_process = QProcess(self)
        self.rescue_process.readyReadStandardError.connect(lambda: status_label.setText(self.tr('msg_ext4_inprogress_status')))
        self.rescue_process.finished.connect(self.ext4_magic_process_finished) 
        
        progress_bar.setRange(0, 0) # Belirsiz ilerleme
        status_label.setText(self.tr('msg_ext4_start_status').format(folder_name=os.path.basename(final_output_path)))

        gif_path = self.find_icon_path("dosyakurtariliyor.gif") 
        
        if gif_path and ext4_gif_label:
            movie = QMovie(gif_path)
            if movie.isValid():
                ext4_gif_label.setMovie(movie)
                movie.start()
                ext4_gif_label.setVisible(True)
            else:
                 ext4_gif_label.setVisible(False)
        
        try:
            self.rescue_process.start(command[0], command[1:])
            
        except Exception as e:
            if ext4_gif_label:
                ext4_gif_label.setVisible(False)
                
            QMessageBox.critical(self.tr('msg_err_title'), f"Hata", self.tr('msg_ext4_command_failed_start').format(error=e))
            self.rescue_process = None
            progress_bar.setRange(0, 100)


    def start_foremost_rescue(self):
        """foremost komutunu QProcess ile pkexec yetkisiyle çalıştıran işlev."""
        
        source = ""
        target_folder = self.input_fields['all_target_folder'].text() 
        
        progress_bar = self.input_fields.get('all_progress_bar')
        status_label = self.input_fields.get('all_status_label')
        all_gif_label = self.input_fields.get('all_gif_label')
        
        pkexec_path = "/usr/bin/pkexec"
        foremost_path = "/usr/bin/foremost"
        
        if not os.path.isfile(pkexec_path) or not os.path.isfile(foremost_path):
             QMessageBox.critical(self, self.tr('msg_err_title'), self.tr('msg_all_command_missing'))
             return
            
        if self.rescue_source_type['all'] == 'image':
            source = self.input_fields['all_source_file'].text()
            if not source or not os.path.isfile(source):
                QMessageBox.warning(self, self.tr('msg_warn_title'), self.tr('msg_all_invalid_image'))
                return
            
        elif self.rescue_source_type['all'] == 'device':
            source_text = self.input_fields['all_source_device'].currentText()
            # Hiyerarşik işaretleri atlatsa bile /dev/ yolunu bul
            source_match = re.search(r'(\/dev\/[a-zA-Z0-9]+)', source_text)
            
            if not source_match or self.tr('device_lsblk_error') in source_text:
                 QMessageBox.warning(self, self.tr('msg_warn_title'), self.tr('msg_all_invalid_device'))
                 return
            source = source_match.group(1) # Partisyon veya disk yolu
            
            QMessageBox.information(self, self.tr('msg_info_title'), 
                self.tr('msg_ext4_mounted_warning').format(source=source)) # foremost için de aynı uyarı geçerli

        if not target_folder:
            QMessageBox.warning(self, self.tr('msg_warn_title'), self.tr('msg_ext4_no_target_folder'))
            return

        timestamp = datetime.datetime.now().strftime("%Y-%m-%d-%H-%M-%S")
        final_output_path = os.path.join(target_folder, f"Rescuvera-{timestamp}")
        
        try:
            if not os.path.exists(target_folder):
                os.makedirs(target_folder, exist_ok=True)
        except OSError as e:
            QMessageBox.critical(self, self.tr('msg_err_title'), self.tr('msg_ext4_target_folder_fail').format(folder=target_folder, error=e))
            return
            
        self.current_target_folder = final_output_path 
            
        command = [
            pkexec_path,  
            foremost_path, 
            "-v",
            "-i",
            source,      
            "-o", 
            final_output_path, 
        ]

        self.rescue_process = QProcess(self)
        self.rescue_process.readyReadStandardError.connect(lambda: status_label.setText(self.tr('msg_all_inprogress_status')))
        self.rescue_process.finished.connect(self.foremost_process_finished) 
        
        progress_bar.setRange(0, 0)
        status_label.setText(self.tr('msg_all_inprogress_status'))

        gif_path = self.find_icon_path("dosyakurtariliyor.gif") 
        
        if gif_path and all_gif_label:
            movie = QMovie(gif_path)
            if movie.isValid():
                all_gif_label.setMovie(movie)
                movie.start()
                all_gif_label.setVisible(True)
            else:
                 all_gif_label.setVisible(False)
        
        try:
            self.rescue_process.start(command[0], command[1:])
            
        except Exception as e:
            if all_gif_label:
                all_gif_label.setVisible(False)
                
            QMessageBox.critical(self, self.tr('msg_err_title'), self.tr('msg_img_command_failed_start').format(error=e))
            self.rescue_process = None
            progress_bar.setRange(0, 100)


    def start_image_copy(self):
        """ddrescue komutunu QProcess ile pkexec yetkisiyle çalıştıran işlev."""
        
        source_text = self.input_fields['image_source'].currentText() 
        # /dev/ yolunu bul
        source_match = re.search(r'(\/dev\/[a-zA-Z0-9]+)', source_text)
        
        if not source_match or self.tr('device_lsblk_error') in source_text:
             QMessageBox.warning(self, self.tr('msg_warn_title'), self.tr('msg_img_invalid_source'))
             return
             
        source = source_match.group(1) # Seçilen disk veya partisyon yolu
        logfile = self.input_fields['image_logfile'].text()      
        target = ""
        
        if self.target_type == 'file':
            target = self.input_fields['image_target_file'].text()    
            if not target:
                QMessageBox.warning(self, self.tr('msg_warn_title'), self.tr('msg_img_no_target_file'))
                return
            
        elif self.target_type == 'device':
            target_text = self.input_fields['image_target_device'].currentText()
            # Hedef aygıt yolunu bul
            target_match = re.search(r'(\/dev\/[a-zA-Z0-9]+)', target_text)
            
            if self.tr('image_target_device_warning') in target_text or self.tr('device_not_found') in target_text or not target_match:
                 QMessageBox.warning(self, self.tr('msg_warn_title'), self.tr('msg_img_no_target_device'))
                 return
            target = target_match.group(1) # Seçilen hedef disk veya partisyon yolu
            
            if source == target:
                 QMessageBox.critical(self, self.tr('msg_err_title'), self.tr('msg_img_source_target_same'))
                 return
            
            reply = QMessageBox.question(self, self.tr('msg_crit_title'), 
                self.tr('msg_img_confirm_copy').format(source=source, target=target),
                QMessageBox.Yes | QMessageBox.No, QMessageBox.No)
            
            if reply == QMessageBox.No:
                return


        if not source or not source.startswith('/dev/'):
            QMessageBox.warning(self, self.tr('msg_warn_title'), self.tr('msg_img_invalid_source'))
            return
        if not logfile:
            QMessageBox.warning(self, self.tr('msg_warn_title'), self.tr('msg_img_no_log'))
            return
            
        pkexec_path = "/usr/bin/pkexec"
        ddrescue_path = "/usr/bin/ddrescue"
        
        if not os.path.isfile(pkexec_path) or not os.path.isfile(ddrescue_path):
             QMessageBox.critical(self, self.tr('msg_err_title'), self.tr('msg_img_command_missing'))
             return

        command = [
            pkexec_path,  
            ddrescue_path, 
            "-d",      
            "-f",      
            "-v",      
            source,      
            target, 
            logfile      
        ]

        self.ddrescue_process = QProcess(self)
        self.ddrescue_process.readyReadStandardError.connect(self.update_progress) 
        self.ddrescue_process.finished.connect(self.process_finished) 
        
        self.progress_bar.setRange(0, 0) 
        self.progress_status_label.setText(self.tr('msg_img_start_status'))

        gif_path = self.find_icon_path("diskklonlaniyor.gif") 
        if gif_path and self.loading_gif_label:
            movie = QMovie(gif_path)
            if movie.isValid():
                self.loading_gif_label.setMovie(movie)
                movie.start()
                self.loading_gif_label.setVisible(True)
            else:
                 self.loading_gif_label.setVisible(False)
        
        try:
            self.ddrescue_process.start(command[0], command[1:])
            
        except Exception as e:
            if self.loading_gif_label:
                self.loading_gif_label.setVisible(False)
                
            QMessageBox.critical(self, self.tr('msg_err_title'), self.tr('msg_img_command_failed_start').format(error=e))


    def create_image_tab_content(self, tab_widget):
        """Ham İmaj Al sekmesinin içeriğini oluşturan yardımcı fonksiyon."""
        tab_layout = QVBoxLayout(tab_widget)
        tab_layout.setContentsMargins(15, 15, 15, 15)
        
        title_label = QLabel(self.tr('image_title')) # Çevrildi
        title_label.setStyleSheet("font-weight: bold; margin-bottom: 5px;") 
        tab_layout.addWidget(title_label)
        
        image_layout = QGridLayout()
        image_layout.setContentsMargins(0, 0, 0, 0)
        image_layout.setHorizontalSpacing(10) 
        image_layout.setVerticalSpacing(10)
        
        label1 = QLabel(self.tr('image_source_label')) # Çevrildi
        combo_box_source = QComboBox() 
        combo_box_source.addItem(self.tr('device_scan_wait')) # Çevrildi
        
        image_layout.addWidget(label1, 0, 0, 1, 1, Qt.AlignRight)
        image_layout.addWidget(combo_box_source, 0, 1, 1, 2)
        
        self.input_fields['image_source'] = combo_box_source

        label_target_type = QLabel(self.tr('image_target_type_label')) # Çevrildi
        
        radio_file = QRadioButton(self.tr('radio_file')) # Çevrildi
        radio_device = QRadioButton(self.tr('radio_device')) # Çevrildi
        radio_file.setChecked(True)

        radio_layout = QHBoxLayout()
        radio_layout.addWidget(radio_file)
        radio_layout.addWidget(radio_device)
        radio_layout.addStretch()

        image_layout.addWidget(label_target_type, 1, 0, 1, 1, Qt.AlignRight)
        image_layout.addLayout(radio_layout, 1, 1, 1, 2)
        
        # Sadece metin değil, buton nesnesi baz alınacağı için connect bozulmaz
        radio_file.toggled.connect(lambda: self.update_target_mode(radio_file))
        radio_device.toggled.connect(lambda: self.update_target_mode(radio_device))
        
        label_target = QLabel(self.tr('image_target_label')) # Çevrildi
        
        target_stack = QStackedWidget()
        self.input_fields['target_stack'] = target_stack

        file_widget = QWidget()
        file_layout = QHBoxLayout(file_widget)
        file_layout.setContentsMargins(0, 0, 0, 0) 
        line_edit_file = QLineEdit()
        browse_button_file = QPushButton(self.tr('button_browse')) # Çevrildi
        # Gözat düğmesinin filtre metni çevrildi, diyalog başlığı browse_file içinde çevrildi
        browse_button_file.clicked.connect(lambda: self.browse_file(line_edit_file, is_save=True, filter_text=self.tr('image_file_filter'), default_extension=".img"))
        file_layout.addWidget(line_edit_file)
        file_layout.addWidget(browse_button_file)
        target_stack.addWidget(file_widget)
        self.input_fields['image_target_file'] = line_edit_file

        device_widget = QWidget()
        device_layout = QHBoxLayout(device_widget)
        device_layout.setContentsMargins(0, 0, 0, 0)
        
        combo_box_target_device = QComboBox() 
        combo_box_target_device.addItem(self.tr('image_target_device_warning')) # Çevrildi
        combo_box_target_device.setSizePolicy(QSizePolicy.Expanding, QSizePolicy.Preferred)
        
        device_layout.addWidget(combo_box_target_device)
        target_stack.addWidget(device_widget)
        self.input_fields['image_target_device'] = combo_box_target_device 

        image_layout.addWidget(label_target, 2, 0, 1, 1, Qt.AlignRight)
        image_layout.addWidget(target_stack, 2, 1, 1, 2)

        label_log = QLabel(self.tr('image_log_label')) # Çevrildi
        line_edit_log = QLineEdit()
        browse_button_log = QPushButton(self.tr('button_browse')) # Çevrildi
        # Gözat düğmesinin filtre metni çevrildi
        browse_button_log.clicked.connect(lambda: self.browse_file(line_edit_log, is_save=True, filter_text=self.tr('image_log_filter'), default_extension=".log"))

        image_layout.addWidget(label_log, 3, 0, 1, 1, Qt.AlignRight)
        image_layout.addWidget(line_edit_log, 3, 1)
        image_layout.addWidget(browse_button_log, 3, 2)

        self.input_fields['image_logfile'] = line_edit_log
        
        start_button = QPushButton(self.tr('button_start_image')) # Çevrildi
        start_button.setObjectName("button_start_image") # Dil değiştirme için nesne adı ekle
        start_button.clicked.connect(self.start_image_copy)
        image_layout.addWidget(start_button, 4, 2, 1, 1) 
        
        self.progress_status_label = QLabel(self.tr('status_waiting')) # Çevrildi
        self.progress_status_label.setStyleSheet("font-weight: bold; margin-top: 10px; border: none;") 
        image_layout.addWidget(self.progress_status_label, 5, 0, 1, 3)
        
        self.progress_bar = QProgressBar()
        self.progress_bar.setAlignment(Qt.AlignCenter) 
        self.progress_bar.setFormat('%p%')
        self.progress_bar.setValue(0)
        
        image_layout.addWidget(self.progress_bar, 6, 0, 1, 3)

        self.loading_gif_label = QLabel()
        self.loading_gif_label.setAlignment(Qt.AlignCenter)
        self.loading_gif_label.setFixedSize(180, 92) 
        self.loading_gif_label.setVisible(False) 

        gif_container_layout = QHBoxLayout()
        gif_container_layout.addStretch()
        gif_container_layout.addWidget(self.loading_gif_label)
        gif_container_layout.addStretch()

        image_layout.addLayout(gif_container_layout, 7, 0, 1, 3)
        
        image_container = QWidget()
        image_container.setLayout(image_layout)
        tab_layout.addWidget(image_container)
        tab_layout.addStretch(1)

        # Dil değiştirme için tüm UI öğelerini kaydet
        self.input_fields['image_ui_elements'] = {
            'title_label': title_label,
            'label1': label1,
            'radio_file': radio_file,
            'radio_device': radio_device,
            'label_target_type': label_target_type,
            'label_target': label_target,
            'label_log': label_log,
            'browse_button_file': browse_button_file,
            'browse_button_log': browse_button_log,
            'start_button': start_button,
            'combo_box_source': combo_box_source,
            'combo_box_target_device': combo_box_target_device,
        }

    def create_rescue_tab_content(self, tab_widget, tab_id):
        """Dosya Kurtar sekmelerinin içeriğini oluşturan yardımcı fonksiyon. Durum bilgileri sekmeye özgü yapıldı."""
        
        tab_layout = QVBoxLayout(tab_widget)
        tab_layout.setContentsMargins(15, 15, 15, 15)
        
        selection_title = QLabel(self.tr('rescue_selection_title')) # Çevrildi
        selection_title.setStyleSheet("font-weight: bold; margin-bottom: 5px; border: none;")
        tab_layout.addWidget(selection_title)
        
        selection_content_widget = QWidget()
        selection_layout = QHBoxLayout(selection_content_widget)
        selection_layout.setContentsMargins(0, 0, 0, 0)
        
        radio_button1 = QRadioButton(self.tr('radio_image_rescue')) # Çevrildi
        radio_button2 = QRadioButton(self.tr('radio_device_rescue')) # Çevrildi
        radio_button1.setChecked(True)
        
        selection_layout.addWidget(radio_button1)
        selection_layout.addWidget(radio_button2)
        selection_layout.addStretch()

        tab_layout.addWidget(selection_content_widget)
        
        # Sadece metin değil, buton nesnesi baz alınacağı için connect bozulmaz
        radio_button1.toggled.connect(lambda: self.update_rescue_source_mode(tab_id, radio_button1))
        radio_button2.toggled.connect(lambda: self.update_rescue_source_mode(tab_id, radio_button2))

        file_title = QLabel(self.tr('rescue_file_title')) # Çevrildi
        file_title.setStyleSheet("font-weight: bold; margin-bottom: 5px; margin-top: 15px;") 
        tab_layout.addWidget(file_title)

        file_container = QWidget()
        file_layout = QGridLayout(file_container)
        file_layout.setContentsMargins(0, 0, 0, 0)
        file_layout.setHorizontalSpacing(10)
        file_layout.setVerticalSpacing(10)

        
        label3 = QLabel(self.tr('rescue_from')) # Çevrildi
        label3.setStyleSheet("border: none;")

        source_stack = QStackedWidget()
        self.input_fields[f'{tab_id}_source_stack'] = source_stack

        file_widget = QWidget()
        file_layout_h = QHBoxLayout(file_widget)
        file_layout_h.setContentsMargins(0, 0, 0, 0)
        line_edit_file = QLineEdit()
        browse_button_file = QPushButton(self.tr('button_browse')) # Çevrildi
        # Gözat düğmesinin filtre metni image_file_filter ile aynı, diyalog başlığı browse_file içinde çevrildi
        browse_button_file.clicked.connect(lambda: self.browse_file(line_edit_file, is_save=False, filter_text=self.tr('image_file_filter'), default_extension=None))
        file_layout_h.addWidget(line_edit_file)
        file_layout_h.addWidget(browse_button_file)
        source_stack.addWidget(file_widget)
        self.input_fields[f'{tab_id}_source_file'] = line_edit_file

        device_widget = QWidget()
        device_layout_h = QHBoxLayout(device_widget)
        device_layout_h.setContentsMargins(0, 0, 0, 0)
        
        combo_box_source_device = QComboBox() 
        combo_box_source_device.addItem(self.tr('device_scan_wait')) # Çevrildi
        device_layout_h.addWidget(combo_box_source_device)
        source_stack.addWidget(device_widget)
        self.input_fields[f'{tab_id}_source_device'] = combo_box_source_device
        
        file_layout.addWidget(label3, 0, 0, 1, 1, Qt.AlignRight)
        file_layout.addWidget(source_stack, 0, 1, 1, 2)
        
        label4 = QLabel(self.tr('rescue_to')) # Çevrildi
        label4.setStyleSheet("border: none;")
        line_edit4 = QLineEdit()
        browse_button4 = QPushButton(self.tr('button_browse')) # Çevrildi
        # Diyalog başlığı browse_directory içinde çevrildi
        browse_button4.clicked.connect(lambda: self.browse_directory(line_edit4)) 
        
        file_layout.addWidget(label4, 1, 0, 1, 1, Qt.AlignRight)
        file_layout.addWidget(line_edit4, 1, 1)
        file_layout.addWidget(browse_button4, 1, 2)
        
        self.input_fields[f'{tab_id}_target_folder'] = line_edit4

        start_button = QPushButton(self.tr('button_rescue')) # Çevrildi
        start_button.setObjectName(f"button_rescue_{tab_id}") # Dil değiştirme için nesne adı ekle
        
        if tab_id == 'ext4':
            start_button.clicked.connect(self.start_ext4_rescue)
        elif tab_id == 'all':
            start_button.clicked.connect(self.start_foremost_rescue)
            
        file_layout.addWidget(start_button, 2, 2, 1, 1)
        
        tab_layout.addWidget(file_container)
        
        status_label = QLabel(self.tr('status_waiting')) # Çevrildi
        status_label.setStyleSheet("font-weight: bold; margin-top: 10px; border: none;") 
        tab_layout.addWidget(status_label)
        self.input_fields[f'{tab_id}_status_label'] = status_label 
        
        progress_bar = QProgressBar()
        progress_bar.setAlignment(Qt.AlignCenter) 
        progress_bar.setFormat('%p%')
        progress_bar.setValue(0)
        tab_layout.addWidget(progress_bar)
        self.input_fields[f'{tab_id}_progress_bar'] = progress_bar 

        gif_label = QLabel()
        gif_label.setAlignment(Qt.AlignCenter)
        gif_label.setFixedSize(113, 128) 
        gif_label.setVisible(False)
        self.input_fields[f'{tab_id}_gif_label'] = gif_label 
        
        gif_container_layout = QHBoxLayout()
        gif_container_layout.addStretch()
        gif_container_layout.addWidget(gif_label)
        gif_container_layout.addStretch()

        tab_layout.addLayout(gif_container_layout)
        
        tab_layout.addStretch(1)

        # Dil değiştirme için tüm UI öğelerini kaydet
        self.input_fields[f'{tab_id}_ui_elements'] = {
            'selection_title': selection_title,
            'radio_button1': radio_button1,
            'radio_button2': radio_button2,
            'file_title': file_title,
            'label3': label3,
            'label4': label4,
            'browse_button_file': browse_button_file,
            'browse_button4': browse_button4,
            'start_button': start_button,
            'combo_box_source_device': combo_box_source_device,
        }

    def init_ui(self):
        main_layout = QVBoxLayout(self)
        
        header_container = QWidget()
        header_layout = QHBoxLayout(header_container)
        header_layout.setAlignment(Qt.AlignLeft)
        header_layout.setContentsMargins(7, 7, 7, 7)
        
        icon_path = self.find_icon_path("amblem.png")
        if icon_path:
            icon_label = QLabel()
            pixmap = QPixmap(icon_path)
            scaled_pixmap = pixmap.scaledToHeight(60, Qt.SmoothTransformation)
            icon_label.setPixmap(scaled_pixmap)
            icon_label.setFixedSize(scaled_pixmap.size())
            icon_label.setStyleSheet("border: none;") 
            header_layout.addWidget(icon_label, alignment=Qt.AlignLeft)
        
        self.title_label = QLabel(self.tr('title_main')) # Çevrildi
        self.title_label.setStyleSheet("""
            background-color: transparent; 
            border: none;
            font-weight: bold;
            font-size: 24px;
        """)
        
        header_layout.addStretch()
        header_layout.addWidget(self.title_label, alignment=Qt.AlignCenter)
        header_layout.addStretch()

        header_container.setStyleSheet("""
            background-color: #96bc90; 
            border: 1px solid #a0a0a0; 
            border-radius: 0px; 
        """)
        
        main_layout.addWidget(header_container)

        info_layout = QHBoxLayout()
        info_layout.addStretch(1)
        self.info_label = QLabel(self.tr('info_label')) # Çevrildi
        self.info_button = QPushButton(self.tr('info_button')) # Çevrildi
        info_layout.addWidget(self.info_label)
        info_layout.addWidget(self.info_button)
        main_layout.addLayout(info_layout)
        
        self.info_button.clicked.connect(self.open_readme_pdf)
        
        self.tab_widget = QTabWidget()
        
        tab_image = QWidget()
        # self.tab_widget.addTab metni aşağıda update_ui içinde çevrilecek
        self.tab_widget.addTab(tab_image, self.tr('tab_image')) 
        self.create_image_tab_content(tab_image)
        
        tab_ext4 = QWidget()
        tab_all = QWidget()
        
        self.tab_widget.addTab(tab_ext4, self.tr('tab_ext4')) 
        self.tab_widget.addTab(tab_all, self.tr('tab_all')) 
        
        self.create_rescue_tab_content(tab_ext4, 'ext4') 
        self.create_rescue_tab_content(tab_all, 'all') 
        
        main_layout.addWidget(self.tab_widget)
        
        bottom_buttons_layout = QHBoxLayout()
        bottom_buttons_layout.addStretch()
        
        # Language butonu çeviri kapsamı dışında olduğu için metni değiştirilmiyor.
        self.language_button = QPushButton("Language") 
        self.language_button.clicked.connect(self.toggle_language) # Yeni işlev bağlandı
        bottom_buttons_layout.addWidget(self.language_button)
        
        self.about_button = QPushButton(self.tr('about_button')) # Çevrildi
        bottom_buttons_layout.addWidget(self.about_button)
        main_layout.addLayout(bottom_buttons_layout)

        self.about_button.clicked.connect(self.show_about_dialog)

    def toggle_language(self):
        """Dili değiştirir ve arayüzü günceller."""
        if self.current_lang == 'tr':
            self.current_lang = 'en'
        else:
            self.current_lang = 'tr'

        self.update_ui_texts()
        
    def update_ui_texts(self):
        """Tüm arayüz metinlerini mevcut dile göre günceller."""
        
        # 1. Ana Pencereler ve Sekmeler
        self.title_label.setText(self.tr('title_main'))
        self.info_label.setText(self.tr('info_label'))
        self.info_button.setText(self.tr('info_button'))
        self.about_button.setText(self.tr('about_button'))
        
        self.tab_widget.setTabText(0, self.tr('tab_image'))
        self.tab_widget.setTabText(1, self.tr('tab_ext4'))
        self.tab_widget.setTabText(2, self.tr('tab_all'))
        
        # 2. Ham İmaj Al Sekmesi
        image_elements = self.input_fields['image_ui_elements']
        image_elements['title_label'].setText(self.tr('image_title'))
        image_elements['label1'].setText(self.tr('image_source_label'))
        image_elements['label_target_type'].setText(self.tr('image_target_type_label'))
        image_elements['radio_file'].setText(self.tr('radio_file'))
        image_elements['radio_device'].setText(self.tr('radio_device'))
        image_elements['label_target'].setText(self.tr('image_target_label'))
        image_elements['label_log'].setText(self.tr('image_log_label'))
        image_elements['browse_button_file'].setText(self.tr('button_browse'))
        image_elements['browse_button_log'].setText(self.tr('button_browse'))
        image_elements['start_button'].setText(self.tr('button_start_image'))
        self.progress_status_label.setText(self.tr('status_waiting'))
        
        # ComboBox'ları yeniden doldur
        self.populate_device_combobox(image_elements['combo_box_source'])
        
        # Sadece hedef cihaz seçili modu aktifse hedef combobox'ı da güncelle
        if self.target_type == 'device':
            # Kaynak yolu hariç tutulmazsa hata olabilir, o yüzden sadece metni güncelleyip tekrar dolduruyoruz.
            self.populate_device_combobox(image_elements['combo_box_target_device'])
        else:
             # Eğer Dosya modu aktifse sadece ilk öğeyi güncelle
             if image_elements['combo_box_target_device'].count() > 0:
                 image_elements['combo_box_target_device'].setItemText(0, self.tr('image_target_device_warning'))
        
        # 3. Kurtarma Sekmeleri
        for tab_id in ['ext4', 'all']:
            rescue_elements = self.input_fields[f'{tab_id}_ui_elements']
            
            rescue_elements['selection_title'].setText(self.tr('rescue_selection_title'))
            rescue_elements['radio_button1'].setText(self.tr('radio_image_rescue'))
            rescue_elements['radio_button2'].setText(self.tr('radio_device_rescue'))
            rescue_elements['file_title'].setText(self.tr('rescue_file_title'))
            rescue_elements['label3'].setText(self.tr('rescue_from'))
            rescue_elements['label4'].setText(self.tr('rescue_to'))
            rescue_elements['browse_button_file'].setText(self.tr('button_browse'))
            rescue_elements['browse_button4'].setText(self.tr('button_browse'))
            rescue_elements['start_button'].setText(self.tr('button_rescue'))
            
            self.input_fields[f'{tab_id}_status_label'].setText(self.tr('status_waiting'))
            
            # Eğer Aygıt modu aktifse combobox'ı güncelle
            if self.rescue_source_type[tab_id] == 'device':
                 self.populate_device_combobox(rescue_elements['combo_box_source_device'])
            else:
                 # Eğer İmaj modu aktifse sadece ilk öğeyi güncelle (tarama bekleniyor)
                 if rescue_elements['combo_box_source_device'].count() > 0:
                     rescue_elements['combo_box_source_device'].setItemText(0, self.tr('device_scan_wait'))
        
    # browse_file ve browse_directory fonksiyonları QFileDialog'u kullandığı için 
    # diyalog başlığını çevirmek yeterlidir.
    def browse_file(self, line_edit, is_save=True, filter_text="Tüm Dosyalar *", default_extension=None) -> str:
        """Gözat düğmeleri için genel dosya/dizin seçme işlevi ve otomatik uzantı ekleme."""
        options = QFileDialog.Options()
        file_path = ""
        
        dialog_title = self.tr('image_browse_file_save') if is_save else self.tr('image_browse_file_open')
        
        if is_save:
            file_path, selected_filter = QFileDialog.getSaveFileName(self, dialog_title, os.path.expanduser("~"), filter_text, options=options)
            
            if file_path and default_extension:
                if "." not in os.path.basename(file_path):
                    if default_extension in selected_filter or default_extension == ".img" or default_extension == ".log":
                        file_path += default_extension
                        
        else:
            file_path, _ = QFileDialog.getOpenFileName(self, dialog_title, os.path.expanduser("~"), filter_text, options=options)

        if file_path:
            line_edit.setText(file_path)
        return file_path

    def show_about_dialog(self):
        """Hakkında penceresini oluşturan ve gösteren metot."""
        about_dialog = QDialog(self)
        about_dialog.setWindowTitle(self.tr('about_dialog_title')) # Çevrildi
        about_dialog.setFixedSize(380, 480) 

        dialog_layout = QVBoxLayout(about_dialog)
        dialog_layout.setAlignment(Qt.AlignCenter)
        
        icon_path = self.find_icon_path("amblem.png")
        if icon_path:
            logo_label = QLabel()
            pixmap = QPixmap(icon_path)
            scaled_pixmap = pixmap.scaledToHeight(70, Qt.SmoothTransformation)
            logo_label.setPixmap(scaled_pixmap)
            logo_label.setAlignment(Qt.AlignCenter)
            dialog_layout.addWidget(logo_label)
        
        info_text = (
            "<h2 style='text-align: center;'>Rescuvera</h2>"
            f"<h3 style='text-align: center;'>{self.tr('title_main')}</h3><br>" # Çevrildi
            f"<b>{self.tr('about_version')}</b> 1.0.3 (ext4magic)<br>" # Çevrildi
            f"<b>{self.tr('about_license')}</b> GNU GPLv3<br>" # Çevrildi
            f"<b>{self.tr('about_lang')}</b> Python3<br>" # Çevrildi
            f"<b>{self.tr('about_ui')}</b> Qt5 (PyQt5)<br>" # Çevrildi
            f"<b>{self.tr('about_dev')}</b> Aydın Serhat KILIÇOĞLU<br><br>" # Çevrildi
            f"{self.tr('about_text')}" # Çevrildi
        )
        info_label = QLabel(info_text)
        info_label.setWordWrap(True)
        dialog_layout.addWidget(info_label)
        
        dialog_layout.addStretch()
        
        close_button = QPushButton(self.tr('about_close')) # Çevrildi
        close_button.clicked.connect(about_dialog.close)
        dialog_layout.addWidget(close_button)

        about_dialog.exec_()


if __name__ == "__main__":
    app = QApplication(sys.argv)
    # İlk dilin 'tr' olması varsayıldığı için QLocale ayarı Türkçe olarak bırakıldı
    locale = QLocale(QLocale.Turkish, QLocale.Turkey)
    QLocale.setDefault(locale)
    
    window = RescuveraApp()
    window.show()
    sys.exit(app.exec_())
