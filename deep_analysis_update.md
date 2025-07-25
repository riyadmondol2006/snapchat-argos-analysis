# Deep Analysis Update - Additional Findings

## 1. New Cryptographic Discoveries

### Encryption Algorithms Found
- **AESGCC20** - AES-GCM with 20-byte tag (custom implementation)
- **CC20AESG** - ChaCha20-AES hybrid encryption
- **TLS_RSA_WITH_AES_256_GCM_SHA384** - TLS cipher suite
- **TLS_CHACHA20_POLY1305_SHA256** - Modern cipher suite
- **RSA-SHA1** - RSA signature with SHA1
- **AES-128-OFB, AES-192-ECB, AES-256-CFB1** - Various AES modes
- **id-aes192-GCM** - AES-192 in GCM mode
- **hmac sha-1 openssl** - HMAC-SHA1 implementation

### Key Management
```
pthread_key_create/delete - Thread-local key storage
getKeyForCurrentUserAsync - User-specific key retrieval
mDeviceEncryptionKey - Device-specific encryption key
recipient_key_version - Key versioning system
mWrappedKeys - Key wrapping/unwrapping mechanism
onSecurityKeyChanged - Key rotation callback
```

## 2. Protocol Buffer Definitions

### New Proto Discovery
- **getAttestationPayloadProto** function at context:
  - Located near "/token.bin" string
  - Associated with "token_refresh_scheduled"
  - Part of GrpcCpp implementation

### Device Identification Proto
```
com.snapchat.auth.proto.tivs.DeviceData.device_id
```

### Additional Protos Found
- `snapchat.messaging.GetAdConversationsRequest`
- `snapchat.messaging.GetAdConversationsResponse`
- `snapchat.messaging.EelReEncryptionDestinationByConversation`
- `snap.snap_maps_sdk.Value.KeyValuePair`

## 3. Token Storage and Caching

### Cache Implementation
```sql
SELECT length(data) FROM resources WHERE cache_key = ?
UPDATE resources SET accessed = ?1, expires = ?2, must_revalidate = ?3 WHERE cache_key = ?4
```

### Token File Storage
- **/token.bin** - Binary token storage file
- Cache key-based resource management
- Expiration and revalidation tracking

## 4. Advanced Security Features

### Key Update Protection
```
"Received a second key_update_not_yet_supported"
"key_update_not_yet_supported"
```
- Protection against key update attacks
- Version control for key updates

### Network Isolation
```
SplitCacheByNetworkIsolationKey
PartitionConnectionsByNetworkIsolationKey
```
- Network-level isolation for security
- Cache partitioning by network context

### Public Key Validation
```
ParsePublicKey
"Failed parsing extended key usage"
"1-RTT keys already available when 0-RTT is rejected"
"Rejecting public key chain for domain"
```

## 5. SSL/TLS Integration

### SSL Key Logger
```
../../net/ssl/ssl_key_logger_impl.cc
```
- Integration with Chrome's network stack
- SSL key logging for debugging (likely disabled in production)

### Certificate Operations
```
keyCertSign
"Failed parsing extended key usage"
```

## 6. Database Schema for Token Management

### Foreign Key Constraints
```sql
foreign key(local_message_content_id) references local_message_content(local_message_content_id)
primary key(phone_number, local_message_content_id)
```

### Required Values Storage
```sql
DELETE FROM required_values WHERE KEY = 'NumPrevUpgradeAttempts'
```

## 7. Obfuscation Patterns

### String Obfuscation
- Long concatenated preference keys (e.g., ACQUISITION_PLATFORM_USER_ID_STR_OF_INVITER...)
- Embedded HTML/JavaScript snippets for confusion
- Mixed case and underscore patterns

### Code Patterns
- Heavy use of SQL queries for indirection
- Multiple abstraction layers (Java → JNI → C++ → gRPC)
- Thread-local storage for sensitive data

## 8. Device Fingerprinting Methods

### Android-Specific
```
android.widget.Button
android/widget/OverScroller$SplineOverScroller
(Landroid/content/Context;)Landroid/media/AudioManager;
```

### Cronet Integration
```
../../components/cronet/android/cronet_bidirectional_stream_adapter.cc
```
- Custom Chromium network stack
- Bidirectional streaming support

## 9. Additional Token Components

### Attestation Timing
- `token_refresh_scheduled` - Scheduled token refresh
- `task_response_time` - Performance tracking
- `ttfb` (Time To First Byte) metrics

### Error Handling
```
"map::at: key not found"
"Server conversation version is missing"
```

## 10. Memory Addresses Update

### Key Function Locations
- `getAttestationPayloadProto` - Found in string table
- Associated with GrpcCpp proto buffer writer
- Near token.bin file reference

### Cryptographic Constants Location
- AES/RSA/HMAC implementations scattered throughout
- No obvious hardcoded keys found (likely generated/derived)
- Key material protected through wrapping

## Security Implications

1. **Multi-Layer Encryption**: Uses both symmetric (AES) and asymmetric (RSA) crypto
2. **Key Rotation**: Dynamic key updates with version tracking
3. **Hardware Integration**: Though not directly visible, architecture supports hardware attestation
4. **Network Isolation**: Advanced cache and connection partitioning
5. **Proto-Based Communication**: All token data structured as protocol buffers

## Next Steps for Even Deeper Analysis

1. **Dynamic Analysis**: Use Frida to hook:
   - `getAttestationPayloadProto`
   - `getKeyForCurrentUserAsync`
   - `onSecurityKeyChanged`

2. **Binary Patching**: Modify token.bin access for inspection

3. **Network Analysis**: Intercept gRPC calls to understand proto structures

4. **Memory Dump**: Extract runtime keys and tokens

The token generation system is extremely sophisticated with multiple security layers, making static analysis alone insufficient for complete understanding.