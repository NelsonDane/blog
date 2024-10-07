---
date: '2024-09-22T00:03:31-04:00'
draft: false
title: 'Enabling Automatic Updates on NixOS Doesn''t Have to Be Hard'
description: 'Setting up unattended upgrades on NixOS using GitHub Actions and Systemd.'
series: ['Clarkson ECE Cluster']
series_order: 2
tags: ['nixos', 'github-actions', 'systemd', 'github', 'automation', 'cluster']
---

{{< lead >}}
*Real men test in production*

A response to "Why Is Enabling Automatic Updates In NixOS So Hard?" by [The Lion's Den](https://aires.fyi/blog/why-is-enabling-automatic-updates-in-nixos-hard/)
{{< /lead >}}

## Introduction
As I was putting together the [Clarkson ECE Cluster](https://github.com/NelsonDane/clarkson-nixos-cluster), I realized that I needed a way to keep the cluster nodes up-to-date without having to manually SSH into each one to pull new packages and configurations from the repository. After searching online, this was a common woe of many `NixOS` users, as enabling automatic updates was not as straightforward as it is on other systems. Partly, this is by design, as `NixOS` is a declarative system, and we don't want each node in the cluster to be updating independently at different times. This issue was put into words by [The Lion's Den](https://aires.fyi/blog/why-is-enabling-automatic-updates-in-nixos-hard/), who asked the question *"Why Is Enabling Automatic Updates In NixOS So Hard?"*

In this post, I'll show you how I set up automatic updates on the `Clarkson ECE Cluster` using `GitHub Actions` and `Systemd`, addressing some of the concerns raised by *The Lion's Den*.

## The Main Concerns
The main concerns raised by *The Lion's Den* were:
- 1. `Git` and `flake.lock` Updates

Using the "official" automatic updates provided by `NixOS` overwrites the `flake.lock` file, which is a problem if you're using `Git` to manage your `NixOS` configuration. If you have multiple systems/nodes, they will all have different `flake.lock` files, which is not ideal.
- 2. Multiple Systems

When using a centralized `NixOS` configuration repository, using `system.autoUpgrade` will cause all systems to update independently of each other, and more notably, independently of the committed `flake.lock` file. This can cause issues if you have a multi-node cluster setup. Which node is the source of truth?
- 3. `systems.autoUpgrade` is incomplete

The built-in `system.autoUpgrade` does update the `flake.lock` file, but it doesn't push those changes to the repository. This means that each node will have a different `flake.lock` file and the remote repository will remain outdated.

{{< lead >}}
*Notably, our furry friend did not have a clear solution*
{{< /lead >}}

All of these issues can be boiled down to the fact that there is no "source of truth" for the `flake.lock` file since each node would be updating independently. However, what if the source of truth wasn't the `LOCAL` node's `flake.lock` file, but the `REMOTE` repository's `flake.lock` file? This way, each node would be updating from the same `flake.lock` file, and the repository would always be up-to-date.

## My Solution
The solution that *The Lion's Den* came up with was to use a script that would pull the latest changes from the remote repository, update the `flake.lock` file, and then commit and push the changes back to the repository. However, this solution doesn't work for a multi-node cluster, as each node would be trying to update the repository at the same time, potentially causing conflicts. I'd also have to add a `GitHub` token to the repository, which I'd rather not do if I can avoid it.

To improve upon this solution, I decided to use `GitHub Actions` to update the `flake.lock` file, then push the changes back to the repository. This way, the changes aren't coming from `ANY` node in the cluster, but from a centralized "source of truth". This way, I can avoid the conflicts that would arise from multiple nodes trying to update the repository at the same time, since all the nodes will do is pull from the remote repository, never push.

To illustrate where my solution differs, I've provided the following flowcharts:

1. Official NixOS Solution (`system.autoUpgrade`):

`flake.lock` is updated on the local node, but never pushed to the repository.

{{< mermaid >}}
graph TD
    A[GitHub Repository]
    subgraph B[Local Node]
        C[Update flake.lock]
    end
    C --> |Apply Changes| C
    C --x |Changes Are Not Pushed| A
{{< /mermaid >}}

2. Solution by *The Lion's Den*:

`flake.lock` is updated on the local node, then committed and pushed to the repository.

{{< mermaid >}}
graph TD
    A[GitHub Repository]
    subgraph B[Local Node]
        C[Update flake.lock]
    end
    C --> |Commit and Push| A
    A --> |Pull Changes| B
{{< /mermaid >}}

3. My Solution:

`flake.lock` is updated on the remote repository using `GitHub Actions`, then pulled and applied by the local node(s).

{{< mermaid >}}
graph TD
    A[GitHub Repository]
    subgraph B[GitHub Actions]
        C[Update flake.lock]
    end
    subgraph D[Local Node]
        E[Apply Changes]
    end
    C --> |Commit and Push| A
    A --> |Checkout| B
    A --> |Pull Changes| E

{{< /mermaid >}}

Using my solution, the remote repository is treated as a single "source of truth," and the local nodes will always be updating from the same `flake.lock` file. This way, I can avoid the conflicts that would arise from multiple nodes trying to update the repository at the same time.

## Implementing My Solution
To implement my solution, I created a `GitHub Action` that would update the `flake.lock` file, commit the changes, and push them back to the repository, as shown below:

```yaml
# .github/workflows/update-nixos.yml
name: Update NixOS flake.lock

on:
  schedule:
    - cron: "0 3 * * *"
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Install Nix
        uses: cachix/install-nix-action@v27
      - name: Update Nix
        run: nix flake update
      - name: Commit changes
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: Bump flake.lock
          branch: main
          commit_options: '--no-verify --signoff'
          file_pattern: flake.lock
          commit_user_name: Nix Flake Bot
          commit_author: Nix Flake Bot <actions@github.com>
          skip_dirty_check: false
```

Let's break this down:

- `on`: This specifies that the workflow should run every day at 3 AM, or when triggered manually.
- `permissions`: This gives the workflow permission to write to the repository.
- `run: nix flake update`: This updates the `flake.lock` file.
- `uses: stefanzweifel/git-auto-commit-action`: This commits the changes to the repository with the following options:
    - `file_pattern: flake.lock`: This specifies that only the `flake.lock` file should be committed.
    - `skip_dirty_check: false`: This specifies that the changes should only be committed if the file actually changed.

{{< lead >}}
*Now the Source of Truth is Always True*
{{< /lead >}}

With this workflow running everyday at 3am, the `flake.lock` file on the repository will always be up-to-date. Now, all that's left to do is have the local nodes pull the changes from the repository and apply them. This can be done with a simple [systemd Timer](https://nixos.wiki/wiki/Systemd/Timers) as shown below:

```nix
# configuration.nix
system.autoUpgrade.enable = false;
systemd.timers."update-nixos" = {
    wantedBy = [ "timers.target" ];
    partOf = [ "update-nixos.service" ];
    timerConfig = {
        OnCalendar = "*-*-* 03:30:00";
        Unit = "update-nixos.service";
    };
};

systemd.services."update-nixos" = {
    serviceConfig = {
        Type = "oneshot";
        User = "root";
    };
    path = with pkgs; [ nixos-rebuild ];
    script = ''
        nixos-rebuild switch --flake github:NelsonDane/clarkson-nixos-cluster#${meta.hostname}
    '';
};
```
(Don't worry about the `meta.hostname` part, that's just a variable I use to differentiate between the nodes in the cluster)

This systemd timer is basically a cron job (cron has been deprecated in NixOS, see [here](https://nixos.wiki/wiki/Cron)) that runs the `update-nixos.service` every day at 3:30 AM. The `update-nixos.service` is a simple service that runs `nixos-rebuild switch` against the remote repository. This way, the local nodes will always be updating from the same `flake.lock` file, resulting in a consistent cluster.

If it was successful, you should see the following output in the `journalctl` logs:

```bash
[cluster@cluster-node-2:~]$ sudo journalctl -fu update-nixos.service
Oct 05 03:30:01 cluster-node-2 systemd[1]: Starting update-nixos.service...
Oct 05 03:30:03 cluster-node-2 update-nixos-start[2517577]: building the system configuration...
Oct 05 03:30:09 cluster-node-2 systemd[1]: update-nixos.service: Deactivated successfully.
Oct 05 03:30:09 cluster-node-2 systemd[1]: Finished update-nixos.service.
Oct 05 03:30:09 cluster-node-2 systemd[1]: update-nixos.service: Consumed 1.053s CPU time, 22.4M memory peak, 0B memory swap peak, received 33.8K IP traffic, sent 5.6K IP traffic.
```

## Conclusion (And a Possible Downside)
We still need to talk about the initial hook at the beginning of this post. If you've made it this far, you'd notice that every node in the cluster now automatically applies updates without any human intervention. While this is by design, it can be risky if a package introduces a breaking change and suddenly every node in the cluster decides to apply the update at the same time. While `NixOS` makes it easy to roll back to a previous configuration, it's still something to keep in mind... but for me, the benefits outweigh the risks.
