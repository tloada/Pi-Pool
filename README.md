
---
**CRÉDITOS: >-
  ESTA GUÍA FUE HECHA POR ALESSANDRO KONRAD [(PI POOL [BERRY])](https://pipool.online/).
  LA GUÍA ORIGINAL EN INGLÉS LA PUEDES ENCONTRAR** [**AQUÍ**](https://github.com/tloada/Pi-Pool).
---
---
TRADUCIDA POR: >-
  ESTA GUÍA FUE TRADUCIDA POR [THE LEGEND OF ₳DA POOL [TLOA]](https://tloada.github.io/tloa/español.html).
  SI DESEAS APOYARNOS, PUEDES HACERLO DELEGANDO A NUESTRO POOL CON TICKER [[TLOA]](https://tloada.github.io/tloa/español.html).
---


<!-- <p align="center"><img width="80px" src="https://github.com/alessandrokonrad/Pi-Pool/blob/master/images/logo.svg"></img></p>

# [BERRY] Pi Pool

Pi Pool is a Cardano Stakepool on Raspberry Pi. Check out my <a href="https://pipool.online">website</a> to see more about my stakepool. You can support me by just delegating to my pool. I'm definitely around at mainnet release of Shelley with a stakepool!<br>
This repository is a guide to setup a stakepool on a Raspberry Pi by your own. -->

## El porqué de esta guía
Básicamente tenemos dos tipos diferentes de arquitectura de CPU conocidos. Primero consideremos solamnente máquinas de 64-bits. Muchos conocemos Intel y AMD. Ambos construyen sus CPUs en una arquitectura x86_64 principalmente. Por otro lado tenemos ARM, donde los CPUs están construidos en la arquitectura aarch64, y un Raspberry Pi tiene un CPU aarch64. Sin entrar en mucho detalle, el problema es que las guías para instalar Cardano-Node están hechas para máquinas x86_64 y actualmente no soportan aarch64 de manera predeterminada. El objetivo de esta guía es facilitar la instalación de Cardano-Node en un Raspberry Pi 4.

## Prerrquisitos

* Raspberry Pi 4 con 8GB RAM (la versión de 4GB solamente con la partición Swap como extra RAM) 
* Ubuntu 20.04 LTS <b>64-bit</b> (Fácil de instalar con <a href="https://www.raspberrypi.org/downloads/">Pi Imager (sition en inglés)</a>. Para ejecutar Ubuntu en una SSD, revisa la sección más adelante)

## Comencemos

Esta guía está actualizada para la Mainnet de Cardano. Luego de seguir ocn esta guía, puedes usar CNTools si lo deseas.


#### 1. Primerament actualicemos Ubuntu:
```
sudo apt-get update
sudo apt-get upgrade
```
Posteriormente, tal vez reinicies tu Pi.

#### 2. Instala las dependencias necesarias:
```
sudo apt-get install libsodium-dev build-essential pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make g++ tmux git jq wget libncursesw5 llvm -y

``` 
#### 3. Obtén la Plataforma Haskell:
```
sudo apt-get install -y haskell-platform
```
Ahra deberías tener GHC 8.6.5 y Cabal 2.4. Puedes revisar con <code>ghc --version</code> y <code>cabal --version</code>.
GHC 8.6.5 es la versión que necesitamos, pero necesitamos una versión de Cabal más reciente, Cabal (3.2).<br>

#### 4. Obtén Cabal 3.2 y remueve Cabal 2.4:
```
wget https://github.com/alessandrokonrad/Pi-Pool/raw/master/aarch64/cabal3.2/cabal
chmod +x cabal
mkdir -p ~/.cabal/bin
mv cabal ~/.cabal/bin
sudo rm /usr/bin/cabal
```
También puedes crear tu propio binario para Cabal aarch64. Revisa <a href="/Crossbuilding.md">aquí</a>.

#### 5. Agrega el nuevo Cabal al PATH:

Abre el archivo .bashrc en tu directorio de inicio y al final agrega lo siguiente:
```
export PATH="~/.cabal/bin:$PATH"
```
Para activar el nuevo PATH ouedes reiniciar el Pi o teclear <code>source .bashrc</code> desde tu directorio de inicio. Luego ejecuta:
```
cabal update
```
<br>

¡Ahora estamos listos para construir el Cardano-Node!

#### 6. Clona el repertorio Github de cardano-node y constrúyelo (esto toma algo de tiempo):
```
git clone https://github.com/input-output-hk/cardano-node.git
cd cardano-node
echo -e "package cardano-crypto-praos\n  flags: -external-libsodium-vrf" > cabal.project.local
git fetch --all --tags
git checkout tags/1.18.1
cabal build all
cp -p dist-newstyle/build/aarch64-linux/ghc-8.6.5/cardano-node-1.18.1/x/cardano-node/build/cardano-node/cardano-node ~/.cabal/bin/
cp -p dist-newstyle/build/aarch64-linux/ghc-8.6.5/cardano-cli-1.18.1/x/cardano-cli/build/cardano-cli/cardano-cli ~/.cabal/bin/

```
El tag que usamos aquí es 1.18.1. Revisa el tag más reciente y utiliza ese.
Finalmente tenemos nuestro nodo. Si todo funcionó debidamente, deberías de podes teclear <code>cardano-cli</code> y <code>cardano-node</code>.

#### 7. Ejecutando un nodo:

We need first of all some configuration files:
```
mkdir pi-node
cd pi-node
wget https://hydra.iohk.io/build/3644329/download/1/mainnet-config.json
wget https://hydra.iohk.io/build/3644329/download/1/mainnet-byron-genesis.json
wget https://hydra.iohk.io/build/3644329/download/1/mainnet-shelley-genesis.json
wget https://hydra.iohk.io/build/3644329/download/1/mainnet-topology.json

```
You can change "ViewMode" from "SimpleView to "LiveView" in mainnet-config.json to get a fancy node monitor.<br>
Now start the node:
```
cardano-node run \
   --topology mainnet-topology.json \
   --database-path db \
   --socket-path db/socket \
   --host-addr 0.0.0.0 \
   --port 3000 \
   --config mainnet-config.json
```

That's it. Your node is now starting to sync!

<b>Note:</b> The syncing process for the mainnet blockchain can take really long. My node also crashed sometimes during the syncing process. Having a backup machine (x86_64) where you sync the node and just copy the db on the Raspberry Pi, makes it much easier and faster. As soon as the Pi is in sync it runs really smooth and just uses about 5% of the CPU. 


## Setup a stakepool
I might create a detailed guide soon, on how to register a stakepool. Anyway there are plenty tutorials out there: <br />
I can recommend <a href="https://cardano-community.github.io/guild-operators/Scripts/cntools.html">CNTools</a> (make sure the CNTools version is compatible with the Cardano-Node version).<br />
Otherwise I would follow the official guide of <a href="https://cardano-foundation-cardano.readthedocs-hosted.com/en/latest/getting-started/stake-pool-operators/index.html">cardano.org</a>

## Run Ubuntu on a SSD
#### Running Ubuntu from SSD, while booting from SD card:

1. Flash the Ubuntu image on your SSD and your SD card.
2. Now go to to the boot partition of the SD card and change in cmdline.txt the root path to: <code>root=/dev/sda2</code>
3. Insert the SD card into the Pi and the SSD into one of the USB 3.0 ports.
This should boot now from the SD card, but the OS will run on the SSD then.

#### Running and booting from SSD (no need for SD card):

I recently saw a post, where you could boot Ubuntu also from the SSD, so there is no need for the SD card anymore. I haven't tried that yet, but it might work. You can check that out:
<a href="https://www.raspberrypi.org/forums/viewtopic.php?t=278791">Directly boot from SSD</a>

#### Problems with running Ubuntu from USB 3.0:
<a href="https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/">Adding quirks to your chipset, if it's not working</a>

## Cross-building
If you want to build your own Cabal binary for aarch64 or a different version of Cabal, follow <a href="/Crossbuilding.md">this</a> guide.


## Port forwarding
Go to your router settings. You can access them via your browser with the IP address of the router (e.g. 192.168.178.1 or if you have a FritzBox with fritz.box).
Then look for a option Port Forwarding. Choose the IP address of your relay node(s) and open its/their port(s). Allow TCP and UDP. Then save it and that's it.
