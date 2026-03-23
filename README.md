# PROJET DOCKER


## Création de la branche develop
git checkout -b develop

## Création de la branche feature pour le travail Docker
git checkout -b feature/docker-configuration
```

## Merge final
git checkout develop
git merge feature/docker-configuration
git checkout main
git merge develop
```
```bash
# Génération du fichier de logs
git log --pretty=format:"%h %ad | %s%d [%an]" --date=short > logs_projet.txt
```

---

## Installation


### 1. Générer les package-lock.json

`npm ci` nécessite un `package-lock.json`. On les génère avec :
```bash
cd services/auth-service && npm install && cd ../..
cd services/product-service && npm install && cd ../..
cd services/order-service && npm install && cd ../..
cd frontend && npm install && cd ..
```

---

## Environnement de développement

### Lancer tous les services
```bash
docker compose up --build
```

### En arrière-plan
```bash
docker compose up -d --build
```

### Accès à l'application
```
http://localhost:8080
```

### Voir les logs en temps réel
```bash
docker compose logs -f

# Logs d'un service spécifique
docker compose logs -f auth-service
docker compose logs -f product-service
docker compose logs -f order-service
```


### Initialiser les données produits
```bash
./scripts/init-products.sh
```

---

## Environnement de production avec Docker Swarm

### 1. Initialiser Docker Swarm
```bash
docker swarm init
```

### 2. Créer le secret JWT
```bash
echo "JWT_SECRET" | docker secret create jwt_secret -
```

### 3. Vérifier que le secret est créé
```bash
docker secret ls
```

### 4. Builder les images
```bash
docker compose -f docker-compose.prod.yml build
```

### 5. Taguer les images pour Swarm
```bash
docker tag e-commerce-vue-main-frontend:latest e-commerce_frontend:latest
docker tag e-commerce-vue-main-auth-service:latest e-commerce_auth-service:latest
docker tag e-commerce-vue-main-product-service:latest e-commerce_product-service:latest
docker tag e-commerce-vue-main-order-service:latest e-commerce_order-service:latest
```

### 6. Déployer la stack
```bash
docker stack deploy -c docker-compose.prod.yml e-commerce
```

### 7. Vérifier le déploiement
```bash
docker stack services e-commerce
```

Résultat obtenu :
```
ID             NAME                         MODE         REPLICAS
ps8j48t7qgwh   e-commerce_auth-service      replicated   2/2
9tpi6eq0yi5x   e-commerce_frontend          replicated   2/2
z7rkw1d3p40v   e-commerce_mongo-auth        replicated   1/1
ptnqul8pyo82   e-commerce_mongo-order       replicated   1/1
vz4twke2yswk   e-commerce_mongo-product     replicated   1/1
ogyci08n10m8   e-commerce_order-service     replicated   2/2
hxo9scn86mfe   e-commerce_product-service   replicated   2/2
```

### 8. Accès à l'application
```
http://localhost:8080
```

### Supprimer la stack
```bash
docker stack rm e-commerce
```



## Tests fonctionnels

### Auth Service
```bash
# Inscription
curl -X POST http://localhost:3001/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"password123"}'

# Résultat obtenu :
# {"message":"Utilisateur créé avec succès","token":"eyJ...","userId":"69c120812b2fdae58c5fcf60"}

# Connexion
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"password123"}'
```

### Product Service
```bash
# Initialiser les produits
./scripts/init-products.sh

# Résultat obtenu : 8 produits créés (Smartphone, MacBook, PS5, AirPods, etc.)

# Liste des produits
curl http://localhost:3000/api/products

# Ajouter au panier
curl -X POST http://localhost:3000/api/cart/add \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -d '{"userId":"<USER_ID>","productId":"<PRODUCT_ID>","quantity":1}'
```

### Order Service
```bash
# Passer une commande
curl -X POST http://localhost:3002/api/orders \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -d '{"products":[{"productId":"<PRODUCT_ID>","quantity":1}],"shippingAddress":{"street":"123 Rue Test","city":"Paris","postalCode":"75001"}}'

# Historique des commandes
curl http://localhost:3002/api/orders \
  -H "Authorization: Bearer <JWT_TOKEN>"
```

---

## Scan de sécurité avec Trivy
```bash
# Lancer Trivy via Docker
docker run aquasec/trivy image e-commerce_auth-service:latest
docker run aquasec/trivy image e-commerce_product-service:latest
docker run aquasec/trivy image e-commerce_order-service:latest
docker run aquasec/trivy image e-commerce_frontend:latest
```

---