# FDO 2.0 Support

This client supports both FDO 1.1 and FDO 2.0 protocols.

## Protocol Version Selection

Use the `--fdo-version` flag to select the protocol version:

```bash
# FDO 1.1 (default - for backward compatibility)
go-fdo-client onboard --blob cred.bin --key ec256 --kex ECDH256

# FDO 2.0 (explicit)
go-fdo-client onboard --fdo-version 200 --blob cred.bin --key ec256 --kex ECDH256
```

**Default is FDO 1.1** (`101`) for backward compatibility with existing servers.

## FDO 2.0 Features

### Message Types
FDO 2.0 uses different message type ranges:
- **TO0**: Message types 20-23 (instead of 30-33 in v1.1)
- **TO1**: Message types 30-33 (instead of 40-43 in v1.1)
- **TO2**: Message types 80-91 (instead of 60-71 in v1.1)

### HTTP Routing
- FDO 1.1: `/fdo/101/msg/{msgType}`
- FDO 2.0: `/fdo/200/msg/{msgType}`

The client automatically uses the correct path based on `--fdo-version`.

### Capability Flags
FDO 2.0 introduces capability flags exchanged during TO1:
- `Capb0SupFDO10`: Supports FDO 1.0
- `Capb0SupFDO11`: Supports FDO 1.1
- `Capb0SupFDO20`: Supports FDO 2.0

The client advertises FDO 2.0 support when using `--fdo-version 200`.

### Hash Binding Chain
FDO 2.0 adds anti-replay protection via hash binding:
- `HashPrev`: Hash of previous TO2 message (n-1)
- `HashPrev2`: Hash of message n-2

This prevents message replay attacks during ownership transfer.

### Additional Authenticated Data (AAD)
FDO 2.0 uses AAD for COSE signatures to prevent signature substitution attacks.

## Cross-Version Compatibility

The client can onboard with servers supporting either protocol version:
- FDO 2.0 client → FDO 2.0-capable server ✓
- FDO 1.1 client → FDO 2.0-capable server ✓

Version negotiation happens automatically during TO1 via capability flags.

## Examples

### Basic FDO 1.1 Onboarding (Default)

```bash
# Default FDO 1.1
go-fdo-client onboard \
  --blob /etc/fdo/device_credential.bin \
  --key ec256 \
  --kex ECDH256 \
  --cipher A128GCM
```

### FDO 2.0 Onboarding

```bash
# Explicit FDO 2.0
go-fdo-client onboard \
  --fdo-version 200 \
  --blob /etc/fdo/device_credential.bin \
  --key ec256 \
  --kex ECDH256
```

### Configuration File

```yaml
onboard:
  fdo-version: "200"  # or "101" for FDO 1.1
  key: ec256
  kex: ECDH256
  cipher: A128GCM
  blob: /etc/fdo/device_credential.bin
```

```bash
go-fdo-client onboard --config config.yaml
```

## Debugging

Enable debug logging to see protocol details:

```bash
go-fdo-client onboard --fdo-version 200 --debug ...
```

Look for:
- `Using FDO 2.0 protocol (message types 80-91)` - Confirms FDO 2.0 is used
- Message type numbers in logs (80-91 for TO2 in FDO 2.0)
- Capability flags in TO1 messages

## Reference

- [FDO 2.0 Specification](https://fidoalliance.org/specs/FDO/FIDO-Device-Onboard-WD-v2.0-20250617/)
- [FDO 1.1 Specification](https://fidoalliance.org/specs/FDO/FIDO-Device-Onboard-PS-v1.1-20220419/)
- [go-fdo library](https://github.com/fido-device-onboard/go-fdo)
