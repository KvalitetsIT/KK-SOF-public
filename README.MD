# KK-SOF-public
Dette projekt indeholder testsetups der kan benyttes til at login mod de IdP der er sat op til Københavs kommune (SOF).

Strukturen for projektet er som følger.
<pre>
 ┬  
 └ SP-testsetup  
     ┬  
     ├ EG   
     ├ LP
     └ MM
</pre>

EG, LP og MM refererer til de tre projektdeltagere EG, Life-partners og Mobilize Me.

Hver mappe indeholder en docker-compose file der kan benytte til at starte test setup'et.

Det forudsættes at den enkelte leverandør har [Docker Community Edition (CE) (eller tidligere version)](https://docs.docker.com/engine/installation/) installeret.

For at hente komponenterne skal der først logges ind mod KITs docker registry:
```
åbn en prompt
docker login
indtast brugernavn og password
```
Du er nu klar til at benytte test setup'et

```
cd ~/../KK-SOF-public/SP-testsetup/
docker-compose up

```

For at kunne benytte dette setup i leverandørens egen kontekst skal der laves enkelte ændringer til docker-compose filen.

## Opsætning af docker-compose
Dette afsnit hvilke parametre der kan/skal ændres i docker-compose filen for at benytte eget setup.
![overview](/images/overview.png)


For at kunne benytte testsetup'et som en del af levenrandørens eget setup skal docker-compose filen rettes til.





