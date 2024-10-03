# CP/NET & MP/M Client-Server simulation

This is Following my research documented in my video
on [MP/M & CP/NET Networking Uncovered](https://www.youtube.com/watch?v=YI2rXPHUQ9c).

I wanted to get into CP/NET and MP/M networking,
and before tampering around with the real hardware,
get my stuff up and running using Udo Munk's [z80pack](https://github.com/udo-munk/z80pack) simulator.

What I wanted to achive, is a client-server simulation,
with one MP/M server, and four CP/NET clients as depicted here:

![Simulation Setup](doc/simulation_setup.png)

Building this was not as straight forward with `z80pack` and `cpmsim`,
as it needs some alterations, furtherly explained below.

But let's start first with what's contained in this repository:

- 1 set of MP/M 2.0 boot disks
 - sourced from [z80pack](https://www.icl1900.co.uk/unix4fun/z80pack/#download)
 - was recompiled using SYSGEN to have only 1 (instead of 3) TMPs
 - includes a patched and recompiled NETWRKIF.RSP to support 4 network connections
- 4 sets of CP/M 2.2 + CP/NET 1.2 boot disks
 - sourced from [z80pack](https://www.icl1900.co.uk/unix4fun/z80pack/#download)
 - includes a patched and recompiled SNIOS.RSP with a Unique Network Requestor ID per each disk
- The [cpnet](./cpnet) startup script, to run the simulation


# Prerequisites

You need:

- [z80pack](https://github.com/udo-munk/z80pack)
- [tmux](https://github.com/tmux/tmux)

Plus, of course, this repository.
Place all the files directly into your ´./z80pack/cpmsim` directory, i.e. something like this:

```
cd ./z80pack/cpmsim
https://github.com/gpdm/TPC-CPNET-simulation/archive/refs/heads/main.zip
bsdtar xfv main.zip --strip-components=1
rm main.zip
chmod 755 ./cpnet
```

Note: You may not have the `bsdtar` available. So when extracting, make sure to strip away
the `TPC-CPNET-simulation` directory.

All files should be inside the `/z80pack/cpmsim` directory.

# Run it!

It's as simple as this, run the command from the `/z80pack/cpmsim` directory:

`./cpnet`

This will give you something like this:

![Simulation Startup](doc/simulation_startup.png)

And eventually you'll end up here:

![Simulation](doc/mpm_server_ready.png)


# How it works

The simulation runs 1 server and 4 clients in the background.
They're launched into a `tmux` session, so you can switch through the consoles
using the usual `tmux` hotkeys, i.e.

- ^B-N (next)
- ^B-P (previous)
- `B+(0-4) for the indivual consoles
 - console 0 is the MP/M server
 - consoles 1-4 are the CP/NET clients 1-4

To exit from the simulation, simply type `bye` into each console window,
then press `^-C` to exit.

Note: If you don't press `^-C` the simulation on that console will restart.


# Setting it into Motion

All simulations will boot directly into CP/M, however they will not yet
startup MP/M or CP/NET automatically.

I read how this can be patched for automatic startup, but it's currently *not* implemented yet.

So you need some manual steps to set it in motion.


## MP/M Server

This is only quick and dirty, for all details, please refer to the MP/M manual.

1. start MP/M by running `mpmldr`
2. That's it. You can get some stats using the `mpmstats` command


## CP/NET Clients

Again, refer to the CP/NET manual for all the details.

1. start CP/NET by running `cpnetldr`
2. run `login` (no passwords required)
3. map the network drives from the server to the client:

   ```
   network C:=A:
   network D:=B:

4. Stats command is `cpnetsts`

Want to send mail?

Check the `cpnetsts` command. It provides the requester ID,
which is `11h` for the first client, `12h` for the second,
`13h` for the third, and `14h` for the fourth.

Run mail like this:

```
D:
mail
```

When asked for a recipient, use one of the requester IDs named above.
There's no user accounts, or anything, so that's the only way to
direct a mail message at a given client.

That's it!


# Behind the Scenes

The simulation rewrites the config files within the `./z80pack/cpmsim/conf` directory,
notably:

- `net_client.conf`
- `net_server.conf`

It does this to enable the MP/M server to run 4 listening serial consoles via a TCP socket,
and each client to allow for connecting to one of these TCP sockets for a simulated
serial connection.


# Applied Patches

The original MP/M images provided by Udo Munk were changed to allow 4 ingress serial line connections.
For that, the `NETWRKIF.ASM` file was patched to refer to the serial interfaces
0000h, 0100h, 0200h and 0300h in the source code.

Furtherly, `SYSGEN` was used to recompile the kernel, effectively reducing the TMPs (Terminal Message Processors)
from original 3 down to 1.

References are provided in the [MP/M patches](patches/mpm) directory.


As for the 4 CP/M + CP/NET disk images, these include changes in the `SNIOS.ASM` file,
essentially changing the network requestor ID to a unique value.

Without that, all simulations would run with the default ID of 11h, which would break
functions like CP/NET mail, which rely on having unique requestor IDs.

References are provided in the [CP/NET patches](patches/cpnet) directory.


# References

- [z80pack sources](https://github.com/udo-munk/z80pack)
- [z80pack](https://www.icl1900.co.uk/unix4fun/z80pack)
- [An Analysis of CP/NET](https://dl.acm.org/doi/pdf/10.1145/800219.806658)
- [MP/M Users Guide](http://www.cpm.z80.de/manuals/mpm2ug.pdf)
- [MP/M Implementors Guide](https://bitsavers.trailing-edge.com/pdf/digitalResearch/mpm_II/MPM_II_System_Implementors_Guide_Aug82.pdf)
- [CP/NET manual: Overview](http://sebhc.durgadas.com/CPNET-docs/cpnet.html#Fig1_1)
- [CP/NET manual: Notes about Mail](http://sebhc.durgadas.com/CPNET-docs/cpnet.html#Sec2_10)
- [CP/NET manual: NETWRKIF.ASM original source](http://sebhc.durgadas.com/CPNET-docs/cpnet.html#ListE_2)
- [CP/NET manual: SNIOS source](http://sebhc.durgadas.com/CPNET-docs/cpnet.html#SecE_4)
- [Unofficial CP/M website, source](http://www.retroarchive.org/cpm/archive/unofficial/source.html)
