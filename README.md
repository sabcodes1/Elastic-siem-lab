# Windows Brute-Force Detection Lab (Elastic SIEM)

Dette prosjektet dokumenterer etableringen av et lokalt SIEM-miljø for monitorering av sikkerhetshendelser på en Windows-maskin. Labben demonstrerer hele verdikjeden fra logginnsamling og normalisering til deteksjonslogikk og aktiv varsling.

## Architecture

* **Endpoint**: Windows 11 Host med **Elastic Agent** kjørende i **Standalone Mode**.
* **SIEM Stack**: Elasticsearch og Kibana deployet via **Docker Desktop** på **WSL2 (Ubuntu)**.
* **Pipeline**: Lokale hendelseslogger blir hentet av agenten, formatert og sendt direkte til Elasticsearch API-et på port 9200 via YAML-konfigurasjon.

## Detection Logic (The SOC Mindset)

Kjernen i prosjektet er en terskelbasert deteksjonsregel (Threshold Rule) utviklet for å identifisere forsøk på passordgjetting:

* **Log Source**: Windows Security Events hentet via `windows.security`-datasettet.
* **Event ID**: Fokusert på **4625** (An account failed to log on).
* **Threshold**: Alarmen utløses ved 3 eller flere feilede påloggingsforsøk innenfor et tidsvindu på 5 minutter.
* **Aggregation**: Regelen grupperer treff på `user.name` for å isolere angrep mot spesifikke brukerkontoer og redusere støy.

## Technical Challenges and Troubleshooting

Gjennomføringen krevde aktiv feilsøking av integrasjonen mellom Windows-host og de virtualiserte containerne:

* **Authentication Conflict**: Identifiserte tidlig `401 Unauthorized`-feilmeldinger under kommunikasjon mellom agent og cluster.
* **Resolution**: Utførte manuell passord-reset av `elastic`-brukeren via `docker exec` inne i Elasticsearch-containeren. Verifiserte API-tilgang ved bruk av PowerShell og `Invoke-RestMethod` før endelig oppdatering av `elastic-agent.yml`.
* **Standalone Optimization**: Valgte Standalone-modus for å få full kontroll over konfigurasjonsfilene og redusere ressursbruk på lokal maskin, fremfor sentralisert Fleet-styring.

## Verification Results

Labben ble verifisert gjennom et kontrollert angrepsløp:

* **Attack Simulation**: Genererte bevisst 4 feilede påloggingsforsøk på Windows-maskinen.
* **Telemetry**: Bekreftet vellykket innsamling av logger i Kibana Discover ved bruk av KQL-spørringen `event.code : "4625"`.
* **Alerting**: SIEM-løsningen korrelerte hendelsene og genererte en **High Severity Alert** som bekreftet at deteksjonslogikken fungerte etter hensikten.

## Project Structure

* **/screenshots**: Dokumentasjon av loggstrømmer, regeldefinisjoner og aktive alarmer.
* **elastic-agent.yml**: Eksempel på konfigurasjonsfil (sensitiv informasjon er fjernet).
