# Internet Speed Tracker

## 📌 Projektbeschreibung
Dieses Projekt misst regelmässig die Internetgeschwindigkeit und speichert die Ergebnisse in einer SQLite-Datenbank. Zusätzlich werden Diagramme zur Analyse der Geschwindigkeit erstellt, und Benachrichtigungen werden gesendet, wenn die Geschwindigkeit zu niedrig ist.

## 🔧 Voraussetzungen
Bevor du beginnst, stelle sicher, dass du die folgenden Pakete installiert hast:

```bash
sudo apt update && sudo apt upgrade -y
pip3 install speedtest-cli matplotlib sqlite3 plyer
```

## 📂 Projektstruktur
```
📁 speedtest-tracker
│── speedtest_tracker.py  # Hauptskript
│── speedtest_results.db  # SQLite-Datenbank
│── README.md  # Dokumentation
```

## 🛠️ Schritt-für-Schritt-Anleitung

### 1️⃣ Repository einrichten
Erstelle ein neues GitHub-Repository und klone es:

```bash
git clone https://github.com/deinbenutzername/speedtest-tracker.git
cd speedtest-tracker
```

### 2️⃣ Hauptskript erstellen
Erstelle die Datei `speedtest_tracker.py` und füge folgenden Code hinzu:

```python
import speedtest
import sqlite3
import time
import schedule
import matplotlib.pyplot as plt
from plyer import notification

def run_speedtest():
    st = speedtest.Speedtest()
    st.get_best_server()
    download_speed = st.download() / 1_000_000  # Mbit/s
    upload_speed = st.upload() / 1_000_000  # Mbit/s
    ping = st.results.ping
    
    conn = sqlite3.connect("speedtest_results.db")
    cursor = conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS results (timestamp TEXT, download REAL, upload REAL, ping REAL)")
    cursor.execute("INSERT INTO results (timestamp, download, upload, ping) VALUES (datetime('now'), ?, ?, ?)", 
                   (download_speed, upload_speed, ping))
    conn.commit()
    conn.close()
    
    print(f"Download: {download_speed:.2f} Mbit/s, Upload: {upload_speed:.2f} Mbit/s, Ping: {ping} ms")
    check_speed_alert(download_speed, upload_speed)

def check_speed_alert(download_speed, upload_speed):
    if download_speed < 5:
        notification.notify(title="⚠️ Niedrige Download-Geschwindigkeit!", 
                            message=f"Download-Speed: {download_speed:.2f} Mbit/s", timeout=10)
    if upload_speed < 1:
        notification.notify(title="⚠️ Niedrige Upload-Geschwindigkeit!", 
                            message=f"Upload-Speed: {upload_speed:.2f} Mbit/s", timeout=10)

def plot_results():
    conn = sqlite3.connect("speedtest_results.db")
    cursor = conn.cursor()
    cursor.execute("SELECT timestamp, download, upload, ping FROM results")
    data = cursor.fetchall()
    conn.close()
    
    timestamps = [row[0] for row in data]
    download_speeds = [row[1] for row in data]
    upload_speeds = [row[2] for row in data]
    ping_values = [row[3] for row in data]
    
    plt.figure(figsize=(10, 6))
    plt.subplot(3, 1, 1)
    plt.plot(timestamps, download_speeds, label="Download (Mbit/s)", color='b')
    plt.xticks(rotation=45)
    plt.subplot(3, 1, 2)
    plt.plot(timestamps, upload_speeds, label="Upload (Mbit/s)", color='g')
    plt.xticks(rotation=45)
    plt.subplot(3, 1, 3)
    plt.plot(timestamps, ping_values, label="Ping (ms)", color='r')
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()

schedule.every(30).minutes.do(run_speedtest)
while True:
    schedule.run_pending()
    time.sleep(1)
```

### 3️⃣ Skript ausführen

```bash
python3 speedtest_tracker.py
```

### 4️⃣ Ergebnisse visualisieren
Um die Geschwindigkeitsschwankungen in einem Diagramm zu sehen, füge diesen Befehl hinzu:

```bash
python3 -c "import speedtest_tracker; speedtest_tracker.plot_results()"
```

### 5️⃣ Automatische Ausführung einrichten (optional)
Falls du das Skript automatisch starten möchtest, erstelle einen **Cronjob**:

```bash
crontab -e
```
Füge folgende Zeile hinzu:
```cron
*/30 * * * * python3 /pfad/zu/speedtest_tracker.py
```

### 6️⃣ Änderungen ins GitHub-Repository hochladen

```bash
git add .
git commit -m "Initialer Commit"
git push origin main
```

## 🎯 Erweiterungen
- Web-Oberfläche mit Flask oder Django
- API für mobile Benachrichtigungen
- Detailliertere Statistiken (Durchschnittswerte, Spitzenzeiten)

## 🏁 Fazit
Dieses Projekt hilft dir, deine Internetgeschwindigkeit zu analysieren und Schwankungen zu erkennen. 🚀
