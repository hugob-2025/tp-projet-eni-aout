# Première étape : utiliser Node.js pour construire l'application Angular
FROM node:20-alpine AS build

# Définir le dossier de travail dans le conteneur
WORKDIR /app

# Copier les fichiers de dépendances
COPY package*.json ./

# Installer les dépendances du projet
RUN npm ci

# Copier le reste du projet Angular
COPY . .

# Configurer l'URL de l'API pour la production
RUN sed -i "s|REPLACE_AT_BUILD_TIME|/api|" src/environments/environment.prod.ts && \
    npm run build -- --configuration production

# Deuxième étape : utiliser Nginx pour servir l'application
FROM nginx:alpine

# Copier les fichiers Angular générés vers le dossier web de Nginx
COPY --from=build /app/dist/frontend /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Exposer le port 80 utilisé par Nginx
EXPOSE 80

# Démarrer Nginx lorsque le conteneur est lancé
CMD ["nginx", "-g", "daemon off;"]
