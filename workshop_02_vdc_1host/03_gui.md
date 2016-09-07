# Exercise: Install the GUI

## Goals

* Get a GUI to pretty up Wakame-vdc a bit

## Assignment

You should still be logged into the machine labeled `Wakame1`. If not, use your ssh client to log into it.

First install the GUI package.

```
sudo yum install -y wakame-vdc-webui-vmapp-config
```

Copy the config files to their correct locations.

```
sudo cp /opt/axsh/wakame-vdc/frontend/dcmgr_gui/config/database.yml.example /etc/wakame-vdc/dcmgr_gui/database.yml

sudo cp /opt/axsh/wakame-vdc/frontend/dcmgr_gui/config/dcmgr_gui.yml.example /etc/wakame-vdc/dcmgr_gui/dcmgr_gui.yml

sudo cp /opt/axsh/wakame-vdc/frontend/dcmgr_gui/config/instance_spec.yml.example /etc/wakame-vdc/dcmgr_gui/instance_spec.yml

sudo cp /opt/axsh/wakame-vdc/frontend/dcmgr_gui/config/load_balancer_spec.yml.example /etc/wakame-vdc/dcmgr_gui/load_balancer_spec.yml
```

The GUI comes with its own database that needs to be initialized and filled. Create the database and then use the `gui-mange` tool to populate it.

Create the database:

```
mysqladmin -uroot create wakame_dcmgr_gui
cd /opt/axsh/wakame-vdc/frontend/dcmgr_gui/
/opt/axsh/wakame-vdc/ruby/bin/rake db:init
```

Use `gui-manage` to create some required records.

```
/opt/axsh/wakame-vdc/frontend/dcmgr_gui/bin/gui-manage

account add --name default --uuid a-shpoolxx
user add --name "demo user" --uuid u-demo --password demo --login-id demo
user associate u-demo --account-ids a-shpoolxx

exit
```

Start the GUI

```
sudo start vdc-webui
```

The GUI will run on port tcp port 9000 by default. You can surf to it in a web browser to see it. In the training environment we have port forwarding set up so you can reach it from your own laptop.

Log in using user `demo` and password `demo`.

You can check out the [basic usage guide](http://wakame-vdc.org/usage/) on wakame-vdc.org to learn more about using the GUI.
