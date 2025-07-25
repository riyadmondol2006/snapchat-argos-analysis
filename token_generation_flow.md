# Complete Token Generation Flow - Argos System

## Overview
Based on deep analysis of the native library, here's the complete token generation flow:

## 1. System Architecture

```
┌─────────────────┐
│   Java Layer    │
│  (ArgosClient)  │
└────────┬────────┘
         │ JNI
┌────────▼────────┐
│  Native Layer   │
│ (libclient.so)  │
└────────┬────────┘
         │
┌────────▼────────┐
│   gRPC Layer    │
│(ArgosService)   │
└────────┬────────┘
         │
┌────────▼────────┐
│  Token Service  │
│   (Backend)     │
└─────────────────┘
```

## 2. Key Components

### Java Layer
- `ArgosClient` - Main interface
- `ArgosClient$CppProxy` - JNI proxy to native code
- `AttestationHeadersCallback` - Callback for async operations

### Native Layer
- `ArgosClientImpl` - Core implementation
- `TokenManagerImpl` - Token lifecycle management
- `ArgosServiceClientImpl` - gRPC client
- `PlatformClientAttestation` - Platform-specific attestation

### Security Components
- Cryptographic signing (SHA, signature verification)
- Device attestation
- Request validation

## 3. Token Generation Process

### Step 1: Client Initialization
```java
ArgosClient.createInstance(
    PlatformClientAttestation,  // Device attestation
    Configuration,               // Client config
    AuthContextDelegate,         // Auth context
    ArgosPlatformBlizzardLogger, // Logging
    DispatchQueue               // Threading
)
```

### Step 2: Token Request
When making an API request:

1. **Java calls native method**:
   ```java
   getAttestationHeaders(url, method, hasBody, body, argosMode)
   ```

2. **Native layer processes request**:
   - Checks token cache (`ArgosTokenManagerQueue`)
   - If cached and valid, returns immediately
   - If not, proceeds to generate new token

### Step 3: Token Generation

1. **Create GetTokensRequest**:
   ```
   snap.security.GetTokensRequest {
       url: string
       method: string
       body_hash: string (if body exists)
       timestamp: int64
       device_info: DeviceInfo
       argos_mode: ArgosMode
   }
   ```

2. **Platform Attestation**:
   - `PlatformClientAttestation` performs device checks
   - Hardware attestation (SafetyNet/DeviceCheck)
   - App integrity verification
   - Returns attestation payload

3. **Request Signing**:
   - Generate request signature
   - Uses cryptographic operations (found signatures for SHA, RSA)
   - Creates `x-snapchat-att-sign` header

4. **gRPC Call**:
   ```
   /snap.security.ArgosService/GetTokens
   ```
   - Sends GetTokensRequest
   - Backend validates attestation
   - Returns GetTokensResponse

### Step 4: Token Response Processing

1. **GetTokensResponse Structure**:
   ```
   snap.security.GetTokensResponse {
       tokens: [TokenAndPolicy]
       cache_ttl: int32
       refresh_strategy: RefreshStrategy
   }
   ```

2. **TokenAndPolicy**:
   ```
   snap.security.TokenAndPolicy {
       token_record: TokenRecord
       policy: TokenPolicy
   }
   ```

3. **TokenRecord**:
   ```
   snap.security.TokenRecord {
       token: string         // The actual att-token
       expiry: int64
       token_type: ArgosType
   }
   ```

### Step 5: Token Caching

1. **Cache Management**:
   - Tokens cached by endpoint key
   - TTL from `cache_ttl` in response
   - Hot tokens kept for frequent requests
   - `ArgosTokenManagerQueue` manages queue

2. **Refresh Strategies**:
   - `PREWARMING` - Background refresh
   - `PREEMPTIVEREFRESH` - Refresh before expiry
   - `BLOCKINGREFRESH` - Immediate refresh required

### Step 6: Header Generation

Final headers returned:
```
{
    "x-snapchat-att-token": <token>,
    "x-snapchat-att-sign": <signature>,
    "x-request-consistent-tracking-id": <tracking_id>
}
```

## 4. Security Features

### Cryptographic Operations
- Request signing with SHA algorithms
- Signature verification
- Token encryption (references to encrypt/decrypt found)

### Platform Security
- Hardware attestation integration
- App signature verification
- Certificate pinning
- Anti-tampering checks

### Token Policies
- Usage restrictions
- Expiry management
- Scope limitations
- Rate limiting

## 5. Performance Optimizations

### Metrics Tracked
- `mArgosLatency` - Total latency
- `mSignatureLatencyMs` - Signing time
- `get_attestation_payload_latency` - Attestation time
- `argoslatency` - Generic metric

### Caching Strategy
- Multi-level cache (memory + persistent)
- Preemptive refresh for hot tokens
- Queue management to prevent duplicates

## 6. Error Handling

### Failure Scenarios
- Attestation failure
- Network errors
- Invalid signatures
- Expired tokens

### Recovery Mechanisms
- Retry with exponential backoff
- Fallback to cached tokens
- Grace period for expired tokens

## 7. Token Lifecycle

1. **Generation**: On-demand when needed
2. **Caching**: Stored with TTL
3. **Usage**: Added to request headers
4. **Refresh**: Based on strategy
5. **Expiry**: Automatic cleanup

## Implementation Notes

The actual token generation involves:
1. Complex device fingerprinting
2. Hardware-backed cryptographic operations
3. Multiple layers of validation
4. Server-side verification

This makes the system highly secure and resistant to:
- Token forgery
- Replay attacks
- Client tampering
- Man-in-the-middle attacks