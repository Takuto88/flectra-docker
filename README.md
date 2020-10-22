# Flectra Docker Development Setup

This is my little dev-setup for developing Flectra addons while having Flectra installed & configured in Docker and not
having it installed on my local machine.

Once you've got this docker setup going, just start the development of your addon in the `addons/` sub-directory.
It will reside in `/mnt/extra-addons` from within the docker container.

## Setup

### Prerequisites

- Docker >= 19.03
- Docker-Compose >= 1.25.5

Recommended IDE: PyCharm Professional (any recent version should do)

### Starting the container

Simply run `docker-compose up -d` to start Flectra. After that, you can visit `http://localhost:8080` to login to
Flectra. The default username + password is "admin / admin".

### ℹ️ First-time issues

On the very first time, your visiting Flectra from your browser might not work. Sypmtom: You just get a error like
`ERR_EMPTY_RESPONSE` from your browser. To fix this, simply run this:

```bash
docker-compose exec -u root flectra chown flectra:flectra /opt/flectra/.local/share/Flectra
```

You'll only need to do this once. Afterwards, this change persists between restarts.

#### What this does

(For documentation purposes - if you don't care feel free to skip this) 

As far as I can tell, this is an issue that needs fixing in the Flectra upstream docker container.  The issue is, that
`/opt/flectra/.local/share/Flectra` is where Flectra stores data like session information, compiled assets like CSS and
JS etc. This must persist between restarts of your container so it is declared as a named volume in
`docker-compose.yml`. This volume belongs `root` by default but Flectra runs as user `flectra` thus the volume is not
writable to Flectra. The command above changes the ownership of that volume to the `flectra` user so that Flectra can
write to this directory again.

### Using a debugger through Docker 🐳

FIXME: This is currently very Mac specific. On Linux, things may or may not be similar (untested).
Also, this only works with PyCharm Professional. PRs welcome.

#### Step 1: Open your addon

Please note: Your project **MUST** reside in the `addons/` subdirectory of this repository. Place your addon in the
`addons/` directory and open it with PyCharm Professional.
 
#### Step 2: Connect PyCharm to Docker

Open the PyCharm Settings, search for "Docker". Under "Build, Execution & Deployment", select "Docker". Click the "+"
icon to add a new Connection if you don't have a connection already and setup the connection. On a Mac, this looks
like this:

![docker-connection][pycharm-docker]

Now, PyCharm is able to start and stop Docker containers - in principle. We can't to much with that yet, we need to
tell PyCharm more. Like what exactly to start and stop. So let's continue.

#### Step 3: Creating a remote interpreter for your project

PyCharm needs to be made aware that we have a Python interpreter in Docker that it can use to provide IDE functionality
from inside the Docker container. To do this, jump into the PyCharm settings again and search for "Project Interpreter".

You should find a dialog where you can click a gear icon for more advanced project settings. Click "Show all":

![project-interpreter][pycharm-project-interpreter]

A new dialog will pop up where you can add new interpreters by clicking the "+" sign:

![pycharm-project-interpreter-add][pycharm-project-interpreter-add]

Here you can Configure the docker interpreter. On the left side choose the "Docker Compose" interpreter type. Make the
following settings:

- Choose the Docker server connection you have created earlier. In this example, this is "Docker 4 Mac"
- Add **only** these configuration files from the flectra-docker repository in **exactly that order**
   - `docker-compose.yml` 
   - `docker-compose.override.yml` (If you have one - it's .gitignored)
- Under "Service", select the  `flectra` service. 
- The python interpreter path must be `python3`.

The end result should look like this:

![pycharm-docker-interpreter][pycharm-docker-interpreter]

We're almost done. Click "OK" in the dialog shown above to add the interpreter. On the list of project interpreters,
also click "OK", so you are now back in the Project interpreter settings.

Under "Project interpreter" select your newly created remote interpreter. Also, make sure to create a path mapping.
What's that? Since we will run your addon in flectra's docker container, the path of your source code is different
inside the container from what is visible on your computer. But the IDE needs to translate between these paths when
doing things like adding debug breakpoints for instance. 

Everything that is in `addons/your-project-name` will live in `/mnt/extra-addons/your-project-name` inside the
container. So setup a path mapping, that maps between these. For Example like this:

![pycharm-docker-path-mapping][pycharm-docker-path-mapping]

Once that is done, click "OK" to save your settings.

#### Step 4: Run / Debug App within PyCharm

Now for the last step: We need to be able to launch Flectra from PyCharm. This will allow us to attach
a debugger when needed.

Open the "Run / Debug" configuration (located in the toolbar in the top-right by default) and click the "+" to add
a new Run configuration. Select "Python" and configure it as follows:

![pycharm-run-cfg][pycharm-run-cfg]

Please note the following:

- We need to use the remote Docker Compose Python interpreter we created earlier
- Make sure to launch the `/opt/flectra/flectra-bin` script.
- Make sure to use these parameters: `--db_host=db --db_port=5432 --db_user=flectra --db_password=flectra`
- Make sure the path mapping is correct (like the one created earlier)
- Note that none of the checkboxes are ticked in the screenshot above.

And that is it. Now you can save your new Run Configuration and use it. You'll still need `docker-compose up -d` in
order to start the Postgres database but after that, simply click "Run" in the toolbar to run Flectra
or click "Debug" to start it in debug-mode. That will also allow you to add breakpoints and use PyCharm's
integrated debugger.

[pycharm-docker]: docs/pycharm-docker.png "PyCharm Docker connection on a Mac"
[pycharm-project-interpreter]: docs/pycharm-project-interpreter.png "Project interpreter settings"
[pycharm-project-interpreter-add]: docs/pycharm-project-interpreter-add.png "Dialog showing all project interpreters"
[pycharm-docker-interpreter]: docs/pycharm-docker-interpreter.png "Docker remote interpreter setup"
[pycharm-docker-path-mapping]: docs/pycharm-docker-path-mapping.png "Interpreter path mapping"
[pycharm-run-cfg]: docs/pycharm-run-cfg.png "Python run config"