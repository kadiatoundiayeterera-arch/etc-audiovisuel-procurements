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

```bash
git add .
git commit -m "Ajout du bot BOAMP"
git push origin main
mkdir -p .github/workflows

cat > .github/workflows/bot.yml <<'EOF'
name: Monitor Procurements

on:
  schedule:
    - cron: "0 */6 * * *"  # toutes les 6 heures
  workflow_dispatch:

jobs:
  run-bot:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run bot
        env:
          BOAMP_API_KEY: ${{ secrets.BOAMP_API_KEY }}
          SMTP_HOST: ${{ secrets.SMTP_HOST }}
          SMTP_PORT: ${{ secrets.SMTP_PORT }}
          SMTP_USER: ${{ secrets.SMTP_USER }}
          SMTP_PASS: ${{ secrets.SMTP_PASS }}
          ALERT_TO: ${{ secrets.ALERT_TO }}
        run: python monitor_procurements.py
EOF
git add .github/workflows/bot.yml
git commit -m "Ajout du workflow GitHub Actions"
git push origin main
