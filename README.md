# <p align="center">Hvordan hoste en webserver med flask applikasjon, med gunicorn

### Dette skal du ha fra før:
1. En Ubuntu sevrer

  
## 1.  Oppgradere serveren og last ned python
I terminalen, skriv:

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-pip python3-venv nginx -y
```
Dette laster ned Python og pakker til python.

  
## 2. Sett opp et Flask prosjekt
1. Skap en ny mappe for prosjektet, og naviger til denne mappe i terminal:
```sh
mkdir ~/flaskapp
cd ~/flaskapp
```

2. Skap et virtuelt miljø for prosjektet.  
Vi oppretter virtuell miljø sånn at pakker vi installerer for dette prosjektet ikke forstyrrer andre prosjekter
```sh
python3 -m venv venv
source venv/bin/activate
```

3. Installer Flask
```sh
pip install flask gunicorn
```

4. Lag en flask applikasjon
```sh
nano app.py
```

5. Legg inn hva du vil i filen.  
Du kan legge inn noe som dette:
```py
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello, World! Flask with Gunicorn and Nginx is working."

if __name__ == "__main__":
    app.run()
```
Du kan teste om den funker ved å skrive `python app.py` i terminalen. Deretter åpner du nettleser og skriv: `http://"serverens-IP":5000`.

## 3. Sette opp Gunicorn
1. Start med å sjekke om det funker
```sh
gunicorn --bind 0.0.0.0:8000 app:app
```

2. Opprett en systemd tjeneste for Gunicorn
```sh
sudo nano /etc/systemd/system/flaskapp.service
```

3. Inni filen, legg inn:
```
[Unit]
Description=Gunicorn instance to serve Flask application
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/home/<brukernavn>/flaskapp
Environment="PATH=/home/<brukernavn>/flaskapp/venv/bin"
ExecStart=/home/<brukernavn>/flaskapp/venv/bin/gunicorn --workers 3 --bind unix:flaskapp.sock -m 007 app:app

[Install]
WantedBy=multi-user.target
```
Bytt ut `<brukernavn>` med server sitt brukernavn.  
Trykk Ctrl + x for å lukke, y for å lagre og ENTER.

4. Start tjenesten
```sh
sudo systemctl daemon-reload
sudo systemctl start flaskapp
sudo systemctl enable flaskapp
```
Sjekk statusen:
```sh
sudo systemctl status flaskapp
```

## 4. Konfigurer Nginx
1. Opprett en NGinx fil
```sh
sudo nano /etc/nginx/sites-available/flaskapp
```
Legg inn dette innholdet:
```
server {
    listen 80;
    server_name <serverens-IP> <ditt-domenenavn>;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/<brukernavn>/flaskapp/flaskapp.sock;
    }
}
```
Bytt ut `brukernavn` med ditt brukernavn, og `serverens-IP` eller `ditt-domenenavn` med IP-adressen eller domenenavnet til serveren.

2. Aktiver NGinx konfigurasjonen
```sh
sudo ln -s /etc/nginx/sites-available/flaskapp /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

3. Test it  
Husk å legge til regelen for å tillate port 8000:
```sh
sudo ufw allow 8000
```
Start gunicorn igjen, og skriv `"Server-IP":8000` i nettleser.
```sh
gunicorn --bind 0.0.0.0:8000 app:app
```
