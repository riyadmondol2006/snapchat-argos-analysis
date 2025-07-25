# Detailed Native Code Analysis - Argos Token System

## Core Components Discovered

### 1. Security Service Architecture

The token generation uses a gRPC service called `ArgosService`:
- **Service Path**: `/snap.security.ArgosService/GetTokens`
- **Namespace**: `snap.security`

### 2. Protocol Buffer Messages

```
snap.security.GetTokensRequest
snap.security.GetTokensResponse
snap.security.TokenAndPolicy
snap.security.TokenRecord
```

### 3. Token Headers Found
- `x-snapchat-att-token` - Main attestation token
- `x-snapchat-att-sign` - Signature header (new finding!)
- `x-snapchat-att` - Partial header found

### 4. Implementation Classes

#### Core Manager Classes
- `TokenManagerImpl` - Main token management implementation
- `ArgosServiceClientImpl` - Client for communicating with Argos service
- `ArgosClientImpl` - Main Argos client implementation

#### Callback and Wrapper Classes
- `ArgosServiceCallbackWrapper`
- `FetchTokenServiceCallback`
- `AttestationHeadersCallback`
- `SynchronousWrapperCallback`

#### Platform Integration
- `PlatformClientAttestation` - Platform-specific attestation interface
- `FetchTokenCallback` - Token fetching callback

### 5. Token Generation Flow

1. **Request Creation**:
   - `GetTokensRequest` is created with request parameters
   - Includes URL, method, and attestation data

2. **Service Call**:
   - Call to `/snap.security.ArgosService/GetTokens`
   - Uses gRPC for communication

3. **Response Processing**:
   - `GetTokensResponse` contains `TokenAndPolicy` objects
   - Each `TokenRecord` includes token and metadata

4. **Token Caching**:
   - `ArgosTokenManagerQueue` manages token queue
   - Tokens are cached with TTL expiry

### 6. Key Functions

Native JNI functions:
```
Java_com_snapchat_client_client_1attestation_ArgosClient_createInstance
Java_com_snapchat_client_client_1attestation_ArgosClient_00024CppProxy_native_1getAttestationHeaders
Java_com_snapchat_client_client_1attestation_ArgosClient_00024CppProxy_native_1getArgosTokenAsync
```

Internal functions:
```
get_argos_token
get_attestation_payload_latency
get_cached_token
```

### 7. Token States and Events

Token states tracked:
- `token_available`
- `token_received`
- `hot_token` (frequently used tokens)

Refresh reasons:
- `preemptive_refresh`
- `refreshcancelled`

### 8. Performance Metrics

The system tracks various latencies:
- `mArgosLatency` - Overall Argos latency
- `mSignatureLatencyMs` - Time for signature generation
- `argoslatency` - Generic latency metric
- `authlatecy` - Authentication latency
- `get_attestation_payload_latency` - Payload generation time

### 9. Security Features

1. **Signature Generation**:
   - `x-snapchat-att-sign` header for request signing
   - Cryptographic operations in native code

2. **Token Policy**:
   - `TokenAndPolicy` structure suggests tokens have associated policies
   - Likely includes expiry, scope, and usage restrictions

3. **Platform Attestation**:
   - `PlatformClientAttestation` interface for platform-specific checks
   - Hardware-backed attestation on supported devices

### 10. Queue Management

- `ArgosTokenManagerQueue` - Manages token request queue
- Prevents duplicate requests for same endpoint
- Handles concurrent token requests

## Technical Implementation Details

### Token Request Structure
Based on the analysis, a token request likely includes:
- URL endpoint
- HTTP method
- Request body (optional)
- Device attestation data
- Timestamp
- Platform information

### Token Response Structure
The response contains:
- Token string
- Expiry time
- Usage policy
- Signature data

### Caching Strategy
- Tokens are cached by endpoint
- Cache TTL is configurable (`mCacheTtlExpiryInSeconds`)
- Hot tokens are kept for frequent requests

## Advanced Features

1. **Interceptors**:
   - `PRE_RECV_MESSAGE` hook for message interception
   - Allows modification of requests/responses

2. **Metadata Delegation**:
   - `ArgosServiceGrpcMetadataDelegateE` handles gRPC metadata
   - Custom headers and authentication

3. **Error Handling**:
   - Comprehensive error tracking
   - Fallback mechanisms for token failures

## Security Implications

The multi-layered approach includes:
1. Native code implementation (harder to reverse)
2. gRPC communication (encrypted)
3. Hardware attestation integration
4. Signature verification
5. Token policies and restrictions

This makes the system highly resistant to:
- Token forgery
- Replay attacks
- Man-in-the-middle attacks
- Client tampering