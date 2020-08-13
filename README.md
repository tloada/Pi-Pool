
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

## Prerrequisitos

* Raspberry Pi 4 con 8GB RAM (la versión de 4GB solamente con la partición Swap como extra RAM) 
* Ubuntu 20.04 LTS <b>64-bit</b> (Fácil de instalar con <a href="https://www.raspberrypi.org/downloads/">Pi Imager (sitio en inglés)</a>. Para ejecutar Ubuntu en una SSD, revisa la sección más adelante)

## Comencemos

Esta guía está actualizada para la Mainnet de Cardano. Luego de seguir con esta guía, puedes usar CNTools si lo deseas.


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
**El tag que usamos aquí es 1.18.1. Revisa el tag más reciente y utiliza ese.**<br>
Finalmente tenemos nuestro nodo. Si todo funcionó debidamente, deberías de poder teclear <code>cardano-cli</code> y <code>cardano-node</code>.

#### 7. Ejecutando un nodo:

Primeramente, necesitamos los archivos de configuración (**asegúrate que éstos sean los más recientes**):
```
mkdir pi-node
cd pi-node
wget https://hydra.iohk.io/build/3644329/download/1/mainnet-config.json
wget https://hydra.iohk.io/build/3644329/download/1/mainnet-byron-genesis.json
wget https://hydra.iohk.io/build/3644329/download/1/mainnet-shelley-genesis.json
wget https://hydra.iohk.io/build/3644329/download/1/mainnet-topology.json

```
Puedes cambiar el "ViewMode" de "SimpleView a "LiveView" en el archivo mainnet-config.json para obtener un monitor sofisticado para tu nodo.<br>
Ahora inicia el nodo:
```
cardano-node run \
   --topology mainnet-topology.json \
   --database-path db \
   --socket-path db/socket \
   --host-addr 0.0.0.0 \
   --port 3000 \
   --config mainnet-config.json
```

Eso es todo. ¡Tu nodo ahora comenzará a sincronizarse!

<b>Nota:</b> El proceso de sincronización a la blockchain de la mainnet tardará un largo tiempo. Pueda que tu nodo se quiebre algunas veces durante el proceso de sincronización. Puedes tener un respaldo en una máquina x86_64 donde puedas sincronizar tu nodo y de ahí solamente copiar la carpeta db en tu Raspberry Pi, lo cual lo hace mucho más fácil y rápido. En cuanto el Pi esté completamente sincronizado no tendrá problema alguno y ejecutará tu nodo de manera correcta, usando aproximadamente el 5% del CPU. 


## Inicia tu stake pool
Puedes chequear <a href="https://github.com/tloada/coincashew/tree/master/coins/overview-ada/guide-how-to-build-a-haskell-stakepool-node">esta guía</a>. Recuerda que esta guía mencionada es para máquinas x86_64, por lo cual es recomendable que la leas y comprendas antes de utilizarla para iniciar tu pool. Esto te ayudará a comprender la ubicación de las carpetas y archivos utilizados en la guía.<br />
De lo contrario te recomiendo la guía oficial (en inglés) de <a href="https://cardano-foundation-cardano.readthedocs-hosted.com/en/latest/getting-started/stake-pool-operators/index.html">cardano.org</a>

## Ejecuta Ubuntu en una SSD
#### Ejecutando Ubuntu en una SSD, e iniciando tu Pi con una microSD:

1. Instala la imagen de Ubuntu image <a href="https://www.raspberrypi.org/downloads/">(Pi Imager sitio en inglés)</a> en tu SSD y en tu microSD.
2. Dirígete a tu partición de inicio de la micrSD y en **cmdline.txt** cambia el **root path a: <code>root=/dev/sda2</code>**
3. Introduce la microSD en el Pi y la SSD en uno de los puertos USB 3.0.
Esto debería iniciar tu Raspberry Pi desde la microSD, pero el OS (Sistema Operativo) será ejecutando en la SSD.

#### Ejecutando e iniciando en una SSD (no necesitas microSD):

Aquí puedes encontrar cómo hacerlo. El sitio está en inglés. No he intentado de esta forma, pero este es el enlace para <a href="https://www.raspberrypi.org/forums/viewtopic.php?t=278791">iniciar tu Pi directamente en la SSD</a>

#### Problemas al ejecutar Ubuntu desde USB 3.0 (sitio en inglés):
<a href="https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/">Añadiendo quirks a tu chipset, si no está funcionando</a>

## Cross-building
Si quieres construir tu propio binario de Cabal para aarch64 o una versión diferente de Cabal, sigue <a href="/Crossbuilding.md">esta guía</a>.


## Port forwarding
Ve a las configuraciones de tu router. Puedes accesarlas desde tu explorador con la dirección IP de tu router (e.g. 192.168.178.1).
Luego busca la opción de **Port Forwarding**. Elige la dirección IP de tu(s) nodo(s) de relevo y habilita su(s) puerto(s). Permite TCP y UDP. Guarda los cambios y listo.
