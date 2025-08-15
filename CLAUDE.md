# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Installation and Setup
```bash
# Install package in development mode
pip install -e .

# Install with test dependencies
pip install -e ".[test]"
```

### Testing
```bash
# Run tests with pytest
pytest

# Run a specific test file
pytest tests/test_filename.py

# Run tests with coverage
pytest --cov=wyze_sdk
```

### Linting
```bash
# Run flake8 linter (configured in .flake8)
flake8 wyze_sdk

# Flake8 configuration:
# - Max line length: 125 characters
# - Max complexity: 18
# - Ignores: E203, E266, E501, W503
```

### Building and Distribution
```bash
# Build distribution packages
python setup.py sdist bdist_wheel

# Upload to PyPI (requires credentials)
python -m twine upload dist/*
```

## High-Level Architecture

### Core Client Structure
The SDK follows a layered architecture with clear separation of concerns:

1. **Client Layer** (`wyze_sdk.api.client.Client`): Main entry point that provides access to device-specific clients. Handles authentication and token management.

2. **Device Clients** (`wyze_sdk.api.devices.*`): Specialized clients for each device type (bulbs, cameras, locks, plugs, scales, sensors, switches, thermostats, vacuums). Each inherits from `BaseClient` and implements device-specific operations.

3. **Service Layer** (`wyze_sdk.service.*`): Backend API communication services:
   - `ApiServiceClient`: Core API communication with api.wyzecam.com
   - `AuthServiceClient`: Authentication and token management
   - Platform-specific services (Venus, Sirius, Earth, Ford, Pluto) for different Wyze services
   - Request signing and verification handled by `wyze_sdk.signature`

4. **Models Layer** (`wyze_sdk.models.devices.*`): Device data models and parsers:
   - Base device classes with common functionality (AbstractNetworkedDevice, SwitchableMixin, etc.)
   - Device-specific models inheriting from base classes
   - `DeviceParser` for converting API responses to typed device objects

### Authentication Flow
- Supports multiple authentication methods: token-based (preferred), email/password, API keys, and TOTP 2FA
- Tokens should be obtained once via `Client().login()` and stored for reuse
- Automatic token refresh capability via refresh tokens

### Key Design Patterns
- **Mixin Classes**: Common device behaviors (SwitchableMixin, LockableMixin, etc.) are implemented as mixins for code reuse
- **Property-based API**: Device properties are exposed as Python properties with getters/setters
- **Unified Error Handling**: All API errors raised as `WyzeApiError` exceptions
- **Response Wrapper**: All API responses wrapped in `WyzeResponse` objects for consistent handling

### Device Model Support
The SDK supports various Wyze device models identified by model codes (e.g., WLPA19C for bulbs). Device models are mapped in `DeviceModels` class and used for API routing and feature detection.

### Important Dependencies
- `requests`: HTTP communication
- `blackboxprotobuf`: Protocol buffer handling for certain API responses
- `mintotp`: TOTP 2FA code generation
- `pycryptodomex`: Cryptographic operations for request signing