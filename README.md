# Open Game System (OGS) Specification v1

This repository contains the official specification for the Open Game System (OGS), defining the protocols and requirements for integrating with the OGS ecosystem.

## Table of Contents

- [Introduction](#introduction)
- [Core Specifications](#core-specifications)
  - [Account Linking Protocol](#account-linking-protocol)
  - [Push Notification Protocol](#push-notification-protocol)
  - [TV Casting Protocol](#tv-casting-protocol)
- [Certification Process](#certification-process)
- [Version History](#version-history)

## Introduction

The Open Game System specification defines the protocols and APIs that allow web games to access native capabilities through the OGS ecosystem. This specification is designed to be:

- **Web-First**: Built for games that live primarily on the web
- **Protocol-Based**: Focused on defining clear communication protocols
- **Independent**: Allows games to maintain their own identity and authentication
- **Extensible**: Designed to accommodate future capabilities

The specification is implemented by the OGS platform and SDKs, which follow these protocols to enable cross-platform features.

## Core Specifications

The OGS v1 specification includes three core protocols:

### Account Linking Protocol

The Account Linking Protocol enables games to link their independent user accounts with the OGS platform, enabling cross-platform features like push notifications while maintaining authentication independence.

#### Protocol Overview

1. Game server requests a link token from the OGS platform
2. User is redirected to the OGS platform's link account page with the link token
3. User authenticates with the OGS platform and confirms the link
4. OGS platform links the accounts and redirects back to the game
5. Game server verifies the link status with the OGS platform

#### HTTP Sequence for Account Linking

```
┌──────────┐                      ┌──────────┐                      ┌──────────┐
│   Game   │                      │    OGS   │                      │   User   │
│  Server  │                      │ Platform │                      │  Browser │
└────┬─────┘                      └────┬─────┘                      └────┬─────┘
     │                                 │                                 │
     │ 1. Request Link Token           │                                 │
     │ ────────────────────────────>   │                                 │
     │                                 │                                 │
     │ 2. Return Link Token & URL      │                                 │
     │ <────────────────────────────   │                                 │
     │                                 │                                 │
     │ 3. Redirect to Link URL         │                                 │
     │ ─────────────────────────────────────────────────────────────>   │
     │                                 │                                 │
     │                                 │ 4. User Authenticates          │
     │                                 │ <────────────────────────────  │
     │                                 │                                 │
     │                                 │ 5. User Confirms Link          │
     │                                 │ <────────────────────────────  │
     │                                 │                                 │
     │                                 │ 6. Redirect to Game Callback   │
     │                                 │ ────────────────────────────>  │
     │                                 │                                 │
     │ 7. Verify Link Status           │                                 │
     │ ────────────────────────────>   │                                 │
     │                                 │                                 │
     │ 8. Return Link Information      │                                 │
     │ <────────────────────────────   │                                 │
     │                                 │                                 │
```

#### Required API Endpoints

##### Request Link Token (Game Server → OGS Provider)

```http
POST /api/v1/auth/account-link-token HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer GAME_API_KEY

{
  "gameUserId": "user-123",
  "redirectUrl": "https://yourgame.com/auth/callback"
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "linkToken": "xyz123",
  "expiresAt": "2024-06-30T20:00:00Z",
  "linkUrl": "https://opengame.org/link-account?token=xyz123"
}
```

##### Verify Link Token (Game Server → OGS Provider)

```http
POST /api/v1/auth/verify-link-token HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer GAME_API_KEY

{
  "token": "xyz123"
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "valid": true,
  "userId": "ogs-user-456",
  "email": "user@example.com",
  "status": "linked",
  "linkedAt": "2024-06-29T15:35:00Z"
}
```

#### Web Auth Token Protocol

The Web Auth Token Protocol enables seamless single sign-on between the OGS platform and games, allowing users to access games through the OGS app without re-authenticating.

##### Protocol Overview

1. OGS app requests a web auth code from the OGS platform
2. OGS app opens the game in a WebView with the code as a parameter
3. Game server verifies the code with the OGS platform
4. OGS platform returns user information
5. Game creates a session for the user

##### HTTP Sequence for Web Auth Token

```
┌──────────┐                      ┌──────────┐                      ┌──────────┐
│  OGS App │                      │    OGS   │                      │   Game   │
│          │                      │ Platform │                      │  Server  │
└────┬─────┘                      └────┬─────┘                      └────┬─────┘
     │                                 │                                 │
     │ 1. Request Web Auth Code        │                                 │
     │ ────────────────────────────>   │                                 │
     │                                 │                                 │
     │ 2. Return Web Auth Code         │                                 │
     │ <────────────────────────────   │                                 │
     │                                 │                                 │
     │ 3. Open Game URL with Code      │                                 │
     │ ─────────────────────────────────────────────────────────────>   │
     │                                 │                                 │
     │                                 │ 4. Verify Web Auth Code         │
     │                                 │ <────────────────────────────  │
     │                                 │                                 │
     │                                 │ 5. Return User Information      │
     │                                 │ ────────────────────────────>  │
     │                                 │                                 │
     │                                 │ 6. Create Game Session         │
     │                                 │ <───────────────────────────   │
     │                                 │                                 │
```

##### Required API Endpoints

###### Generate Web Auth Code (OGS App → OGS Provider)

```http
POST /api/v1/auth/web-code HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer SESSION_TOKEN
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "code": "xyz123",
  "expiresIn": 300
}
```

###### Verify Web Auth Token (Game Server → OGS Provider)

```http
POST /api/v1/auth/verify-token HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer GAME_API_KEY

{
  "token": "xyz123"
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "valid": true,
  "ogsUserId": "ogs-user-456",
  "email": "user@example.com",  // Only included if verified
  "isVerified": true
}
```

#### Token Refresh Protocol

The Token Refresh Protocol enables games to refresh expired session tokens using a refresh token.

##### Required API Endpoints

###### Refresh Token (Game Server → OGS Provider)

```http
POST /api/v1/auth/refresh HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Cookie: refresh=refresh-token

{
  "refreshToken": "refresh-token"
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "success": true,
  "sessionToken": "new-jwt-token",
  "expiresIn": 900
}
```

### Push Notification Protocol

Push notifications allow games to engage users even when they're not actively playing. The Push Notification Protocol defines how games can send notifications to users through the OGS platform.

#### Protocol Overview

1. Game server sends a notification request to the OGS platform
2. OGS platform verifies the game's API key and the account link
3. OGS platform delivers the notification to the user's device
4. OGS platform returns the delivery status to the game server
5. Game server can later check the notification status (delivered, read, etc.)

#### HTTP Sequence for Sending Notifications

```
┌──────────┐                      ┌──────────┐                      ┌──────────┐
│   Game   │                      │    OGS   │                      │   User   │
│  Server  │                      │ Platform │                      │  Device  │
└────┬─────┘                      └────┬─────┘                      └────┬─────┘
     │                                 │                                 │
     │ 1. Send Notification            │                                 │
     │ ────────────────────────────>   │                                 │
     │                                 │                                 │
     │ 2. Validate Request            │                                 │
     │ <───────────────────────────   │                                 │
     │                                 │                                 │
     │                                 │ 3. Deliver Notification         │
     │                                 │ ────────────────────────────>  │
     │                                 │                                 │
     │ 4. Return Delivery Status       │                                 │
     │ <────────────────────────────   │                                 │
     │                                 │                                 │
     │ 5. Check Notification Status    │                                 │
     │ ────────────────────────────>   │                                 │
     │                                 │                                 │
     │ 6. Return Updated Status        │                                 │
     │ <────────────────────────────   │                                 │
     │                                 │                                 │
```

#### Required API Endpoints

##### Send Notification (Game Server → OGS Notification API)

```http
POST /api/v1/notifications/send HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer GAME_API_KEY

{
  "recipient": {
    "gameUserId": "user-123"
  },
  "notification": {
    "type": "game_invitation",
    "title": "New Invitation",
    "body": "PlayerOne invited you to join Trivia Night!",
    "data": {
      "gameId": "trivia-456",
      "inviterId": "user-789",
      "inviterName": "PlayerOne",
      "gameName": "Trivia Night",
      "expiresAt": "2024-06-30T20:00:00Z"
    },
    "deepLink": "opengame://trivia-jam/join/trivia-456"
  }
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "notification-123",
  "status": "delivered",
  "deliveredAt": "2024-06-29T15:35:00Z"
}
```

##### Get Notification Status (Game Server → OGS Notification API)

```http
GET /api/v1/notifications/status/notification-123 HTTP/1.1
Host: api.opengame.org
Authorization: Bearer GAME_API_KEY
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "notification-123",
  "status": "read",
  "deliveredAt": "2024-06-29T15:35:00Z",
  "readAt": "2024-06-29T15:36:00Z"
}
```

##### Register Device (Game Client → OGS Notification API)

```http
POST /api/v1/notifications/register HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer SESSION_TOKEN

{
  "deviceToken": "device-token-from-fcm-or-apns",
  "platform": "ios", // or "android", "web"
  "topics": ["game_invitation", "turn_notification", "event_reminder"]
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "success": true,
  "deviceId": "device-123"
}
```

### TV Casting Protocol

TV casting allows games to be displayed on larger screens while using mobile devices as controllers. Importantly, **Chromecast support is completely independent** and does NOT require Auth Kit or Notification Kit implementation.

#### Protocol Overview

1. Game server creates a cast session with the OGS platform
2. OGS platform initializes the receiver application on the Chromecast device
3. Game client switches to controller mode
4. Game client sends input events to the OGS platform
5. OGS platform forwards input events to the receiver application
6. When finished, game client ends the cast session

#### HTTP Sequence for Casting

```
┌──────────┐                      ┌──────────┐                      ┌──────────┐
│   Game   │                      │    OGS   │                      │Chromecast│
│  Client  │                      │ Platform │                      │  Device  │
└────┬─────┘                      └────┬─────┘                      └────┬─────┘
     │                                 │                                 │
     │ 1. Create Cast Session          │                                 │
     │ ────────────────────────────>   │                                 │
     │                                 │                                 │
     │ 2. Return Session Info          │                                 │
     │ <────────────────────────────   │                                 │
     │                                 │                                 │
     │                                 │ 3. Initialize Receiver App      │
     │                                 │ ────────────────────────────>  │
     │                                 │                                 │
     │ 4. Switch to Controller Mode   │                                 │
     │ <───────────────────────────   │                                 │
     │                                 │                                 │
     │ 5. Send Input Events            │                                 │
     │ ────────────────────────────>   │                                 │
     │                                 │                                 │
     │                                 │ 6. Forward Input Events         │
     │                                 │ ────────────────────────────>  │
     │                                 │                                 │
     │ 7. End Cast Session             │                                 │
     │ ────────────────────────────>   │                                 │
     │                                 │                                 │
     │                                 │ 8. Terminate Receiver App       │
     │                                 │ ────────────────────────────>  │
     │                                 │                                 │
```

#### Required API Endpoints

##### Create Cast Session (Game Server → OGS Cast API)

```http
POST /api/v1/cast/session HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer GAME_API_KEY

{
  "gameId": "your-game-id",
  "gameUrl": "https://yourgame.com/play?mode=cast",
  "sessionData": {
    "gameState": "initial",
    "players": ["player1", "player2"]
  }
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "sessionId": "cast-session-123",
  "receiverUrl": "https://receiver.opengame.org/cast?session=cast-session-123",
  "status": "created"
}
```

##### Send Input to Cast Session (Game Client → OGS Cast API)

```http
POST /api/v1/cast/input HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer SESSION_TOKEN

{
  "sessionId": "cast-session-123",
  "inputType": "action",
  "inputData": {
    "action": "jump",
    "parameters": {
      "height": 2,
      "direction": "forward"
    }
  }
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "delivered",
  "timestamp": "2024-06-29T15:40:00Z"
}
```

##### Get Cast Session Status (Game Client → OGS Cast API)

```http
GET /api/v1/cast/session/cast-session-123 HTTP/1.1
Host: api.opengame.org
Authorization: Bearer SESSION_TOKEN
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "sessionId": "cast-session-123",
  "status": "active",
  "createdAt": "2024-06-29T15:30:00Z",
  "lastActivityAt": "2024-06-29T15:35:00Z"
}
```

##### End Cast Session (Game Client → OGS Cast API)

```http
DELETE /api/v1/cast/session/cast-session-123 HTTP/1.1
Host: api.opengame.org
Authorization: Bearer SESSION_TOKEN
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "terminated",
  "sessionId": "cast-session-123"
}
```

## Certification Process

To certify your game as OGS-compatible:

### 1. Domain Verification

Create a `.well-known/opengame-association.json` file at your domain root:

```json
{
  "appId": "your-game-id",
  "name": "Your Game Name",
  "version": "1.0.0",
  "contact": "developer@yourgame.com",
  "features": ["authentication", "notifications", "chromecast"],
  "apiVersion": "v1",
  "verification": "VERIFICATION_TOKEN"
}
```

The `VERIFICATION_TOKEN` will be provided during certification. Include only the features you've implemented in the features array. The `apiVersion` field should match the version of the OGS API you're implementing (currently "v1").

### 2. Trigger Verification

After creating the `.well-known` file, trigger the verification process by making a request to:

```
https://opengame.org/api/v1/verify?host=your-game-domain.com
```

Note: This endpoint is rate-limited to prevent abuse. Please avoid making multiple requests in quick succession.

### 3. API Key Generation

After successful domain verification, our automated system will send your API key to the contact email provided in the `.well-known` file:

* Keys are specific to each game and environment
* Store securely, never expose in client-side code
* Use the API key for server-to-server communication with the OGS platform

## SDKs for Implementation

While this specification defines the protocols at the HTTP level, we provide SDKs that implement these protocols to make integration easier:

- **[auth-kit](https://github.com/open-game-collective/auth-kit)**: Implementation of the Account Linking Protocol
- **[notification-kit](https://github.com/open-game-collective/notification-kit)**: Implementation of the Push Notification Protocol
- **[cast-kit](https://github.com/open-game-collective/cast-kit)**: Implementation of the TV Casting Protocol

## Future Extensions

The OGS specification is designed to be extensible. Future versions may include:

- **Wallet Integration**: Connecting with web3 wallets
- **In-App Purchases**: Cross-platform payment processing
- **Multiplayer Services**: Matchmaking and real-time communication
- **Achievements & Leaderboards**: Cross-game achievement tracking

## Version History

- **v1.0.0 (March 2025)** - Initial OGS specification release with authentication, push notifications, and Chromecast support

## Contact

- Website: [https://opengame.org](https://opengame.org)
- Email: [hello@opengame.org](mailto:hello@opengame.org)

## License

This specification is licensed under the MIT License. 
