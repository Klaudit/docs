---
layout: docs
category: running_coreos
sub_category: cloud_provider
supported: true
weight: 6
title: NIFTY Cloud
---

# Running CoreOS on NIFTY Cloud

Before proceeding, you will need to [install NIFTY Cloud CLI][cli-documentation].

[cli-documentation]: https://translate.google.com/translate?hl=en&sl=ja&tl=en&u=http%3A%2F%2Fcloud.nifty.com%2Fapi%2Fcli%2F

## Cloud-Config

CoreOS allows you to configure machine parameters, launch systemd units on startup and more via cloud-config. Jump over to the [docs to learn about the supported features]({{site.url}}/docs/cluster-management/setup/cloudinit-cloud-config). Cloud-config is intended to bring up a cluster of machines into a minimal useful state and ideally shouldn't be used to configure anything that isn't standard across many hosts. On NIFTY Cloud, the cloud-config can be modified while the instance is running and will be processed next time the machine boots.

You can provide cloud-config to CoreOS via [NIFTY Cloud CLI][cli-documentation].

The most common cloud-config for NIFTY Cloud looks like:

```yaml
#cloud-config

coreos:
  etcd:
    # generate a new token for each unique cluster from https://discovery.etcd.io/new
    discovery: https://discovery.etcd.io/<token>
    # multi-region and multi-cloud deployments need to use $public_ipv4
    addr: $private_ipv4:4001
    peer-addr: $private_ipv4:7001
  units:
    - name: etcd.service
      command: start
    - name: fleet.service
      command: start
```

The `$private_ipv4` and `$public_ipv4` substitution variables are fully supported in cloud-config on NIFTY Cloud.

## Choosing a Channel

CoreOS is designed to be [updated automatically]({{site.url}}/using-coreos/updates) with different schedules per channel. You can [disable this feature]({{site.url}}/docs/cluster-management/debugging/prevent-reboot-after-update), although we don't recommend it. Read the [release notes]({{site.url}}/releases) for specific features and bug fixes.

<div id="niftycloud-images">
  <ul class="nav nav-tabs">
    <li class="active"><a href="#stable" data-toggle="tab">Stable Channel</a></li>
    <li><a href="#beta" data-toggle="tab">Beta Channel</a></li>
    <li><a href="#alpha" data-toggle="tab">Alpha Channel</a></li>
  </ul>
  <div class="tab-content coreos-docs-image-table">
    <div class="tab-pane" id="alpha">
      <p>The alpha channel closely tracks master and is released to frequently. The newest versions of <a href="{{site.url}}/using-coreos/docker">docker</a>, <a href="{{site.url}}/using-coreos/etcd">etcd</a> and <a href="{{site.url}}/using-coreos/clustering">fleet</a> will be available for testing. Current version is CoreOS {{site.alpha-channel}}.</p>
      <p>Launch via NIFTY Cloud CLI by specifying <code>$ZONE</code>, <code>$TYPE</code>, <code>$FW_ID</code> and <code>$SSH_KEY_ID</code>:</p>
      <pre>nifty-run-instances $(nifty-describe-images --delimiter ',' --image-name "CoreOS Alpha {{site.alpha-channel}}" | awk -F',' '{print $2}') --key $SSH_KEY_ID --availability-zone $ZONE --instance-type $TYPE -g $FW_ID -f cloud-config.yml</pre>
    </div>
    <div class="tab-pane" id="beta">
      <p>The beta channel consists of promoted alpha releases. Current version is CoreOS {{site.beta-channel}}.</p>
      <p>Launch via NIFTY Cloud CLI by specifying <code>$ZONE</code>, <code>$TYPE</code>, <code>$FW_ID</code> and <code>$SSH_KEY_ID</code>:</p>
      <pre>nifty-run-instances $(nifty-describe-images --delimiter ',' --image-name "CoreOS Alpha {{site.beta-channel}}" | awk -F',' '{print $2}') --key $SSH_KEY_ID --availability-zone $ZONE --instance-type $TYPE -g $FW_ID -f cloud-config.yml</pre>
    </div>
    <div class="tab-pane active" id="stable">
      <p>The Stable channel should be used by production clusters. Versions of CoreOS are battle-tested within the Beta and Alpha channels before being promoted. Current version is CoreOS {{site.stable-channel}}.</p>
      <p>Launch via NIFTY Cloud CLI by specifying <code>$ZONE</code>, <code>$TYPE</code>, <code>$FW_ID</code> and <code>$SSH_KEY_ID</code>:</p>
      <pre>nifty-run-instances $(nifty-describe-images --delimiter ',' --image-name "CoreOS Alpha {{site.stable-channel}}" | awk -F',' '{print $2}') --key $SSH_KEY_ID --availability-zone $ZONE --instance-type $TYPE -g $FW_ID -f cloud-config.yml</pre>
    </div>
  </div>
</div>

### Adding More Machines

To add more instances to the cluster, just launch more with the same cloud-config and the appropriate firewall group.

## SSH

You can log in your CoreOS instances using:

```sh
ssh core@<ip address> -i <path to keyfile>
```

## Using CoreOS

Now that you have a machine booted it is time to play around.
Check out the [CoreOS Quickstart]({{site.url}}/docs/quickstart) guide or dig into [more specific topics]({{site.url}}/docs).