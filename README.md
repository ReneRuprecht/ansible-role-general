# General

Eine Ansible-Basisrolle zur Ersteinrichtung von Debian-Systemen.
Sie verwaltet zentrale Aspekte wie Benutzer, SSH, Firewall (UFW), Zeitzonen, Logging und grundlegende Sicherheits-Features.

## Unterstützte Betriebssysteme

Die Rolle ist primär für Debian getestet. 

* **Debian:** 12 (Bookworm), 13 (Trixie)

## Voraussetzungen (Prerequisites)

Damit die Rolle fehlerfrei durchläuft, prüft sie beim Start mehrere Pflichtvariablen (via `ansible.builtin.assert`). Folgende Bedingungen müssen erfüllt sein:

### Erforderliche Variablen-Konfigurationen

1. **Admin SSH-Key:** 
   * **Bedingung:** `general_admin_ssh_key` darf nicht leer sein.
   * **Ausnahme:** Kann übersprungen werden, wenn `general_ignore_ssh_key_deployment: true` gesetzt ist.

2. **Root-Passwort:**
   * **Bedingung:** Wenn `general_root_setup_enabled: true` aktiv ist, muss `general_root_password` zwingend definiert und befüllt sein.

3. **Netzwerk-Konfiguration:**
   * **Bedingung:** Für jeden Eintrag in der Liste `general_networking` muss zwingend eine gültige IP-Adresse unter `ip_address` angegeben sein (darf nicht leer sein).

## Rollen-Variablen

Die folgenden Variablen können angepasst werden.


### Allgemein & Hostname

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_environment` | `"generic"` | Definiert die Umgebungsumgebung (z. B. prod, staging, generic). |
| `general_hostname` | `{{ inventory_hostname }}` | Der zu setzende System-Hostname. |

### Root Local Setup

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_root_setup_enabled` | `false` | Aktiviert oder deaktiviert die lokale Konfiguration des Root-Benutzers. |
| `general_root_password` | `""` | Das verschlüsselte Passwort für den Root-Benutzer. |

### MOTD Configuration

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_motd_enabled` | `true` | Aktiviert oder deaktiviert das Verwalten der MOTD-Begrüßung. |
| `general_motd_path` | `"/etc/motd"` | Der Dateipfad zur MOTD-Konfigurationsdatei. |
| `general_motd_managed_by` | `"Ansible (general role)"` | Der Signatur-Text, wer diese Datei verwaltet. |
| `general_motd_title` | `"System Information"` | Die Überschrift innerhalb der MOTD-Anzeige. |

### APT Packages

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_debian_base_packages` | `["ssh", "vim"]` | Liste von Standard-Paketen, die immer installiert sein müssen. |
| `general_debian_extra_packages` | `[]` | Optionale Liste für zusätzliche, projektspezifische Pakete. |

### Admin User Configuration

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_admin_user` | `"ansible"` | Name des administrativen Systembenutzers, der angelegt wird. |
| `general_admin_sudo` | `true` | Steuert, ob der Admin-Benutzer passwortlose Sudo-Rechte erhält. |
| `general_admin_ssh_key` | `""` | Der öffentliche SSH-Schlüssel, der für den Admin-Benutzer hinterlegt wird. |
| `general_admin_shell` | `"/bin/bash"` | Die Standard-Shell für den Admin-Benutzer. |

### SSH Server Configuration

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_ssh_permit_root_login` | `"no"` | Erlaubt oder verbietet den SSH-Login direkt als Root-Benutzer. |
| `general_ssh_password_authentication` | `"no"` | Aktiviert oder deaktiviert die passwortbasierte Authentifizierung via SSH. |
| `general_ssh_port` | `22` | Der Port, auf dem der SSH-Dienst lauscht. |
| `general_ssh_log_level` | `"INFO"` | Die Protokollierungsstufe (LogLevel) des SSH-Daemons. |
| `general_ssh_keys` | `[]` | Liste globaler SSH-Schlüssel (z. B. für zentrale Admin- oder Root-Zugriffe). |
| `general_ssh_extra_keys` | `[]` | Optionale Liste für zusätzliche, temporäre oder hostspezifische SSH-Schlüssel. |

