# Authentication Flows

## Initial Login & Onboarding

This is the initial user login experience via Workday. They will be directed to Workday, enter their username and password and then referred back to the mobile app to begin onboarding via the app.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant App as Mobile App
    participant Browser as System Browser
    participant Workday as Workday SAML SP
    participant Entra as Microsoft Entra ID
    participant API as Backend API

    User->>App: Opens app after receiving link
    App->>API: Check onboarding link / invite
    API-->>App: Requires enterprise login
    App->>Browser: Launch SAML login

    Browser->>Workday: Start SAML sign-in
    Workday->>Entra: Redirect SAML AuthnRequest
    Entra-->>Browser: Render login form

    User->>Browser: Username + password
    Browser->>Entra: Submit credentials
    Entra->>Entra: Validate credentials

    alt Credentials valid
        Entra-->>Workday: SAML assertion
        Workday-->>Browser: Redirect to mobile deep link
        Browser->>App: Return auth result
        App->>API: Create authenticated mobile session
        API-->>App: Session token + onboarding status
    else Credentials invalid
        Entra-->>Browser: Show authentication error
        Browser-->>User: Retry login
    end
```

---

## Resuming App Auth Flow

Once the user has finished using the app, they will inevitably go away for a while eventually coming back to pick up where they left off. This is the auth process that will follow...

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant App as Mobile App
    participant API as Backend API
    participant IdP as Entra ID / Identity Provider
    participant MFA as MFA Provider

    User->>App: Opens app later
    App->>API: Call protected endpoint with session token

    alt Session valid
        API-->>App: Protected data
    else Session expired
        API-->>App: 401 reauthentication required
        App->>IdP: Start reauth flow
        IdP->>MFA: Require second factor
        MFA-->>User: MFA challenge
        User->>MFA: Approves challenge
        MFA-->>IdP: MFA success
        IdP-->>App: New auth result
        App->>API: Exchange auth result
        API-->>App: New session token
    end
```

---

## Standard High-level Auth Flow

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

---

## End Session (Logout) Flow

Once the user has finished their session they can choose to logout of the app. This is what happens:

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant App as Mobile App
    participant API as Backend API
    participant Browser as System Browser / Auth Session
    participant Workday as Workday SAML SP
    participant Entra as Microsoft Entra ID

    User->>App: Taps Sign out
    App->>API: Revoke mobile app session / refresh token
    API-->>App: App session revoked

    App->>Browser: Open SAML logout URL

    Browser->>Workday: Send SAML LogoutRequest
    Workday->>Workday: Clear Workday session
    Workday->>Entra: Redirect / POST SAML LogoutRequest

    Entra->>Entra: Clear Entra SSO session where applicable

    alt Logout succeeds
        Entra-->>Workday: SAML LogoutResponse
        Workday-->>Browser: Redirect to app logout callback
        Browser->>App: Open logout callback deep link
        App->>App: Clear local tokens, cookies, cached user state
        App-->>User: Show signed-out state
    else IdP logout fails or is partial
        Entra-->>Browser: Logout error / session may remain active
        Browser->>App: Return logout failure or timeout
        App->>App: Clear local app state anyway
        App-->>User: Show signed-out locally
    end
```
