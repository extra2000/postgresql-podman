Production Deployment
=====================

Example how to deploy PostgreSQL for production environment.

Clone repositories
------------------

Execute the following command to clone ``postgresql-podman`` repository into your ``~/extra2000`` directory:

.. code-block:: bash

    mkdir ~/extra2000
    cd ~/extra2000
    git clone --recursive https://github.com/extra2000/postgresql-podman.git

Then, ``cd`` into project root directory:

.. code-block:: bash

    cd postgresql-podman

Deploy PostgreSQL
-----------------

From the project root directory, ``cd`` into ``deployment/production/postgresql``:

.. code-block:: bash

    cd deployment/production/postgresql

Create config files and fix permission:

.. code-block:: bash

    cp -v configmaps/postgresql.yaml{.example,}
    cp -v configs/postgresql.conf{.example,}
    cp -v configs/pg_hba.conf{.example,}
    chmod og+r ./configs/postgresql.conf ./configs/pg_hba.conf

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t configs

Create pod file:

.. code-block:: bash

    cp -v postgresql-pod.yaml{.example,}

Load SELinux security policy:

.. code-block:: bash

    sudo semodule -i selinux/postgresql_podman.cil /usr/share/udica/templates/base_container.cil

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "postgresql_podman"

Deploy postgresql:

.. code-block:: bash

    podman play kube --configmap configmaps/postgresql.yaml --seccomp-profile-root ./seccomp postgresql-pod.yaml

Test postgresql. Make sure the following command success:

.. code-block:: bash

    podman exec -it postgresql-pod-srv01 psql -h 127.0.0.1 -Upostgres

Create systemd files to run at startup:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name postgresql-pod-srv01
    systemctl --user enable container-postgresql-pod-srv01.service
