# Schnittstellen-Verträge (Interface Contracts)

Diese Datei legt die verbindlichen Übergabepunkte zwischen **Anwendung (App)**, **Infrastruktur (Infra)** und **Pipeline (CI/CD)** fest. Solange diese Verträge eingehalten werden, können alle drei Bereiche parallel entwickelt und getestet werden.

## 1. Gemeinsamer Tech-Stack & Laufzeit-Vorgaben
- **Anwendung:** Java 21, Spring Boot 3, Maven 3.9+ (eingecheckter Wrapper `./mvnw`), Docker
- **Infrastruktur:** Terraform (`>= 1.10, < 2.0`), AWS Provider (`~> 6.0`)
- **CI/CD:** GitHub Actions
- **Cloud-Umgebung:** AWS Academy Learner Lab
  - Vorgegebene IAM-Rolle (Berechtigungs-Ausweis): `LabRole`
  - Persönliche Test-Umgebungen über die Variable `env_name`

## 2. App ↔ Infrastruktur
- **Datenbank (DynamoDB):** Tabelle `links` mit Partition Key `code` (Typ: `String`)
- **Umgebungsvariablen (von der Infrastruktur an den Container übergeben):**
  - `TABLE_NAME` – Name der DynamoDB-Tabelle (Standard: `links`)
  - `AWS_REGION` – AWS-Region (z. B. `us-east-1`)
  - `DYNAMODB_ENDPOINT` *(optional)* – lokal gesetzt (z. B. `http://dynamodb:8000`), in AWS leer
- **Container-Port:** `8080`
- **Health-Endpunkt:** `GET /actuator/health` (liefert HTTP `200` / `UP` für den ALB)

## 3. App ↔ Pipeline
- **Einheitlicher Build- & Testbefehl:** `./mvnw verify`
- **Container-Bauplan:** `Dockerfile` liegt direkt im Repository-Root (`./Dockerfile`)

## 4. Infrastruktur ↔ Pipeline
- **Terraform-Hauptordner:** `infra/environments/dev`
- **Input-Variable (Pipeline → Terraform):** `image_tag` (Git-Commit-SHA des gebauten Docker-Images)
- **Outputs (Terraform → Pipeline):**
  - `ecr_repository_url` – Adresse des ECR-Paket-Lagers für den Image-Upload
  - `alb_dns_name` – öffentliche Adresse des Load Balancers (ALB) für den Post-Deploy-Test

