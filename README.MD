# KK-SOF-public
Dette projekt indeholder et testsetup, der kan benyttes til login mod de IdP der er sat op til Københavns Kommune (SOF).

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

Hver mappe indeholder en docker-compose file, der kan benytte til at starte test setup'et.

Det forudsættes at den enkelte leverandør har [Docker Community Edition (CE) (eller en tidligere version)](https://docs.docker.com/engine/installation/) installeret.

For at hente komponenterne skal der først logges ind mod KITs docker registry:
```
åbn en prompt
docker login kitdocker.kvalitetsit.dk
indtast brugernavn og password
```
Derefter kan følgende setup'et startes op

```
cd ~/../KK-SOF-public/SP-testsetup/
docker-compose up

```

For at benytte setup'et i leverandørens egen kontekst skal der laves enkelte ændringer til docker-compose filen.

## Opsætning af docker-compose
Dette afsnit hvilke parametre der kan/skal ændres i docker-compose filen for at benytte det eget setup.

Vedlagte docker-compose fil starter de komponenter som udgør "SAML 2.0 Service Provider". "SAML 2.0 Service Provider"'s opgave er at snakke med IdP'en. Metadata er udvekslet. 

Nedenstående tegning viser hvordan "SAML 2.0 Service Provider" er tænkt ind i levenrandørens miljø. De grønne kasser stilles til rådighed af leverandøren. 
![overview](/images/overview.png)
Det antages, at der står en webserver foran komponenten. Webserveren modtager trafik (requests) fra internettet og router videre til "SAML 2.0 Service Provider". Kald vil blive verificeret og sendes videre til "Login adaptor", som står for at logge brugeren ind i levenrandørens applikationen.

Følgende parametre skal rettes for at "SAML 2.0 Service Provider" fungerer sammen med en webserver og "Login adaptor".

### Webserver
Nedenstående parametre bestemmer lokationen af webserveren som der redirectes til når brugeren har foretaget login. 
Hvis nedenstående miljøvariable ikke ændres vil kald på http://localhost:8080/appa kunne bruges som test url (kan være relevant under udvikling).

```
      - SAML_LB_SCHEME=http           # TODO Webserver (LB), http or https
      - SAML_LB_SERVERNAME=localhost  # TODO Webserver (LB), servername
      - SAML_LB_SERVERPORT=8080       # TODO Webserver (LB), Port 
      - APP_CONTEXT=appa              # TODO contexten for applikationen (SAML_LB_SCHEME://SAML_LB_SERVERNAME:SAML_LB_SERVERPORT/APP_CONTEXT)
```
### LoginAdaptor
Nedenstående skal ændres til at pege på loginadaptoren (eller en usikret applikation uden login). Alt trafik vil blive proxy'et videre til ENDPOINT_URL når brugeren er logget ind. 
```
      - ENDPOINT_URL=http://app-a:8080/       # TODO Url på applikation (Login Adaptor), localhost/127.0.0.1 can NOT be used
```

### Fremsøg brugeroplysninger fra LoginAdaptor

Det er loginproxys (defineret via ENDPOINT_URL) opgave at trække data ud fra "SAML 2.0 Service Provider". Nedenstående skitserer dette.

![overview](/images/userdata.png)

For at "Login Adaptoren" kan fremsøge data, skal "login adaptoren":
2. Loginadaptoren skal parse den cookie der sendes med i request’et for at finde attributten SESSION.
3. Værdien af SESSION sættes som headerinformation i et GET kald til http://localhost:8080/getsessiondata

/getsessiondata giver en JSON struktur tilbage, som kan se ud som følger:

![json](/images/json.png)






