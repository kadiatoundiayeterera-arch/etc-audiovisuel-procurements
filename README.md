# etc-audiovisuel-procurements
Bot d'aide aux appels d'offres 
git clone https://github.com/<kadiatoundiayeterera-arch>/etc-audiovisuel-procurements.gitcd etc-audiovisuel-procurements
cat > monitor_procurements.py <<'EOF'
import os
import requests
import smtplib
import tempfile
from email.message import EmailMessage
from dotenv import load_dotenv

# Charger les variables d'environnement
load_dotenv()

BOAMP_API_KEY = os.getenv("BOAMP_API_KEY")
SMTP_HOST = os.getenv("SMTP_HOST")
SMTP_PORT = int(os.getenv("SMTP_PORT", "587"))
SMTP_USER = os.getenv("SMTP_USER")
SMTP_PASS = os.getenv("SMTP_PASS")
ALERT_TO = os.getenv("ALERT_TO")

# Mots-clés pour filtrer les annonces
KEYWORDS = ["audiovisuelle", "installation fixe", "prestation"]

def search_boamp():
    """Appelle l'API BOAMP pour récupérer les annonces correspondant aux mots-clés"""
    url = "https://api.dila.fr/opendata/api-boamp/v2/annonces/search"
    headers = {"accept": "application/json", "X-API-KEY": BOAMP_API_KEY}

    results = []
    for keyword in KEYWORDS:
        params = {"q": keyword, "nature": "appelOffre"}
        resp = requests.get(url, headers=headers, params=params)
        if resp.status_code == 200:
            data = resp.json()
            if "results" in data:
                results.extend(data["results"])
        else:
            print(f"Erreur API BOAMP ({resp.status_code}): {resp.text}")
    return results

def download_dce(dce_url):
    """Télécharge le DCE si disponible et le retourne comme fichier temporaire"""
    try:
        resp = requests.get(dce_url, timeout=30)
        if resp.status_code == 200:
            tmp = tempfile.NamedTemporaryFile(delete=False, suffix=".zip")
            tmp.write(resp.content)
            tmp.close()
            return tmp.name
    except Exception as e:
        print(f"Erreur lors du téléchargement du DCE: {e}")
    return None

def send_email(subject, body, attachment=None):
    """Envoie un email avec ou sans pièce jointe"""
    msg = EmailMessage()
    msg["From"] = SMTP_USER
    msg["To"] = ALERT_TO kadiatoundiayeterera@gmail.com
    msg["Subject"] = APPEL D'OFFRES
    msg.set_content(body)

    if attachment:
        with open(attachment, "rb") as f:
            msg.add_attachment(f.read(), maintype="application", subtype="zip", filename="DCE.zip")

    try:
        with smtplib.SMTP(SMTP_HOST, SMTP_PORT) as server:
            server.starttls()
            server.login(SMTP_USER, SMTP_PASS)
            server.send_message(msg)
        print("✅ Email envoyé")
    except Exception as e:
        print(f"❌ Erreur envoi email: {e}")

def main():
    annonces = search_boamp()
    for annonce in annonces:
        titre = annonce.get("objet", "Sans titre")
        ref = annonce.get("reference", "n/a")
        date_limite = annonce.get("dateLimite", "Non précisée")
        dce_url = annonce.get("urlDce")

        body = f"""
Nouvelle annonce détectée :

Titre : {titre}
Référence : {ref}
Date limite : {date_limite}
Lien BOAMP : {annonce.get('url')}"""

        attachment = None
        if dce_url:
            attachment = download_dce(dce_url)

        send_email(f"[Marché Public] {titre}", body, attachment)

if __name__ == "__main__":
    main()
EOF
