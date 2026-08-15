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



Ovaj repozitorij je referentni uzorak aplikacije za kolegij **Uvod u DevOps - DevSecOps**.
Prikazuje cijeli tok: lokalni razvoj kroz Compose i produkcijski deployment kroz Kubernetes manifeste.

## Arhitektura

- `frontend` - web UI za pregled evenata i kupnju karata
- `api` - REST API za evente, narudzbe i health provjere
- `worker` - pozadinska obrada queue poruka
- `postgres` - trajna pohrana narudzbi
- `redis` - queue/cache sloj

### Brza validacija funkcionalnosti

1. Health API:
   ```bash
   curl http://localhost:8080/healthz
   curl http://localhost:8080/readyz
   ```
2. Dohvati evente:
   ```bash
   curl http://localhost:8080/events
   ```
3. Posalji narudzbu:
   ```bash
   curl -X POST http://localhost:8080/tickets/purchase \
     -H "Content-Type: application/json" \
     -d '{"eventId":"evt-1001","customerEmail":"student@example.com","quantity":2}'
   ```
4. Provjeri obradene narudzbe:
   ```bash
   curl http://localhost:8080/tickets/orders
   ```
5. UI:
   - Otvori `http://localhost:3000`

## Sigurnosni elementi

- Multi-stage Docker build i non-root runtime korisnik
- Secret + ConfigMap odvojena konfiguracija
- Liveness/Readiness probe
- Resource requests/limits
- ServiceAccount + RBAC
- NetworkPolicy segmentacija
- Trivy skeniranje slika u CI pipelineu

Detalji skeniranja: `docs/security/image-scan-report.md`