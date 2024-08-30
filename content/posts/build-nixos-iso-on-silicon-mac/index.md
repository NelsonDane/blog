---
date: '2024-08-30T16:13:18-04:00'
draft: false
title: 'Building an x86 NixOS ISO on an Apple Silicon Mac'
description: 'How to build an x86 NixOS ISO on an Apple Silicon Mac.'
series: []
series_order: 0
tags: ['nixos', 'docker', 'apple-silicon', 'x86_64']
---

{{< lead >}}
ERROR: a `x86_64-linux` is required to build, but I am a `aarch64-darwin`

*Sigh...*
{{< /lead >}}

## Introduction
I've recently taken the dive into the [NixOS](https://nixos.org/) system, and for the most part, it's been great. It's a fully declarative and reproducible OS, and I highly recommend you check it out. However, as expected when learning new things, there's been some growing pains. And unfortunately, the first one came before I even had the it installed.

One of the cool features of NixOS is that you can build your own ISO using an ISO generator, with predefined configuration such as your SSH keys, or a few useful packages. However, when I tried to build the ISO on my Silicon Mac, I was greeted with the error message above. This post will show how I solved the issue and was able to build the ISO on my Apple Silicon based Mac.

## The Plan
While I thought that perhaps I could get NixOS to use the Apple Rosetta translation layer to build the ISO, I was unable to figure that out (Is it possible?). Instead, I opted to use a forced `x86_64` environment using Docker. Fortunately, NixOS has an [official Docker image](https://hub.docker.com/r/nixos/nix) that I could use to build the system.

Inside a folder aptly named `iso`, I created a `flake.nix` file to define my custom installation:

```nix
{
  description = "Minimal NixOS ISO";
  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-24.05";

  outputs = { self, nixpkgs }: {
    nixosConfigurations = {
      exampleIso = nixpkgs.lib.nixosSystem {
        system = "x86_64-linux";
        modules = [
          ({ pkgs, modulesPath, ... }: {
            imports = [ (modulesPath + "/installer/cd-dvd/installation-cd-minimal.nix") ];
            environment.systemPackages = with pkgs; [
              vim
              git
            ];
            systemd.services.openssh.enable = true;
            systemd.services.sshd.wantedBy = [ "multi-user.target" ];
            users.users.root.openssh.authorizedKeys.keys = [
              "ssh-ed25519 <MY_PUBLIC_SSH_KEY>"
            ];
          })
        ];
      };
    };
  };
}
```

Without going into too much detail, this flake defines a base NixOS install, along with enabling SSH from my public key, and installs the wonderful packages `vim` and `git`. I then ran the following command to build the ISO:
```bash
nix build .#nixosConfigurations.exampleIso.config.system.build.isoImage
```

On an x86_64 based system, this would build the ISO without issue and you'd be done! However, on my Apple Silicon Mac, I was greeted with the error message above. Part of the Apple tax I guess.

## Building the ISO in a Docker Container
Thankfully, thanks to Rosetta, Docker can run `x86_64` containers on Apple Silicon Macs. Still inside the folder containing my `flake.nix` above, I created an ephemeral Docker container to build the ISO using the following command:

```bash
docker run --platform=linux/amd64 --rm -it -v $PWD:/out -w=/out nixos/nix
```

This command creates a temporary Docker container (the `--rm` flag) using the `x86_64` architecture (the `--platform=linux/amd64` flag), and maps the current directory (`$PWD`) to `/out` in the container. Then it drops you into a command line inside the container (`-it`). Once inside, I found that some experimental Nix features were required to build the ISO:

```bash
export NIX_CONFIG=$'filter-syscalls = false\nexperimental-features = nix-command flakes'
```

While I could create a Dockerfile and host my own Docker image with these features enabled, I don't think that I'm going to be building enough ISOs to warrant that. So I'll just keep this command handy for when I need to build another.

Now that I'm set to build the ISO, I ran the following command from before (*This will take a while*):

```bash
nix build .#nixosConfigurations.exampleIso.config.system.build.isoImage
```

Once it finishes building, you'll find the fresh ISO in the `result` folder. To copy it to your host machine (the `/out` bindmount), you can use the following command:

```bash
mv result/iso/nixos-*.iso /out
```

## Conclusion
And that's it! You can now use your new custom ISO to install NixOS on any number of nodes and automate the installation using SSH. While I know this won't be the last issue I face with NixOS, at least they'll be related to actually using the system... right?

## Credits
- [ferdy.me](https://ferdy.me/building-custom-nixos-iso-via-docker/) for the ephemeral Docker command.
- [maulana.id](https://maulana.id/soft-dev/2023--08--22--00--running-nix-inside-docker-container-inside-apple-silicon/#use-nixpkgsnix-of-x86_64-arch-to-build-aarch64-nixpkgsnix) for the experimental Nix features.
