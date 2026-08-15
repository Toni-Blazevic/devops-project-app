# Secure Event Ticketing Platform (Sample DevSecOps Project)

Projekt predstavlja višeslojnu aplikaciju za prodaju ulazinica i sastoji se od sljedećih servisa:

 - Frontend
 - API
 - Backend worker
 - Redis
 - PostgresSQL

## Lokalni deployment

Za lokalni deployment koristi se Docker Compose

### Preduvijeti

Potrebno je imati instalirano:

 - Docker destop
 - Docker compose

## Pokretanje aplikacije

Cijeli stacki se pokrece jednom naredbom:

docker compose up --build

Nakon pokretanja:

 - Frontend: http://localhost:3000
 - API: http://localhost:8080

API helth endpoint:

http://localhost:8080/healthz

## Hot reload

API, frontend i worker koriste Nodemon u development okruženju.
Source direktoriji su mountani u containere preko Docker volumea, pa se promjene koda automatski detektiraju


## PostgreSQL

PostgresSQL podatci se spremanju u Docker volumen:

 - postgres_data

Zbog toga se podatci zadrzavaju i nakon gašenja containera.


## Zaustavljanje aplikacije

Stack se zaustavlja naredbom:

 - docker compose down

ili

 - docker compose down -v

ako se želi zaustaviti volumen te izbrisati PostgresSQL podatci.

## Funkcijonalna validacija

Nakon pokretanja aplikacija se može provjeriti health endpointom:

curl http://localhost:8080/healthz

Osnovni workflow aplikacije:


1. Korisnik otvara frontend.
2. Odabire event i kupuje kartu.
3. Frontend šalje zahtjev API servisu.
4. API stavlja narudžbu u Redis queue.
5. Worker preuzima narudžbu.
6. Worker sprema obrađenu narudžbu u PostgreSQL

## Kubernetes deployment

Kubernetes manifesti nalaze se u direktoriju:

infra/k8s/

Struktura uključuje Deployment i Service manifeste za API, frontend, PostgreSQL i Redis, Worker Deployment, ConfigMap, Secret, PersistentVolumeClaim, Ingress, NetworkPolicy i ServiceAccount objekte.

##Deploy aplikacije

Sve manifeste moguće je prijmejniti rekurzivno:
kubectl apply -f infra/k8s/namespace.yaml
kubectl apply -R -f infra/k8s

Provjera Podova:
kubectl get pods -n ticketing

Provjera servisa:
kubectl get services -n ticketing

Provjera Ingressa:
kubectl get ingress -n ticketing

Provjera NetworkPolicy pravila:
kubectl get networkpolicy -n ticketing

Provjera ServiceAccount objekata:
kubectl get serviceaccounts -n ticketing


## Pristup aplikaciji

Nakon uspješnog deploymenta frontend je dostupan preko Ingressa:
http://localhost

API health endpoint:
http://localhost/api/healthz


## Rolling update

Status rollouta API Deploymenta:
kubectl rollout status deployment/api -n ticketing

Povijest revizija:
kubectl rollout history deployment/api -n ticketing

##Rollback

Povratak na prethodnu reviziju:
kubectl rollout undo deployment/api -n ticketing

Nakon rollbacka:
kubectl rollout status deployment/api -n ticketing

