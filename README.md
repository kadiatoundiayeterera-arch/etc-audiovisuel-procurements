# etc-audiovisuel-procurements
Bot d'aide aux appels d'offres 
git clone https://github.com/<kadiatoundiayeterera-arch>/etc-audiovisuel-procurements.gitcd etc-audiovisuel-procurements
# Script Python principal
cat > monitor_procurements.py <<'EOF'
EOF

# Dépendances Python
cat > requirements.txt <<'EOF'
requests
python-dotenv
EOF

# Exemple de fichier d'environnement
cat > .env.example <<'EOF'
BOAMP_API_KEY=your_api_key_here
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=your_email@example.com
SMTP_PASS=your_password
ALERT_TO=destination@example.com
EOF

# Dockerfile pour exécuter le bot
cat > Dockerfile <<'EOF'
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

CMD ["python", "monitor_procurements.py"]
EOF

# README avec instructions
cat > README_RUN.md <<'EOF'
# ETC Audiovisuel Procurement Bot

##  Lancer en local
bash
pip install -r requirements.txt
python monitor_procurements.py
docker build -t etc-bot .
docker run --env-file .env etc-bot
