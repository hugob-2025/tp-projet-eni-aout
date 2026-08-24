# Utiliser une image contenant Node.js 20 sur Alpine Linux
FROM node:20-alpine

# Définir /app comme dossier de travail dans le conteneur
WORKDIR /app

# Copier package.json et package-lock.json
COPY package*.json ./

# Installer uniquement les dépendances nécessaires en production
RUN npm ci --omit=dev

# Copier le reste du code du backend
COPY . .

# Indiquer que l'application écoute sur le port 3000
EXPOSE 3000

# Démarrer le serveur backend
CMD ["npm", "start"]
