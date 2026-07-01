## 📦 Compilation et Déploiement en Production
Pour compiler l'application de façon optimisée pour la production (fichiers statiques légers et minifiés) :
### 1. Build du projet
```bash
npm run build

```
### 2. Fichiers générés
Le processus de build génère un répertoire dist/ à la racine de devweb/. Ce dossier contient la totalité des fichiers statiques autonomes (index.html, fichiers js et css packagés dans le dossier assets/).
### 3. Hébergement
Puisqu'il s'agit d'une application statique (SPA), vous pouvez déployer le contenu du dossier dist/ sur n'importe quel serveur HTTP :
 * **Serveur standard** : Nginx, Apache.
 * **Hébergement Cloud** : Vercel, Netlify, AWS S3, Firebase Hosting.
#### Exemple de bloc serveur minimal pour Nginx :
```nginx
server {
    listen 80;
    server_name votre-domaine.com;

    location / {
        root /chemin/vers/le/projet/devweb/dist;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}

```
