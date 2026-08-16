# Troubleshooting Runbook

Ovaj dokument opisuje osnovne korake za dijagnostiku problema u Kubernetes deploymentu Ticketing aplikacije.

## 1. Provjera Podova

kubectl get pods -n ticketing

Svi Podovi trebali bi biti u stanju Running i Ready.

Za detalje problema:
kubectl describe pod <pod-name> -n ticketing

# 2. Provjera logovađ

API:
kubectl logs deployment/api -n ticketing

Worker:
kubectl logs deployment/worker -n ticketing

Frontend:
kubectl logs deployment/frontend -n ticketing

PostgresSQL:
kubectl logs deployment/postgres -n ticketing

Redis:
kubectl logs deployment/redis -n ticketing


## 3. Provjera health endpointa

API liveness:
http://localhost/api/healthz

API readiness:
http://localhost/api/readyz

/healthz provjerava radi li API proces, dok /readyz provjerava je li API spreman posluživati zahtjeve i ima li potrebne veze prema ovisnostima.


## 4. Provjera Kubernetes servisa

kubectl get services -n ticketing

Za detalje određenog servisa:
kubectl describe service api -n ticketing

## 5. Provjera Ingressa

kubectl get ingress -n ticketing
kubectl describe ingress -n ticketing

Ako Podovi rade, ali aplikacija nije dostupna preko http://localhost, potrebno je provjeriti Ingress i ingress controller.

## 6. Provjera NetworkPolicy pravila

kubectl get networkpolicy -n ticketing

Za detalje:
kubectl describe networkpolicy -n ticketing

NetworkPolicy može blokirati komunikaciju između Podova ako potrebno allow pravilo nije definirano.


## 7. Rollout problemi

Provjera statusa:
kubectl rollout status deployment/api -n ticketing

Povijest:
kubectl rollout history deployment/api -n ticketing

Ako nova verzija ne radi, moguće je vratiti prethodnu:
kubectl rollout undo deployment/api -n ticketing