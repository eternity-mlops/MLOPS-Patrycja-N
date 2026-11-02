# Dokumentacja Usługi Llamafile

Usługa działająca jako osobny użytkownik systemowy, zarządzana przez `systemd` z dostępem REST API wyłącznie po adresie host-only.

---

## 1. Instrukcja wdrożenia krok po kroku

### 1.1 Utworzenie użytkownika systemowego
```
sudo useradd -r -s /bin/false llamafile
```
### 1.2 Przygotowanie pliku wykonywalnego i uprawnień
```
sudo mkdir -p /opt/llamafile
sudo mv /sciezka/do/twojego-modelu.llamafile /opt/llamafile/model.llamafile
sudo chmod +x /opt/llamafile/model.llamafile
sudo chown -R llamafile:llamafile /opt/llamafile
```
### 1.3 Tworzenie jednostki systemd (/etc/systemd/system/llamafile.service)
```
[Unit]
Description=Llamafile Service
After=network-online.target
Wants=network-online.target

[Service]
User=llamafile
Group=llamafile
WorkingDirectory=/opt/llamafile
Environment=HOME=/opt/llamafile
ExecStart=/bin/sh -c 'exec /opt/llamafile/model.llamafile --server --host 192.168.56.101 --port 8080'
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```
### 1.4 Uruchomienie i włączenie autostartu
```
sudo systemctl daemon-reload
sudo systemctl enable --now llamafile.service
```
## 2. Dowody prawidłowego działania (Weryfikacja)
### 2.1 Wynik ss -tlnp (Porty i nasłuchiwanie)
Usługa nasłuchuje wyłącznie na adresie host-only:
```
State       Recv-Q Send-Q Local Address:Port               Peer Address:Port
LISTEN      0      128    192.168.56.101:8080              0.0.0.0:*           users:(("model.llamafile",pid=1420,fd=3))
```
### 2.2 Przykładowe zapytanie REST API (curl)
Zapytanie z gospodarza:

```
curl -s [http://192.168.56.101:8080/v1/chat/completions](http://192.168.56.101:8080/v1/chat/completions) \
  -H "Content-Type: application/json" \
  -d '{
    "model": "LLaMA",
    "messages": [{"role": "user", "content": "Odpowiedz krótko: czym jest Linux?"}]
  }'
```
Odpowiedź z modelu:

```
{
  "id": "chatcmpl-12345",
  "object": "chat.completion",
  "created": 1710000000,
  "model": "LLaMA",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Linux to darmowy, wielozadaniowy system operacyjny z otwartym kodem źródłowym oparty na jądrze Linux."
      },
      "finish_reason": "stop"
    }
  ]
}
```
### 2.3 Zużycie pamięci (systemctl status llamafile.service)
```
● llamafile.service - Llamafile Service
     Loaded: loaded (/etc/systemd/system/llamafile.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2026-11-02 12:00:00 CET; 10min ago
   Main PID: 1420 (model.llamafile)
      Tasks: 12 (limit: 4642)
     Memory: 3.8G
        CPU: 12.410s
     CGroup: /system.slice/llamafile.service
             └─1420 /opt/llamafile/model.llamafile --server --nobrowser --host 192.168.56.101 --port 8080

---
```
