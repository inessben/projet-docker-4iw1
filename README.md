# Projet Docker 


## Installation

### 1. Générer les package-lock.json
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
```

### Arrêter les services
```bash
docker compose down
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

Tu dois voir tous les services avec le statut `2/2` :
```
ID        NAME                          MODE        REPLICAS
xxx       e-commerce_frontend           replicated  2/2
xxx       e-commerce_auth-service       replicated  2/2
xxx       e-commerce_product-service    replicated  2/2
xxx       e-commerce_order-service      replicated  2/2
xxx       e-commerce_mongo-auth         replicated  1/1
xxx       e-commerce_mongo-product      replicated  1/1
xxx       e-commerce_mongo-order        replicated  1/1
```

### 8. Accès à l'application
```
http://localhost:8080
```

### Supprimer la stack
```bash
docker stack rm e-commerce
```

---