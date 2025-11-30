---
title: "Om Projektet"
weight: 2
description: "Kontekst: The Way of Coherence og min specialist-rolle i arkitekturen"
---

# The Way of Coherence

Dette projekt tager udgangspunkt i sundhedsplatformen **The Way of Coherence**. Platformen har fungeret som den fælles ramme for vores team, hvor vi har arbejdet ud fra en klar fordeling mellem fælles arkitektur og individuel specialisering.

For at maksimere læringsudbyttet har jeg i projektet indtaget en rolle som **specialiseret konsulent**, med fokus på at optimere data-infrastrukturen og analytics-laget inden for den fælles ramme.

---

## Den Fælles Systemarkitektur

Systemet er en sundhedsplatform bygget i Blazor Server, hvor brugerne får adgang til rollebaserede funktioner og interaktioner.

Vi har opbygget systemet efter en kombination af Clean Architecture[^1] og CQRS, hvilket giver en tydelig struktur og adskillelse mellem kernefunktionalitet, logik og brugergrænsefladen.

### Projektets 4 Hovedlag
Projektet er organiseret i fire distinkte lag, der sikrer "Separation of Concerns":

1.  **Application:** Indeholder applikationslogik, use cases, MediatR-handlers samt Commands og Queries, der styrer flowet gennem systemet.
2.  **Domain:** Rummer domænemodeller, forretningsregler og domænehændelser, som udgør platformens kerne.
3.  **Infrastructure:** Står for dataadgang, repositories, integrationer og kommunikation til eksterne services. *(Det er primært her, min specialisering finder sted).*
4.  **Blazor Web App:** Leverer UI’et og håndterer interaktioner mellem brugeren og systemet.

### Designvalg
* **CQRS:** Sikrer en klar opdeling mellem læsning og skrivning.
* **Vertical Slices:**[^2] Gør at hver feature (fx posts, medlemskaber eller profiler) er samlet ét sted med sin egen logik.
* **Sikkerhed:** Platformen bruger ASP.NET Identity til login og rollebaseret adgangskontrol.
* **Persistering:** Entity Framework Core håndterer lagring af brugere og interne data.

---

## Min Rolle: Infrastruktur & Integration

Som beskrevet ovenfor integreres systemet med en Python-baseret AI-service via HTTP-gateways, hvilket gør det muligt at udvide funktionaliteten med intelligent assistence.

Det er netop i krydsfeltet mellem **Infrastructure-laget** og denne **eksterne AI-service**, at jeg har lagt mit fokus.

Hvor resten af teamet har fokuseret på at etablere systemarkitekturen, sikkerheden og selve AI-servicen, har min opgave været at validere og optimere de kritiske data-komponenter, der skal bære systemet i fremtiden:

### 1. Database & Storage (Specialisering)
Selvom standard-implementationen benytter Entity Framework Core, har jeg undersøgt og implementeret en **Unified Monolith** strategi med **PostgreSQL**.
Formålet var at bevise, at vi kan håndtere de komplekse vektordata fra AI-servicen mere effektivt direkte i databasen, frem for gennem eksterne afhængigheder.
👉 *[Læs analysen her]({{< relref "database/_index.md" >}})*

### 2. Data Science & Analytics (Specialisering)
I forbindelse med integrationen til Python-servicen har jeg designet en **Privacy-First pipeline**. Min rolle var at sikre, at dataen der flyder gennem HTTP-gatewayen, bliver anonymiseret og målt korrekt, så vi kan vurdere AI-kvaliteten uden at bryde GDPR.
👉 *[Læs analysen her]({{< relref "data-science/_index.md" >}})*

---

## Metode: Konsulent-tilgangen

Ved at definere mig selv som specialist inden for teamet, har jeg kunnet arbejde dybdegående med teknologier (PostgreSQL, Vector Search, Hashing-algoritmer), der ligger "under motorhjelmen" på den fælles arkitektur.

Min leverance til teamet er derfor ikke bare kode, men validerede arkitektur-beslutninger (Proof of Concepts), der er klar til at blive rullet ud i Infrastructure-laget.

**Næste skridt:** Se de specifikke mål, jeg satte for min specialist-rolle 👉 [Læringsmål]({{< relref "laeringsmaal/_index.md" >}})

---
## Referencer

[^1]: Martin, R. C. (2017). "Clean Architecture: A Craftsman's Guide to Software Structure". Prentice Hall.
[^2]: Bogard, J. (2018). "Vertical Slice Architecture". JimmyBogard.com.