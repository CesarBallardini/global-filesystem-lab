# Instalación sobre Debian Buster 


El box de Debian Buster viene con las Virtualbox Guest Additions antiguas, y un kernel antiguo.  En esas condiciones, el plugin de
instalación de VB Guest Adds. no funciona.  Esta pequeña receta resuelve el problema.


Sustituimos el box de Debian, en lugar del de Ubuntu:

```ruby
    srv.vm.box = "debian/buster64"

```

```bash
# crea por primera vez la VM, instala un kernel nuevo (el que viene en el box ya no tiene los headers en los repos y falla la instalaci{on de las VB Guests)
NEW_VALUE=true ; grep --silent  "^[ ]*primera_vez[ ]*=[ ]*${NEW_VALUE}" Vagrantfile  || sed -i "s/^[ ]*primera_vez[ ]*=.*/primera_vez = ${NEW_VALUE}/" Vagrantfile
time vagrant up

# reinicio para que se use el nuevo kernel
time vagrant reload

# instalo las VB Guest Additions
time vagrant vbguest --do install

# ahora si puedo tener el vagrant-cachier y los synced-folders funcionando, cambio primera_vez a false
NEW_VALUE=false ; grep --silent  "^[ ]*primera_vez[ ]*=[ ]*${NEW_VALUE}" Vagrantfile  || sed -i "s/^[ ]*primera_vez[ ]*=.*/primera_vez = ${NEW_VALUE}/" Vagrantfile

# reinicio para que se active el synced folder
time vagrant reload

```


En una línea queda:

```bash
NEW_VALUE=true ; grep --silent  "^[ ]*primera_vez[ ]*=[ ]*${NEW_VALUE}" Vagrantfile  || sed -i "s/^[ ]*primera_vez[ ]*=.*/primera_vez = ${NEW_VALUE}/" Vagrantfile ; \
time vagrant up ; \
time vagrant reload ; \
time vagrant vbguest --do install ; \
NEW_VALUE=false ; grep --silent  "^[ ]*primera_vez[ ]*=[ ]*${NEW_VALUE}" Vagrantfile  || sed -i "s/^[ ]*primera_vez[ ]*=.*/primera_vez = ${NEW_VALUE}/" Vagrantfile ; \
time vagrant reload
```
