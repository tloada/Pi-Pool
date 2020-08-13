## Cross-building
Asegúrate que tengas instalada la Plataforma Haskell (<code>sudo apt install haskell-platform</code>). También <b>necesitarás</b> hacer esto un una máquina aarch64/ARM64 o simplemente en tu Raspberry Pi 4.<br>
Construiremos Cabal 3.0 en esta guía, pero puedes elegir cualquier versión que necesites.
```
wget http://hackage.haskell.org/package/cabal-install-3.0.0.0/cabal-install-3.0.0.0.tar.gz
tar -xf cabal-install-3.0.0.0.tar.gz
cd cabal-install-3.0.0.0
cabal update
cabal install --installdir=$HOME/.cabal/bin
```
Esto puede tomar algo de tiempo. Ahora revisa tu versión con <code>cabal --version</code>.
