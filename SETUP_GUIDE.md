# 📋 Guida Setup Full-Stack: Vue 3 + Laravel + PostgreSQL

Una guida step-by-step riutilizzabile per creare progetti full-stack con **Vue 3 (frontend)**, **Laravel (backend)** e **PostgreSQL (database in Docker)**.

**Tempo stimato:** ~30-40 minuti per il setup completo

---

## 📋 Prerequisiti

Prima di iniziare, assicurati di avere installato:

```bash
# Verifica installazioni
node --version      # Deve essere v20+
npm --version       # Deve essere 9+
php --version       # Deve essere 8.2+
composer --version  # Composer per Laravel
docker --version    # Docker per PostgreSQL
git --version       # Git per versionamento
```

### Se manca qualcosa:

**Node.js 20+** (usa nvm se hai versione vecchia):
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
source ~/.bashrc
nvm install 20
nvm use 20
```

**PHP PostgreSQL driver:**
```bash
sudo apt install php-pgsql
```

**Docker:**
```bash
sudo apt update
sudo apt install docker.io docker-compose
sudo usermod -aG docker $USER
# Riavvia il terminale per applicare i permessi
```

**Composer:**
```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

---

## 🚀 Setup Progetto

### Step 1: Creare la cartella principale

```bash
mkdir nome-progetto
cd nome-progetto
```

### Step 2: Creare il frontend Vue 3

```bash
npm create vite@latest frontend -- --template vue
cd frontend
npm install
npm run dev  # Testa che funzioni su http://localhost:5173/
# Premi Ctrl+C per fermare il server
cd ..
```

### Step 3: Creare il backend Laravel

```bash
composer create-project laravel/laravel backend
cd backend
```

### Step 4: Configurare Laravel per PostgreSQL

Nel file `backend/.env`, modifica le righe di database:

```env
DB_CONNECTION=pgsql
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=nome_app
DB_USERNAME=app_user
DB_PASSWORD=app_password
```

Salva il file.

### Step 5: Creare docker-compose.yml per PostgreSQL

Torna nella cartella principale (`cd ..`) e crea un file `docker-compose.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: nome_app_postgres
    environment:
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD: app_password
      POSTGRES_DB: nome_app
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app_user"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

**Nota:** Sostituisci `nome_app` con il nome del tuo progetto (es: `todo_app`, `blog_app`, ecc.)

### Step 6: Avviare PostgreSQL con Docker

```bash
docker-compose up -d
```

Verifica che sia partito:
```bash
docker ps
```

Dovresti vedere il container in esecuzione.

### Step 7: Testare la connessione a PostgreSQL

```bash
cd backend
php artisan migrate
```

Se vedi i messaggi di migrazione (es: `Migrated: 2014_10_12_000000_create_users_table`), significa che Laravel si è connesso a PostgreSQL! ✅

Se vedi errore `could not find driver`, torna ai prerequisiti e installa `php-pgsql`.

### Step 8: Creare .gitignore

Nella cartella principale, crea un file `.gitignore`:

```
# Frontend
frontend/node_modules/
frontend/dist/
frontend/.env.local

# Backend
backend/node_modules/
backend/vendor/
backend/.env
backend/.env.local
backend/storage/
backend/bootstrap/cache/

# Sistema
.DS_Store
Thumbs.db
*.log

# IDE
.vscode/
.idea/
*.swp
*.swo
```

### Step 9: Inizializzare Git e GitHub

```bash
git init
git add .
git commit -m "Initial commit: Vue 3 + Laravel + PostgreSQL setup"
git branch -M main
git remote add origin https://github.com/TUO_USERNAME/nome-progetto.git
git push -u origin main
```

**Nota:** Crea il repo su GitHub prima di pushare: https://github.com/new

---

## 📂 Struttura finale

```
nome-progetto/
├── frontend/              # Vue 3 app
│   ├── src/
│   ├── package.json
│   └── vite.config.js
├── backend/               # Laravel app
│   ├── app/
│   ├── routes/
│   ├── database/
│   ├── .env               # (NON commitare!)
│   └── artisan
├── docker-compose.yml     # PostgreSQL config
├── .gitignore
└── README.md
```

---

## 🔧 Comandi utili durante lo sviluppo

### Frontend (Vue 3)

```bash
cd frontend

