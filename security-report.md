# Container Security Scan Report

## Tool

Za sigurnosno skeniranje Docker imageova korišten je Trivy.

Skeniranje je integrirano u GitHub Actions CI pipeline i izvršava se nad production imageovima:

- ticketing-api
- ticketing-worker
- ticketing-frontend

## Initial findings

Prilikom početnog skeniranja pronađene su HIGH i CRITICAL ranjivosti.

Većina nalaza dolazila je iz npm alata koji je bio uključen u Node.js runtime image, a nije bio potreban za pokretanje aplikacije.

## Remediation

Dockerfileovi su izmijenjeni tako da koriste multi-stage build.

Production dependencies instaliraju se u zasebnom build stageu, dok finalni runtime image sadrži samo komponente potrebne za pokretanje aplikacije.

Iz runtime imagea uklonjeni su npm i npx:

RUN rm -rf /usr/local/lib/node_modules/npm \
    && rm -f /usr/local/bin/npm /usr/local/bin/npx

Aplikacija se u production containeru pokreće direktno pomoću Node.js:
CMD ["node", "src/server.js"]

Worker koristi:
CMD ["node", "src/worker.js"]

Production containeri također se izvršavaju kao non-root korisnik:
USER node

## Result

Nakon izmjena Docker imageovi su ponovno izgrađeni i skenirani.

Trivy security scan za production imageove prolazi CI security quality gate bez HIGH i CRITICAL ranjivosti.

Development imageovi odvojeni su od production runtime imageova. Development stage sadrži npm i Nodemon radi lokalnog hot-reloada, dok se taj stage ne koristi za production deployment.


## Security quality gate

CI pipeline je konfiguriran tako da HIGH ili CRITICAL ranjivost uzrokuje neuspjeh security scan koraka.

Time se sprječava objavljivanje production imagea koji ne zadovoljava definirani sigurnosni prag.


## Verzioniranje Docker imageova

Docker imageovi objavljuju se s dva taga:

- latest označava najnoviji uspješni build s main grane.
- Git commit SHA koristi se kao jedinstveni tag koji omogućuje povezivanje Docker imagea s točnim commitom iz kojeg je izgrađen.
