# pocket-id-declarative

Declarative OIDC client configuration for [Pocket-ID](https://pocket-id.org).

A NixOS module that syncs OIDC application declarations into Pocket-ID via its REST API idempotently. Services that want OIDC authentication declare their client requirements in Nix, and this module creates or updates them on every deploy.

## Why

Pocket-ID's Web UI is fine for one-off setup, but when you have many services (Sonarr, Radarr, Grafana, Homarr, etc.), each needing an OIDC client, it's tedious and error-prone. This module makes the OIDC client registry part of your NixOS config — declarative, version-controlled, reproducible.

## Usage

### 1. Add the flake input

```nix
# flake.nix
inputs = {
  pocket-id-declarative.url = "github:ogglord/pocket-id-declarative";
};
```

### 2. Import the module

```nix
# configuration.nix
imports = [
  inputs.pocket-id-declarative.nixosModules.default
];
```

### 3. Declare your OIDC clients

Each service declares its own client:

```nix
services.pocket-id-declarative.clients.sonarr = {
  name = "Sonarr";
  redirectUris = [ "https://sonarr.cignl.cc/oauth/callback" ];
  # Optional:
  # isPublic = false;
  # pkceEnabled = true;
  # launchURL = "https://sonarr.cignl.cc";
};

services.pocket-id-declarative.clients.radarr = {
  name = "Radarr";
  redirectUris = [ "https://radarr.cignl.cc/oauth/callback" ];
};
```

### 4. Configure the API connection

```nix
services.pocket-id-declarative = {
  enable = true;
  baseUrl = "http://127.0.0.1:1411";                # Pocket-ID internal URL
  staticApiKeyFile = "/run/secrets/pocket-id/STATIC_API_KEY";  # file containing the API key
};
```

The sync script runs on every `nh os switch` via `postStart` on the pocket-id service.
