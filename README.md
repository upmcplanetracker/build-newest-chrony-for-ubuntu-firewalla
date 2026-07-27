# Chrony .deb Builder

Automatically downloads the latest stable [Chrony](https://chrony-project.org/) NTP client/server release, compiles it from source, and packages it as `.deb` files for Ubuntu.

## What this does

- **Scans daily** for new Chrony releases at `https://chrony-project.org/releases/`
- **Skips pre-releases** — only builds stable versions
- **Builds in clean Docker containers** for both Ubuntu 22.04 and 26.04 (amd64)
- **Creates a GitHub Release** automatically with the `.deb` files attached
- **No manual tracking file** — compares against existing GitHub Releases to avoid duplicate builds

## Download the latest package

1. Go to the [**Releases**](https://github.com/YOUR_USERNAME/YOUR_REPO/releases) page of this repository.
2. Download the `.deb` for your Ubuntu version.

| File | Target |
|------|--------|
| `chrony-X.Y.Z-ubuntu22.04_amd64.deb` | Ubuntu 22.04 (Jammy) |
| `chrony-X.Y.Z-ubuntu26.04_amd64.deb` | Ubuntu 26.04 |

> **Firewalla users:** The Ubuntu 22.04 package works on both 22.04 and 26.04 due to glibc backward compatibility. You likely only need the 22.04 build.

## Installing on Firewalla / Ubuntu

```bash
# Download the .deb from the Releases page, then:
sudo dpkg -i chrony_4.8-ubuntu22.04_amd64.deb

# If you see dependency errors, fix them with:
sudo apt-get install -f

# Or install via apt (handles deps automatically):
sudo apt install ./chrony_4.8-ubuntu22.04_amd64.deb
```

## Manual trigger

You can force a build at any time:

1. Go to **Actions** → **Build Chrony .deb Packages**
2. Click **Run workflow**
3. (Optional) Enter a specific version like `4.8` to force-build that version
4. Click **Run workflow**

## How it works

| Step | Details |
|------|---------|
| **Check** | Scrapes the upstream releases page for the latest stable version |
| **Compare** | Queries existing GitHub Releases to see if it's already been built |
| **Build** | Spins up `ubuntu:22.04` and `ubuntu:26.04` containers, installs build deps, compiles from the upstream tarball |
| **Package** | Uses `dpkg-deb` to create proper `.deb` packages with systemd service files |
| **Release** | Creates a GitHub Release (e.g., `chrony-4.8`) and attaches both `.deb` files |

## Build configuration

Chrony is compiled with:

```
--prefix=/usr
--with-user=_chrony
--enable-ntp-signd
--enable-scfilter
```

The resulting package includes:
- `chronyd` (daemon)
- `chronyc` (control utility)
- systemd service file (`chrony.service`)

## Troubleshooting

### Ubuntu 26.04 build fails
The `ubuntu:26.04` Docker image may not be available yet. If the 26.04 job fails:

1. Edit `.github/workflows/build-chrony.yml`
2. Find the `matrix` section
3. Change `ubuntu_version: "26.04"` to `ubuntu_version: "24.04"` (or `"devel"`)
4. Commit the change

### "Release already exists"
If you want to rebuild an existing version, use the **Run workflow** button and enter the version in the `force_version` field.

## License

The workflow and packaging scripts in this repository are provided as-is. Chrony itself is licensed under the GNU GPL v2.

---

*Built with GitHub Actions. No affiliation with the Chrony project.*
