# MyDentist

## Getting Started

```bash
npx expo start
```

In the output, you'll find options to open the app in a

- [development build](https://docs.expo.dev/develop/development-builds/introduction/)
- [Android emulator](https://docs.expo.dev/workflow/android-studio-emulator/)
- [iOS simulator](https://docs.expo.dev/workflow/ios-simulator/)

### Starting Over

If you make mistakes or break things, simply run this get back to a working state.

```bash
npm run reset-project
```

## Authentication Basics

### Standard Flow

Below is a pretty standard sequence diagram showing the flow of credentials and data for a Entra-based user. This has been extended to include the Workday API to add the context.

```mermaid
sequenceDiagram
    autonumber

    actor User as Mobile App User
    participant App as Mobile App
    participant Entra as Microsoft Entra ID
    participant API as Worker API (/worker)
    participant WD as Workday

    User->>App: Enter credentials / SSO
    App->>Entra: OAuth2/OIDC Authorization Request

    Entra-->>App: ID Token + Access Token

    Note over App,Entra: ID Token contains user identity claims

    App->>App: Extract employee identifier<br/>(email, UPN, employeeId, oid)

    App->>API: GET /worker?email=mr.dentist@mydentist.co.uk<br/>Authorization: Bearer access_token

    API->>WD: Lookup worker by email

    WD-->>API: Worker record

    API-->>App: Worker JSON payload

    App->>App: Store WorkerId

    App->>API: GET /worker/123456
    API->>WD: Retrieve worker details by WorkerId

    WD-->>API: Detailed worker profile
    API-->>App: Worker JSON payload

    App-->>User: Personalized application experience
```
