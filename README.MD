# KK-SOF-public
Dette projekt indeholder to testsetups:

- Et, der kan benyttes til login mod de IdP der er sat op til Københavns Kommune (SOF).
- Et, der kan foretages til at foretage sikrede REST-kald til de fhir-services der er sat op til Københavns Kommune (SOF).

Strukturen for projektet er som følger.
<pre>
 ┬  
 ├ fhirservice_client
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
indtast brugernavn og password (fremsendt i mail)
```
Derefter kan følgende setup'et startes op

```
cd ~/../KK-SOF-public/SP-testsetup/EG|LP|MM
docker-compose up

```

Eller


```
cd ~/../KK-SOF-public/fhirservice_client/
docker-compose up

```

For at benytte "SP-testsetup" i leverandørens egen kontekst skal der laves enkelte ændringer til docker-compose filen.

## Opsætning af docker-compose (SP-testsetup)
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

### Flytning mellem leverandørens miljøer
Hvis leverandøren vil have loginflowet til at køre på andet end localhost (eksempelvis på en testserver), skal følgende gøres:
- Flytte docker-compose setup over på testserveren
- Ændre "SP_ENTITY_ID" til noget andet f.eks. SP_ENTITY_ID=test:xx:voresapp
- Ændre SAML_LB_SERVERNAME til f.eks. test.xx.dk (Ved at ændre properties, kan der kan også køres https og trafikken kan flyttes til en anden port).
- Skriv til KIT vedrørende URL til metadata. Det kan være "http://test.xx.dk:8080/appa/saml/metadata" (metadata bliver automatisk udstillet ved opstart af dockercompose filen). 

Når de nye metadata er uploaded, vil det nye setup virke. 

## Brug af docker-compose (fhirservice_client)

Docker-compose'n består af tre images:

- Webservice comsumer (wsc) - som alle kald foretages igennem
- Mongo - letvægts-database som cacher tokens trukket fra STS
- Stsclient - hjælpe-komponent til at trække tokens fra STS (Secure Token Service)

Når docker-compose er startet op, vil man kunne kalde wsc´en på localhost:9090 (denne port kan konfigureres ved port-mapningen på sidste linje), hvorefter den vil trække et sikkerheds-token fra vores STS (konfigureret ved STS_WSDL_URL), og herefter foretage et sikret kald videre til https://fhirservice.kkff-api.kvalitetsit.dk/fhirservice/ (konfigureret ved ENDPOINT_URL).

I en drift-situation, bør wsc´en deployes på det kaldende systems servermiljø således at kun det kaldende system´s serverkode kan komme til at kalde wsc´en.

docker-compose setuppet indeholder også en mongo-database, som cacher de sikkerheds-tokens som der trækkes fra STS. Man bør derfor (i alle REST-kald igennem wsc) angive en http-header SESSION indeholdende et sessions-id - da det er på dette sessions-id at det cachede sikkerheds-token vil blive knyttet op.

For at skifte hvilket endpoint man kører op imod skal ENDPOINT_URL udskiftes

For at skifte hvilken STS man kører op imod:

- STS_WSDL_URL skal ændres
- STS´ens certifikat placeres i volume og refereres til med STS_TRUST_CERTIFICATE
- Det certifikat og den privatnøgle som man identificerer sig med overfor STS skal placeres i volume og refereres til med STS_CLIENT_CERTIFICATE og STS_CLIENT_KEY




