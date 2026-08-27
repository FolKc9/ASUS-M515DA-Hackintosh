# SMBIOS / Privacy

The public repository intentionally does not include a populated `EFI/OC/config.plist` because OpenCore SMBIOS identifiers must be unique to each installation.

Before using this EFI, create your own `config.plist` from the repository configuration/reference and generate your own values under `PlatformInfo -> Generic` for:

- `SystemSerialNumber`
- `MLB`
- `SystemUUID`
- `ROM`

Do not copy these values from another Hackintosh and do not publish your generated identifiers in a public repository.

Recommended practice: keep your personal `config.plist` only on your local machine and publish a sanitized template with placeholders if you want to distribute the complete EFI.
