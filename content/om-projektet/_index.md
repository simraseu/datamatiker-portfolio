---
title: "Om Projektet"
weight: 2
description: "Kontekst: The Way of Coherence og min rolle i systemarkitekturen"
---

# The Way of Coherence

Dette projekt tager udgangspunkt i sundhedsplatformen **The Way of Coherence**. Platformen fungerer som den fælles ramme for vores team, hvor vi har arbejdet med en klar arbejdsdeling mellem fælles applikationslogik og specialiseret infrastruktur.

For at maksimere læringsudbyttet har jeg indtaget rollen som **Backend Specialist** med ansvar for optimering af data-infrastrukturen, persistens og implementation af analytics-laget.

---

## Den Fælles Systemarkitektur

Systemet er en sundhedsplatform bygget som en **Blazor Server** applikation. Arkitekturen er designet med fokus på høj modularitet og testbarhed gennem **Separation of Concerns**.

### Det Logiske View (4+1 Arkitekturmodellen)
Med afsæt i pensums **Systemarkitektur og 4+1 Views**, er systemet struktureret i distinkte lag:

1.  **Application:** Indeholder use cases og forretningslogik. Her anvender vi **Dependency Injection** til at koble logikken løst til infrastrukturen.
2.  **Domain:** Rummer domænemodeller og forretningsregler (Kernen).
3.  **Infrastructure:** Håndterer dataadgang og integrationer. *(Det er primært i dette lag, min specialisering finder sted).*
4.  **Blazor Web App (Presentation):** Håndterer UI via Razor components, struktureret efter **MVC-principper**, hvor view-logik er adskilt fra data.

### Designvalg & Mønstre
* **Feature-baseret organisering (Vertical Slices):** Selvom vi har logiske lag, er koden organiseret i vertikale slices per feature for at samle relateret logik og sikre modularitet.
* **Sikkerhed:** Platformen benytter **ASP.NET Identity** til autentificering og rollebaseret adgangskontrol.
* **Integration:** Systemet agerer som **API Consumer** mod en ekstern Python AI-service.

---

## Min Rolle: Infrastruktur & Integration

Mit primære fokus har været at sikre systemets **Non-Functional Requirements** (Performance, Scalability, Privacy) i krydsfeltet mellem infrastrukturen og den eksterne AI-service.

Hvor resten af teamet har fokuseret på feature-udvikling, har min opgave været at designe og validere de kritiske data-komponenter:

### 1. Database & Storage (Infrastruktur)
Jeg har udfordret standard-løsningen ved at implementere en **Unified Monolith** strategi med **PostgreSQL**. Målet var at minimere kompleksiteten ved distribuerede systemer og håndtere vektordata lokalt for at reducere latency.
👉 *[Læs analysen her]({{< relref "database/_index.md" >}})*

### 2. Data Science & Analytics (Metode)
Jeg har designet en **Privacy-First pipeline** for integrationen til Python-servicen. Her har fokus været på datasikkerhed og anonymisering i overensstemmelse med **NIST-frameworket**.
👉 *[Læs analysen her]({{< relref "data-science/_index.md" >}})*

---

## Metode: Fra HLD til LLD

Min arbejdsproces har fulgt en struktureret systemudviklingsmetode.

> **Metodisk definition:** Jeg anvender **arkitekturevaluering** som *teknik* (aktiviteten), litteraturbaseret research som *input*, og Proof of Concept som *værktøj* til at validere mine valg.

Processen er forløbet således:
1.  **High Level Design (HLD):** Fastlæggelse af arkitekturstrategi og teknologivalg (Postgres vs. Mongo).
2.  **Low Level Design (LLD):** Detaljeret design af databaseschema og API-kontrakter.
3.  **Implementation:** Udvikling af komponenter via C# og SQL.

**Næste skridt:** Se de specifikke kompetencemål for min rolle 👉 [Læringsmål]({{< relref "laeringsmaal/_index.md" >}})