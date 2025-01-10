---
date: "2024-09-19T21:10:21-08:00"
draft: false
title: "A Simple Guide to Name Resolution"
cover:
  image: "https://storage.googleapis.com/timengledev-blog-bucket/static/dist/img/icon_sm.png"
---

Network management requires machines to be able to communicate efficiently, and
one of the simplest ways to achieve this is via domain names. But how does a
machine know where to send traffic for any particular domain name?
Without proper name resolution, communication becomes challenging—at best
requiring humans to memorize IP addresses, and at worst causing a complete
communication failure. By the end of this article, you will know how to
configure name resolution on a Linux machine and ensure seamless
communication across your network.

---

Let's assume we have a network with the following machines:

- Host A: `192.168.1.1`
- Host B: `192.168.1.11`

These machines are all on the same network, with the switch address at `192.168.1.0`.

Now, when we ping Host B from inside Host A, it responds. However, we'd like to
be able to ping `db` directly using the following command:

```bash
ping db
```

Currently, running this command results in:

```shell
ping: unknown host db
```

### Understanding Name Resolution

In this example, we want Host A to resolve `db` to Host B's IP address.
To accomplish this, we have two common options: the older method of adding an
entry to the `/etc/hosts` file, and the more modern method of using a DNS
server.

#### Local Name Resolution: `/etc/hosts`

For name resolution of `db` to Host B's IP address, add the following entry in
the `hosts` file on Host A at `/etc/hosts`:

```shell
echo 192.168.1.11 db >> /etc/hosts
```

Now, when `db` is pinged, the request goes to `192.168.1.11` (Host B's IP address).
In this case, Host A uses `/etc/hosts` as the source of truth and doesn't care
what Host B calls itself.

**\*Implications**: This approach allows us to manually configure any domain name
to any IP address. For example, we could map
`www.google.com` to `192.168.1.11`, and every time we try to go to
`www.google.com` from Host A, it would actually go to Host B, not Google.\*

#### DNS Servers: The Modern Approach

As networks grew, managing name resolution by editing the `/etc/hosts` file on
each individual machine became too cumbersome. This is where
DNS (Domain Name System) servers come into play. DNS servers act as a
centralized source of truth for all machines on a particular network.

### How to Point a Host at a DNS Server

Each machine has a **DNS Resolution Configuration File** located at
`/etc/resolv.conf`. To point your machine to a DNS server, you can edit
this file as follows:

```shell
echo nameserver 192.168.1.100 >> /etc/resolv.conf
```

This command tells the machine to use the DNS server at `192.168.1.100` to
resolve hostnames to IP addresses.

### What if There is a Conflict?

When both `/etc/hosts` and the DNS server contain the same name, the host will
always look at `/etc/hosts` first. If the name is not found there,
it then refers to the DNS server.

_Important_: This behavior can be modified by editing the `/etc/nsswitch.conf` file.
The default configuration is:

```shell
hosts:   files dns
```

Here, `files` refers to `/etc/hosts`, and `dns` refers to the DNS server. If you
reverse the order, the host will refer to the DNS server before consulting
`/etc/hosts`.

### What if the Name is in Neither?

If the name cannot be found in either `/etc/hosts` or the DNS server, the host
will not be able to resolve it, resulting in an error like "unknown host."

### Using Public DNS Servers

A common choice is Google's public DNS server, `8.8.8.8`. You can add it to
your `/etc/resolv.conf` file as follows:

```shell
echo nameserver 8.8.8.8 >> /etc/resolv.conf
```

### Abbreviating Subdomains

Let’s say your company owns the domain `mycompany.com`, with subdomains like:

- `web`
- `www`

Typically, to access these subdomains, users would need to enter the full
domain like `http://web.mycompany.com`. However, on your local network, you
can configure your machines to resolve subdomains with just the
short name (`web`), by adding a `search` entry in the `/etc/resolv.conf` file:

```shell
echo search mycompany.com >> /etc/resolv.conf
```

Now, simply entering `web` will resolve to `web.mycompany.com`.

You can even add multiple search domains:

```shell
search mycompany.com prod.mycompany.com
```

Linux will search through both domains in order before notifying you that the
name cannot be resolved.

---

### Conclusion

Whether you’re managing a small network or a large one, understanding how name
resolution works and how to configure it properly can help you streamline
communication between hosts. While the traditional `/etc/hosts` file method
still works, modern networks rely heavily on DNS servers for scalable,
centralized name resolution.

By leveraging both methods, you can optimize how machines in your network
resolve names, improving efficiency and reducing potential conflicts.
