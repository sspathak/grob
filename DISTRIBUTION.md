# Distribution Guide

This document explains how to distribute `git-grob` via various package managers.

## Homebrew Distribution

### Prerequisites
- A GitHub account
- The repository pushed to GitHub at `github.com/sspathak/zsh-grob`

### Step 1: Create a GitHub Release

```bash
# Tag the current version
git tag v1.0.0
git push origin v1.0.0
```

GitHub will automatically create a release tarball at:
`https://github.com/sspathak/zsh-grob/archive/refs/tags/v1.0.0.tar.gz`

### Step 2: Calculate SHA256

```bash
curl -sL https://github.com/sspathak/zsh-grob/archive/refs/tags/v1.0.0.tar.gz | shasum -a 256
```

Copy the SHA256 hash output.

### Step 3: Create a Homebrew Tap

A "tap" is a GitHub repository containing Homebrew formulae.

```bash
# Create a new repository on GitHub named: homebrew-tap
# Clone it locally
git clone https://github.com/sspathak/homebrew-tap.git
cd homebrew-tap

# Copy the formula
cp /path/to/zsh-grob/git-grob.rb ./git-grob.rb

# Update git-grob.rb with the correct SHA256 from Step 2
# Edit line 5 to replace the empty sha256 with your actual hash

# Commit and push
git add git-grob.rb
git commit -m "Add git-grob formula"
git push origin main
```

### Step 4: Test Installation

```bash
# Users can now install with:
brew tap sspathak/tap
brew install git-grob

# Verify it works
git grob --version  # or just: git grob
```

### Updating the Formula

When releasing a new version:

1. Create a new git tag (e.g., `v1.0.1`)
2. Calculate the new SHA256
3. Update `git-grob.rb` in the tap repository with:
   - New version number in the `url` field
   - New SHA256 hash
4. Commit and push the changes

## Manual Installation (Alternative)

Users can install without Homebrew:

```bash
# Download directly from GitHub
curl -o /usr/local/bin/git-grob https://raw.githubusercontent.com/sspathak/zsh-grob/main/git-grob
chmod +x /usr/local/bin/git-grob

# Verify
git grob
```

## Testing Locally Before Publishing

### Test the standalone executable

```bash
cd /path/to/zsh-grob

# Make it executable if not already
chmod +x git-grob

# Test directly
./git-grob

# Test as git subcommand by adding to PATH
export PATH="$PWD:$PATH"
git grob
```

### Test Homebrew formula locally

```bash
# Install from local formula (no tap needed)
brew install --build-from-source ./git-grob.rb

# Test
git grob

# Uninstall
brew uninstall git-grob
```

## Other Package Managers

### AUR (Arch User Repository)

Create a PKGBUILD file:

```bash
# Maintainer: Your Name <your.email@example.com>
pkgname=git-grob
pkgver=1.0.0
pkgrel=1
pkgdesc="Interactive git rebase --onto helper"
arch=('any')
url="https://github.com/sspathak/zsh-grob"
license=('MIT')
depends=('zsh' 'fzf' 'git')
source=("$pkgname-$pkgver.tar.gz::$url/archive/refs/tags/v$pkgver.tar.gz")
sha256sums=('SKIP')

package() {
    cd "$srcdir/zsh-grob-$pkgver"
    install -Dm755 git-grob "$pkgdir/usr/bin/git-grob"
    install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
```

### APT/DEB Package

Create a `.deb` package using `fpm`:

```bash
fpm -s dir -t deb -n git-grob -v 1.0.0 \
    --depends fzf \
    --depends zsh \
    --description "Interactive git rebase --onto helper" \
    --url "https://github.com/sspathak/zsh-grob" \
    --license MIT \
    git-grob=/usr/bin/git-grob
```

### Nix

Add to nixpkgs or create a shell.nix:

```nix
{ pkgs ? import <nixpkgs> {} }:

pkgs.stdenv.mkDerivation {
  name = "git-grob";
  version = "1.0.0";
  
  src = pkgs.fetchFromGitHub {
    owner = "sspathak";
    repo = "zsh-grob";
    rev = "v1.0.0";
    sha256 = "...";
  };
  
  buildInputs = [ pkgs.zsh pkgs.fzf ];
  
  installPhase = ''
    mkdir -p $out/bin
    cp git-grob $out/bin/
    chmod +x $out/bin/git-grob
  '';
}
```

## Publishing Checklist

Before publishing a new version:

- [ ] Test `git-grob` executable works locally
- [ ] Test with various git flags (`-i`, `--autosquash`, etc.)
- [ ] Update version numbers in README.md
- [ ] Update CHANGELOG.md (if exists)
- [ ] Create git tag
- [ ] Generate SHA256 for release
- [ ] Update Homebrew formula
- [ ] Test Homebrew installation locally
- [ ] Push tap repository changes
- [ ] Verify users can install via `brew install`