### SSH Banner

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_ssh_banner_enabled` | `true` | Aktiviert oder deaktiviert das Anzeigen eines Warnbanners vor dem Login. |
| `general_ssh_banner_path` | `/etc/issue.net` | Der Dateipfad, in dem der Text des SSH-Banners hinterlegt wird. |

### Networking Configuration

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_ip_interface` | `"eth0"` | *(Legacy)* Der Name der Standard-Netzwerkschnittstelle. |
| `general_ip_address` | `""` | *(Legacy)* Führt bei fehlender Konfiguration in der Produktion zum harten Abbruch (Sicherheits-Assert). |
| `general_ip_cidr` | `24` | *(Legacy)* Die standardmäßige Subnetzmaske in CIDR-Notation. |
| `general_ip_gateway` | `""` | *(Legacy)* Das Standard-Gateway|
| `general_networking` | *Siehe Block unten* | Eine strukturierte Liste zur Konfiguration einer oder mehrerer Netzwerkschnittstellen. |

#### Standardstruktur für `general_networking` (Übergangsphase):
```yaml
general_networking:
  - interface: "{{ general_ip_interface | default('eth0') }}"
    ip_address: "{{ general_ip_address | default('') }}"
    ip_cidr: "{{ general_ip_cidr | default(24) }}"
    ip_gateway: "{{ general_ip_gateway | default('') }}"
```

### DNS Configuration

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_dns_servers` | `["1.1.1.1", "8.8.8.8"]` | Liste der zu verwendenden DNS-Nameserver. |
| `general_dns_search_domains` | `[]` | Optionale Liste von DNS-Suchdomänen (Search Domains). |
| `general_dns_llmnr_disable` | `true` | Steuert, ob Link-Local Multicast Name Resolution (LLMNR) deaktiviert werden soll. |


### Journald Configuration

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_journald_storage` | `persistent` | Bestimmt die Speicherart der Logs (z. B. `persistent` für dauerhafte Speicherung auf der Festplatte). |
| `general_journald_max_use` | `"1G"` | Maximale Festplattenkapazität, die von Journald-Logs belegt werden darf. |
| `general_journald_keep_free` | `"100M"` | Mindestens freizuhaltender Speicherplatz auf dem Dateisystem für andere Dienste. |
| `general_journald_max_file_size` | `"100M"` | Maximale Dateigröße für eine einzelne Journald-Logdatei vor der Rotation. |
| `general_journald_max_files` | `10` | Maximale Anzahl an rotierungsbasierten Logdateien, die vorgehalten werden. |

### Application Log Directory

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_app_log_owner` | `"root"` | Besitzer (User) des zentralen Anwendungs-Logverzeichnisses. |
| `general_app_log_group` | `"root"` | Gruppe (Group) des zentralen Anwendungs-Logverzeichnisses. |
| `general_app_log_dir` | `"/var/log/app"` | Absoluter Pfad zum primären Verzeichnis für Anwendungs-Logs. |

### Ansible Task Log-Hiding (No Log)

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_hide_dns_logs` | `true` | Blendet sensible Ausgaben oder Parameter bei DNS-Tasks im Ansible-Output aus. |
| `general_hide_hostname_logs` | `true` | Unterdrückt die Task-Protokollierung bei der Hostname-Änderung im Terminal. |
| `general_hide_root_password_logs` | `true` | Verhindert strikt, dass Root-Passwörter oder Hashes im Klartext-Log von Ansible auftauchen. |
| `general_hide_user_logs` | `true` | Schützt Benutzerdaten und Passwörter beim Anlegen von System-Usern vor dem Logging. |
| `general_hide_ssh_logs` | `true` | Blendet Details bei der Einrichtung und Hinterlegung von SSH-Schlüsseln im Log aus. |

### Timezone

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_timezone` | `"Europe/Berlin"` | Die auf dem Zielsystem einzustellende Zeitzone (z. B. für korrekte Log-Zeitstempel). |


### Firewall (UFW) Configuration

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_ufw_logging` | `"medium"` | Die Protokollierungsstufe für UFW-Ereignisse (z. B. low, medium, high). |
| `general_ufw_ports` | `[]` | Liste der primären Ports, die in der Firewall freigeschaltet werden sollen. |
| `general_ufw_extra_ports` | `[]` | Optionale Liste für zusätzliche oder hostspezifische Port-Freischaltungen. |

