# nyanspace-build

Public downloads hub for **nyanspace**. This repo holds no source code — it
only exists so finished, built installers can be published for anyone to
download without needing access to the private source repos.

## Downloads

Grab the latest installer/APK from the [Releases page](../../releases).

- **Desktop** (Windows `.exe`, Linux `.AppImage`/`.deb`/`.rpm`) — tagged
  `desktop-vX.Y.Z`. Source: `nyanspace-pc` (private).
- **Mobile** (Android `.apk`) — tagged `mobile-vX.Y.Z`. Source: `nyanspace`
  (private, legacy — superseded by the desktop app).

Each release includes a SHA-256 checksum file; verify your download against
it before running the installer.

## Where's the source code?

The source repos (`nyanspace`, `nyanspace-pc`) are private. Their CI builds,
signs, and packages each release, then publishes the resulting artifacts
here automatically. This repo is a one-way, read-only mirror of build
output — it has no write-path workflows of its own.

## License

[MIT License](./LICENSE).
