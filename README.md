#  "Up time checks"

Det er god praksis å bruke verktøy som overvåker applikasjonen "eksternt" og rapporterer i ulike kanaler(epost, slack, telefon) dersom applikasjonen ikke oppfører seg som forventet, eller feiler. Målet med en slik overvåkning er å oppdage feil *minst* samtidig som brukere. 

## Statuscake

Statuscake er et verktøy for ekstern overvåkning av websider. Det finnes også andre, for eksempel "pingdom". Amazon AWS har sin egen tjeneste "Canary" og Google Cloud platform har en tjeneste som heter "uptime checks".

I denne øvingen skal vi bli kjent med Statuscake, fordi dette gir oss en mulighet å bruke en ny "provider" og samtidig til å øve på hemmeligheter i Travis og miljøvariabler. Hvis dere blir ferdig med statuscake øvingen, forsøk gjerne med - https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/monitoring_uptime_check_config

Det første dere må gjøre er å lage en ny konto på statuscake.com

## Statuscake contact group

En Contact group representerer en eller flere personer som blir kontaktet dersom ikke endepunktet svarer med HTTP/200 når statuscake sine servere tester.     

* Lag en ny Contact Group
* Forsøk å legge til telefonnumer og epost i Default Contact Group. 
* Deretter Kan du endre terraformkoden, slik at *website_url* ikke finnes. Etter noen minutter skal du da bli kontaktet på telefon eller epost. 

Endre terraform koden ved å legge til en referanse til "statuscake_test"

## Terraform 

Ta utgangspunkt i Terraform repository du har laget for GCP + terraform. Hvis du har problemer med den oppgaven og ennå ikke er i mål, fokuser på å få den til å fungere før du fortsetter. 

Vi skal nå legge til en ny provider, og en en ny ressurs som vil overvåke HTTP endepunktet for applikasjonen vår. 

Lag en ny fil, for eksmpel ved navn "statuscake.tf"

```
provider "statuscake" {
  username = "postglennbechno"
}

resource "statuscake_test" "googlecloudruntest" {
  website_name = "My test"
  website_url  = google_cloud_run_service.hello.status[0].url
  test_type    = "HTTP"
  check_rate   = 300
  contact_id   = 12345
}
```
*OBS!* Legg merke til website_url. Her må dere nedre "hello" til å stemme overens med ressursnavnet til GCP tjenesten. Eksempelvis 
```
resource "google_cloud_run_service" "hello" {
  name = "helloworld-service"
  location = "us-central1"
  project = var.project_id
```
Her der dere at ressursen av av typen google_cloud_run_service, og heter "hello".

Hvis dere leser i dokumentasjonen til Statuscake sin Terraform provider - https://www.terraform.io/docs/providers/statuscake/index.html
Ser dere at man i _provder_ HCL blokken kan oppgi både username og apikey. Men, det vet vi *ikke* er smart (!) Så, heldig hvis kan vi også bruke en 
miljøvariabel med navn _STATUSCAKE_APIKEY_  

## Travis

Vi må gjøre miljøvariabelen _STATUSCAKE_APIKEY_ tilgjengelig for byggeprosessen, og det gjør vi enkelt og greit med ... 

```sh
travis encrypt STATUSCAKE_APIKEY=abcde1234 --add
```

API nøkkel for statuscake får dere ved å finne en bitteliten "nedoverpil" opp til høyre i brukergrensesnittet. og velge "my account". Nederst på siden finner dere funksjoner for å lage/fjerne API nøkler. 

Sjekk inn .travis.yml etter du har gjort en travis encrypt og push til master. Se at du får en ny sjekk i statuscake. 

 

```
...
  contact_group = ["Default"]
...
```

## Slack integrasjon 

velg "Integrations" fra hovedmenyen og legg til en Slack Integrasjon. For webhook kan dere bruke følgende Webhook URL
```
https://hooks.slack.com/services/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	
```
Gå deretter tilbake til Default Contact group og legg til integrasjonen med slack.  Dette vil sende meldinger til #ops kanalen i PGR301 Slack workspacet vi har opprettet for semesteret, dersom tjenesten går ned. 


