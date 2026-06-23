# Arkitektur

Et teknisk dypdykk i hvordan henami er bygget.

## Oversikt

henami består av to klienter som deler én backend:

```
┌─────────────────┐     ┌─────────────────┐
│  Web (React)    │     │  iOS (Expo)     │
│  Vite · Vercel  │     │  React Native   │
└────────┬────────┘     └────────┬────────┘
         │                       │
         └───────────┬───────────┘
                     ▼
         ┌───────────────────────┐
         │       Supabase        │
         │  Auth · Postgres ·    │
         │  RLS · Edge Functions │
         └───────────┬───────────┘
                     ▼
            ┌─────────────────┐
            │ Anthropic Claude│
            └─────────────────┘
```

## Frontend (web)

- **React 19 + Vite** – rask HMR i utvikling, optimalisert produksjonsbygg.
- **Kodesplitting** – hver side lastes som sin egen chunk via `React.lazy`. Spesielt viktig for tunge avhengigheter som Mapbox (kart) og Recharts (grafer), så de ikke lastes ned før man faktisk besøker siden.
- **Ruting** – React Router v7, all ruting bak en auth-gate.
- **Tema** – et eget tema-system basert på CSS-variabler på `<html>`. Lar brukeren tilpasse farger live, lagret i `localStorage`.

## Mobil (iOS)

- **React Native + Expo (SDK 54)** med **Expo Router** for filbasert ruting.
- Gjenbruker ren forretningslogikk (dato-hjelpere, beregninger) fra web, men har egne UI-komponenter (`View`/`Text` i stedet for DOM).
- Auth-session lagres i `AsyncStorage`; bygges og distribueres via EAS / TestFlight.

## Backend (Supabase)

- **Postgres** med **row-level security** på alle tabeller. Mønsteret er `auth.uid() = bruker_id`, så hver bruker kun ser sine egne rader.
- **Deling** – funksjoner som delt kalender og felles økonomi løses med egne `*_deling`-tabeller, slik at man kun eksponerer det man eksplisitt deler.
- **Trigger** – ny bruker oppretter automatisk en profil-rad.
- **Edge Functions (Deno)** – server-side funksjoner som henter markedsdata (RSS + Oslo Børs) og kaller Anthropic Claude for å generere markedsanalyse. Den ene kjører på forespørsel fra klienten, den andre på timeplan med service-role-nøkkel.

## Hvorfor disse valgene?

| Valg | Begrunnelse |
|---|---|
| Supabase | Auth, database og serverless-funksjoner i én pakke – rask å bygge på som soloutvikler |
| RLS i databasen | Sikkerhet håndheves i databasen, ikke i klientkoden – vanskeligere å omgå |
| Expo (ikke bare React Native CLI) | TestFlight/App Store-distribusjon via EAS uten lokal Xcode-konfigurasjon |
| CSS-variabler for tema | Live fargebytte uten re-render av React-treet |
| Edge functions for AI | API-nøkkelen holdes server-side, aldri eksponert i klienten |
