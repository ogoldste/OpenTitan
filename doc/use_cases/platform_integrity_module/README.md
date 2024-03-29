# Platform Integrity Module

An OpenTitan IC used as a Platform Integrity Module interposes between a
platform's boot flash and its main boot devices such as the Baseboard Management
Controller (BMC), the Platform Controller Hub (PCH) and the CPU.

<img src="use_cases_fig1.svg" alt="Fig1" style="width: 300px;"/>

Figure 1: Platform Integrity Module

The Platform Integrity Module use case implements the following security
properties:

*   Measure integrity of first boot firmware stages before bringing the boot
    devices out of reset accessing boot flash via SPI or similar interface.
*   Monitor resets and heartbeat of downstream boot devices. Monitoring tasks
    are handled by OpenTitan as Interrupt Service Routines (ISRs), and are not
    expected to operate under real time constraints.
*   Enforce runtime boot device access policies, and manage A/B firmware updates
    for software stored in boot flash. The OpenTitan to boot device interface is
    implemented on SPI or a similar interface.
*   Provides root key store and attestation flows as part of the platform
    integrity secure boot implementation.

### Minimum Crypto Algorithm Requirements

The current target for all crypto is at least 128-bit security strength. This is
subject to change based on the implementation timeline of any given
instantiation of OpenTitan. It is expected that a future implementation may be
required to target a minimum of 192-bit or 256-bit security strength.

*   TRNG: NIST SP 800-90B compliant entropy source.
*   DRBG: NIST SP 800-90A compliant DRBG.
*   Hash Algorithms:
    *   SHA256: An approved hash algorithm with approximately the same security
        strength as its strongest asymmetric algorithm.
*   Asymmetric Key Algorithms:
    *   RSA-3072: Secure boot signature verification.
    *   ECDSA P-256: Signature and verification for identity and attestation
        keys.
*   Symmetric Key Algorithms:
    *   HMAC-SHA256: NIST FIPS 180-4 compliant. Used in integrity measurements
        for storage and in transit data as well as secure boot.
    *   AES: AES-CTR NIST 800-38A. Used to wrap keys and encrypt data stored in
        internal flash.

### Provisioning Requirements

Provisioning an OpenTitan device is performed in two steps:

*   Device Personalization: The device is initialized with a unique
    cryptographic identity endorsed by a Transit PKI which is only used to
    support initial Ownership Transfer.
*   Ownership Transfer: Ownership is assigned to a user that has the ability to
    run software on the device. As Silicon Owner, the user can generate a
    cryptographic identity strongly associated to the hardware and the software
    version running on the device.

OpenTitan used as a Platform Integrity Module has the following provisioning
requirements:

*   Unique Global Identifier: Non-Cryptographic big integer value (up to 256b)
    used to facilitate tracking of the devices throughout their life cycle. The
    identifier is stored in One Time Programmable (OTP) storage during
    manufacturing.
*   Hardware Cryptographic Identity: Symmetric and asymmetric keys associated
    with the hardware, used to attest the authenticity of the chip and also as a
    component of the Owner Cryptographic Identity. These keys are generated
    inside the device by the secure manufacturing process.
*   Hardware Transport Certificate: Used to endorse the asymmetric hardware
    identity with a transit PKI trusted by the Silicon Owner at Ownership
    Transfer time.
*   Factory Firmware: Baseline image with support for firmware update and
    Ownership Transfer. Firmware update may be actuated by writing an OpenTitan
    update payload to boot flash. Upon reset, OpenTitan scans the boot flash
    device for valid updates. The factory image may not be owned by the Silicon
    Owner and its main purpose is to assist Ownership Transfer.
*   Owner Cryptographic Identity: The Silicon Owner is required to generate an
    identity as part of the Ownership transfer flow. Owner identities are bound
    to the Hardware Identity and the software version running on the device.
    Owner identities are used in Silicon Ownership attestation flows and as a
    root component of Application keys.
*   Application Keys: Keys bound to the owner identity and the application
    version running on the device. Application keys are provisioned in most
    cases when the application runs for the first time. The purpose of each key
    is configured at the application layer and enforced by the kernel.

### Performance Requirements

Performance requirements are derived from integration-specific requirements. The
following performance requirements are presented for illustration purposes:

*   Boot Time: Maximum time allowed to get to device kernel serving state from
    cold reset.
*   Resume Time: Maximum time allowed to get to device kernel serving state from
    sleep.
*   External Flash Verification Time: Maximum time allowed for verification of
    external flash as part of platform boot verified boot implementation.
    Defined in milliseconds for a given flash partition size.

### Packaging Constraints

*   Non-HDI packaging is required.
*   (Proposed) Device packaging QR code with device ID linkable to manufacturing
    data.

### Additional Requirements

#### Memory Requirements

*   At least 512KB of flash storage with 2 partitions, 4KB page size, 100K
    endurance cycles. 1MB flash would be ideal to allow for future code size
    growth.
*   At least 16KB of isolated flash storage for manufacturing and device life
    cycle operations.
*   At least 8KB of OTP for manufacturing and device life cycle operations.
*   At least 64KB of SRAM. 128KB would be ideal for future growth.

#### External Peripherals

The following list of peripheral requirements is speculative at the moment and
subject to change based on platform integration requirements:

*   SPI Host/Device:
    *   Dual support. Quad support needs to be evaluated.
    *   Required features for EEPROM mode:
        -   Passthrough boot flash interface with support for EEPROM command
            handling/filtering.
        -   Access to on-die ram and flash memory regions.
        -   Mailbox interface with support for custom opcode commands.
*   UART: Debug console interface. May be disabled by production firmware.
*   GPIO: Reset control and monitoring. Status signals.
*   I2C interface compatible with SMBus interfaces.
