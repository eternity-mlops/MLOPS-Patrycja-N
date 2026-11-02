# MLOPS-Patrycja-N
# Usługa Llamafile (Systemd & REST API)

Usługa działająca jako odizolowany użytkownik systemowy, zarządzana przez `systemd` z wystawionym interfejsem REST API tylko na interfejsie host-only.

## 1. Instrukcja instalacji i konfiguracji (Krok po kroku)

### 1.1 Tworzenie użytkownika systemowego
```bash
sudo useradd -r -s /bin/false llamafile
