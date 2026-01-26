# 🗄️ Configuration Supabase pour AFTER LIGUE

## Étape 1 : Créer un compte Supabase (Gratuit)

1. Va sur **[supabase.com](https://supabase.com)**
2. Clique sur "Start your project"
3. Connecte-toi avec GitHub (recommandé) ou email
4. Crée un nouveau projet :
   - **Name**: `after-ligue`
   - **Database Password**: choisis un mot de passe fort (note-le !)
   - **Region**: West EU (Paris) pour la France
5. Attends ~2 minutes que le projet se crée

---

## Étape 2 : Créer les tables

Va dans **SQL Editor** (menu de gauche) et exécute ce script :

```sql
-- =============================================
-- SCHÉMA DE BASE DE DONNÉES AFTER LIGUE
-- =============================================

-- Table des joueurs
CREATE TABLE players (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    discord_name TEXT UNIQUE NOT NULL,
    discord_id TEXT UNIQUE,
    avatar_url TEXT,
    status TEXT DEFAULT 'solo' CHECK (status IN ('solo', 'duo')),
    partner TEXT,
    availability TEXT,
    levels JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Table des équipes
CREATE TABLE teams (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    name TEXT UNIQUE NOT NULL,
    image_url TEXT,
    players TEXT[] NOT NULL,
    active BOOLEAN DEFAULT true,
    points JSONB DEFAULT '{"rocket-league":0,"brawlhalla":0,"league-of-legends":0,"valorant":0,"fall-guys":0,"trackmania":0,"hearthstone":0,"2xko":0}',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Table des matchs
CREATE TABLE matches (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    team1_id UUID REFERENCES teams(id),
    team2_id UUID REFERENCES teams(id),
    game_id TEXT NOT NULL,
    winner TEXT CHECK (winner IN ('team1', 'team2', 'draw')),
    declared_by TEXT,
    date TIMESTAMPTZ DEFAULT NOW(),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Table des demandes d'amis
CREATE TABLE friend_requests (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    from_user TEXT NOT NULL,
    to_user TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(from_user, to_user)
);

-- Table des amitiés
CREATE TABLE friendships (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    user1 TEXT NOT NULL,
    user2 TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(user1, user2)
);

-- Table des messages privés
CREATE TABLE messages (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    from_user TEXT NOT NULL,
    to_user TEXT NOT NULL,
    content TEXT NOT NULL,
    read BOOLEAN DEFAULT false,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Table des paramètres
CREATE TABLE settings (
    id INTEGER PRIMARY KEY DEFAULT 1,
    registration_open BOOLEAN DEFAULT true,
    season_name TEXT DEFAULT 'Saison 1',
    tournament_started BOOLEAN DEFAULT false,
    admin_password TEXT DEFAULT 'admin123',
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Insérer les paramètres par défaut
INSERT INTO settings (id) VALUES (1);

-- =============================================
-- POLITIQUES DE SÉCURITÉ (Row Level Security)
-- =============================================

-- Activer RLS sur toutes les tables
ALTER TABLE players ENABLE ROW LEVEL SECURITY;
ALTER TABLE teams ENABLE ROW LEVEL SECURITY;
ALTER TABLE matches ENABLE ROW LEVEL SECURITY;
ALTER TABLE friend_requests ENABLE ROW LEVEL SECURITY;
ALTER TABLE friendships ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE settings ENABLE ROW LEVEL SECURITY;

-- Politiques : tout le monde peut lire (pour le MVP)
CREATE POLICY "Public read access" ON players FOR SELECT USING (true);
CREATE POLICY "Public read access" ON teams FOR SELECT USING (true);
CREATE POLICY "Public read access" ON matches FOR SELECT USING (true);
CREATE POLICY "Public read access" ON friend_requests FOR SELECT USING (true);
CREATE POLICY "Public read access" ON friendships FOR SELECT USING (true);
CREATE POLICY "Public read access" ON messages FOR SELECT USING (true);
CREATE POLICY "Public read access" ON settings FOR SELECT USING (true);

-- Politiques : tout le monde peut écrire (pour le MVP - à sécuriser plus tard)
CREATE POLICY "Public write access" ON players FOR ALL USING (true);
CREATE POLICY "Public write access" ON teams FOR ALL USING (true);
CREATE POLICY "Public write access" ON matches FOR ALL USING (true);
CREATE POLICY "Public write access" ON friend_requests FOR ALL USING (true);
CREATE POLICY "Public write access" ON friendships FOR ALL USING (true);
CREATE POLICY "Public write access" ON messages FOR ALL USING (true);
CREATE POLICY "Public write access" ON settings FOR ALL USING (true);

-- Index pour les performances
CREATE INDEX idx_players_discord ON players(discord_name);
CREATE INDEX idx_messages_users ON messages(from_user, to_user);
CREATE INDEX idx_matches_teams ON matches(team1_id, team2_id);
```

---

## Étape 3 : Récupérer les clés API

1. Va dans **Settings** → **API** (menu de gauche)
2. Copie ces valeurs :
   - **Project URL** : `https://xxxxx.supabase.co`
   - **anon public key** : `eyJhbGciOiJS...` (la clé publique)

---

## Étape 4 : Configurer le site

Ouvre `index.html` et trouve la section `SUPABASE CONFIG` (vers le début du fichier).

Remplace les valeurs :

```javascript
const SUPABASE_URL = 'https://TON-PROJET.supabase.co';
const SUPABASE_ANON_KEY = 'TA-CLE-ANON-PUBLIQUE';
```

---

## Étape 5 : Tester

1. Ouvre ton site
2. Crée un compte / connecte-toi
3. Vérifie dans Supabase → **Table Editor** que les données apparaissent

---

## 🔒 Sécurité (Pour plus tard)

Pour la production, tu devras :
1. Configurer l'authentification Discord OAuth
2. Restreindre les politiques RLS aux utilisateurs authentifiés
3. Ajouter des validations côté serveur

---

## 📊 Tableau de bord Supabase

Tu peux voir toutes tes données dans :
- **Table Editor** : voir/modifier les données
- **SQL Editor** : exécuter des requêtes
- **Authentication** : gérer les utilisateurs
- **Logs** : voir les erreurs

---

## 💡 Aide

- Documentation Supabase : https://supabase.com/docs
- Discord Supabase : https://discord.supabase.com