# Avviare il server di sviluppo
npm run dev

# Build per produzione
npm run build

# Aggiungere una dipendenza
npm install nome-pacchetto
```

### Backend (Laravel)

```bash
cd backend

# Avviare il server di sviluppo (Laravel Artisan)
php artisan serve

# Creare una nuova migration
php artisan make:migration nome_migration

# Eseguire le migrazioni
php artisan migrate

# Creare un nuovo model
php artisan make:model NomeModel

# Creare un nuovo controller
php artisan make:controller NomeController

# Vedere tutte le rotte
php artisan route:list
```

### Database (PostgreSQL)

```bash
# Avviare PostgreSQL (se non è già in esecuzione)
docker-compose up -d

# Fermare PostgreSQL
docker-compose down

# Accedere alla shell di PostgreSQL
docker exec -it nome_app_postgres psql -U app_user -d nome_app

# Vedere tutti i container attivi
docker ps
```

### Git

```bash
# Aggiungere e committare
git add .
git commit -m "Descrizione del commit"

# Pushare su GitHub
git push

# Vedere lo stato
git status

# Vedere il log dei commit
git log --oneline
```

---

## 🧪 Verificare che tutto funzioni

Una volta completato il setup, verifica che tutto sia corretto:

### 1. Frontend Vue funziona?
```bash
cd frontend
npm run dev
# Apri http://localhost:5173/ nel browser
# Dovresti vedere la pagina di benvenuto di Vue
```

### 2. Backend Laravel funziona?
```bash
cd backend
php artisan serve
# Dovresti vedere un messaggio tipo "Laravel development server started..."
```

### 3. Database PostgreSQL è connesso?
```bash
cd backend
php artisan migrate --fresh
# Dovresti vedere i messaggi di migrazione senza errori
```

Se tutto funziona, sei pronto per sviluppare! 🎉

---

## ⚙️ Variabili da personalizzare ogni volta

Quando usi questa guida per un nuovo progetto, sostituisci:

| Variabile | Significato | Esempio |
|-----------|-----------|---------|
| `nome-progetto` | Nome cartella principale | `todo-app`, `blog-app` |
| `nome_app` | Nome in docker-compose.yml | `todo_app`, `blog_app` |
| `nome_app_postgres` | Nome container Docker | `todo_app_postgres` |
| `TUO_USERNAME` | Il tuo username GitHub | `FontaMatte` |

---

## 🐛 Troubleshooting

### Errore: "could not find driver (pgsql)"
**Soluzione:** Installa il driver PostgreSQL per PHP
```bash
sudo apt install php-pgsql
```

### Errore: "docker: command not found"
**Soluzione:** Installa Docker
```bash
sudo apt install docker.io docker-compose
```

### Errore: "Node.js 18.19.1" (versione troppo vecchia)
**Soluzione:** Aggiorna Node con nvm
```bash
nvm install 20
nvm use 20
```

### Porta 5432 già in uso (PostgreSQL)
**Soluzione:** Ferma il container precedente
```bash
docker-compose down
docker-compose up -d
```

### Errore: "Cannot find module" nel frontend
**Soluzione:** Reinstalla le dipendenze
```bash
cd frontend
rm -rf node_modules package-lock.json
npm install
```

---

## 📚 Prossimi step dopo il setup

1. **Sessione 2:** Creare migration per la tabella `todos` + API endpoints
2. **Sessione 3:** Componenti Vue per mostrare i dati
3. **Sessione 4:** Funzionalità complete (add, delete, update)
4. **Sessione 5:** Tema light/dark e deployment

---

## 📝 Note importanti

- ✅ **Committa regolarmente** su GitHub (non aspettare di finire tutto)
- ✅ **Il file `.env` NON deve essere committato** (contiene password)
- ✅ **Docker deve stare in esecuzione** mentre sviluppi
- ✅ **Node 20+** è obbligatorio per Vite moderno
- ✅ **PHP 8.2+** per Laravel moderno

---

## 💡 Tip per futuro

Dopo aver fatto questo setup una o due volte, lo farai automaticamente. La volta dopo puoi anche creare uno script bash che lo fa tutto in automatico! 

Per ora, imparare ogni step è importante per capire cosa stai facendo. 🚀

---

**Creato per:** Matteo | **Versione:** 1.0 | **Data:** Nov 2025