#### Beispielstruktur für `general_ufw_ports`:
```yaml
general_ufw_ports:
  - port: "{{ general_ssh_port }}"
    proto: tcp
    state: enabled
    comment: "SSH"
```

### Fail2Ban Configuration

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_fail2ban_enabled` | `true` | Aktiviert oder deaktiviert den Fail2Ban-Dienst global. |
| `general_fail2ban_ssh_enabled` | `true` | Aktiviert den Fail2Ban-Schutz speziell für den SSH-Dienst. |
| `general_fail2ban_ssh_port` | `{{ general_ssh_port \| default(22) }}` | Der zu überwachende SSH-Port (folgt standardmäßig der SSH-Konfiguration). |
| `general_fail2ban_ssh_maxretry` | `5` | Anzahl der erlaubten Fehlversuche, bevor eine IP blockiert wird. |
| `general_fail2ban_backend` | `"systemd"` | Definiert das Backend für die Protokollüberwachung. |
| `general_fail2ban_logpath` | `"$(systemd_journal)s"` | Pfad für die Logdateien. |
| `general_fail2ban_ssh_bantime` | `600` | Dauer der IP-Sperre in Sekunden (10 Minuten). |
| `general_fail2ban_ssh_findtime` | `600` | Zeitfenster in Sekunden, in dem die Fehlversuche gezählt werden. |

### NTP Configuration

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_ntp_enabled` | `true` | Aktiviert oder deaktiviert die automatische NTP-Zeitsynchronisation auf dem System. |

### GPG Configuration

| Variable | Standardwert | Beschreibung |
| :--- | :--- | :--- |
| `general_gpg_create_key` | `false` | Steuert, ob automatisch ein neues GPG-Schlüsselpaar generiert werden soll. |
| `general_gpg_force_regenerate` | `false` | Erzwingt das Neuerstellen des GPG-Schlüssels, selbst wenn bereits einer existiert. |
| `general_gpg_user` | `{{ general_admin_user }}` | Der Systembenutzer, für dessen Heimatverzeichnis der GPG-Schlüssel eingerichtet wird. |
| `general_gpg_home` | `"/home/{{ general_gpg_user }}/.gnupg"` | Der absolute Pfad zum GPG-Konfigurations- und Schlüsselverzeichnis. |
| `general_gpg_name` | `"Server {{ inventory_hostname }}"` | Der Name (Real Name / User ID), der im GPG-Schlüssel hinterlegt wird. |
| `general_gpg_group` | `"server"` | Logischer Gruppenname, der für die Generierung der E-Mail-Adresse genutzt wird. |
| `general_gpg_email` | `"{{ inventory_hostname }}@{{ general_gpg_group }}"` | Die automatisch generierte E-Mail-Adresse für die GPG-Identität. |
| `general_gpg_comment` | `"Auto generated by Ansible"` | Kommentarfeld innerhalb der Metadaten des GPG-Schlüssels. |
| `general_gpg_key_type` | `"RSA"` | Der zu verwendende Verschlüsselungsalgorithmus für den Hauptschlüssel. |
| `general_gpg_key_length` | `4096` | Die Bit-Länge des GPG-Hauptschlüssels (Sicherheits-Standard). |
| `general_gpg_subkey_type` | `"RSA"` | Der Verschlüsselungsalgorithmus für den GPG-Unterschlüssel (Subkey). |
| `general_gpg_subkey_length` | `4096` | Die Bit-Länge des GPG-Unterschlüssels. |
| `general_gpg_expire_date` | `"0"` | Ablaufdatum des Schlüssels (`0` bedeutet, dass der Schlüssel unendlich gültig ist). |
| `general_gpg_passphrase` | `""` | Passwort zum Schutz des privaten Schlüssels (leer lassen für unverschlüsselte Schlüssel). |
| `general_gpg_import_pub_keys` | `[]` | Liste von bereits existierenden öffentlichen GPG-Schlüsseln, die importiert werden sollen. |

#### Beispielstruktur für `general_gpg_import_pub_keys`:
```yaml
general_gpg_import_pub_keys:
  - name: "Zentraler Admin Key"
    key: "---BEGIN PGP PUBLIC KEY BLOCK---..."
```

