**Lab shared Conda environments on Roar**

Shared environments: `/storage/group/efc5279/default/conda_envs`

Use the lab's shared environments to avoid installing the same tools separately in each account.

**1. Initialize Conda and configure the shared location**

On Roar, run:

```bash
source /storage/icds/RISE/sw8-compat/mambaforge/etc/profile.d/conda.sh
conda config --prepend envs_dirs /storage/group/efc5279/default/conda_envs
conda config --show envs_dirs
```

The `conda config` setting is saved in your own configuration; run it once. It puts the shared directory first in the configured environment search list. If you previously appended this directory, prepending it moves it to the beginning. Existing environment-variable overrides can affect the effective order; check the displayed configuration.

For automatic initialization in future Bash sessions, add the `source` line once to your `~/.bashrc`. If you already initialize a working Conda installation there, you can keep that initialization and run just the configuration command with it. Avoid initializing several different Conda installations in the same shell.

**2. Use an environment**

```bash
conda activate bbmap
echo "$CONDA_PREFIX"
```

The prefix should be `/storage/group/efc5279/default/conda_envs/bbmap`. The full-path form selects the shared copy explicitly:

```bash
conda activate /storage/group/efc5279/default/conda_envs/bbmap
```

List environments with `conda env list`. Finish with `conda deactivate`.

In a Slurm script, after your usual resource directives, include:

```bash
# resources directives

source /storage/icds/RISE/sw8-compat/mambaforge/etc/profile.d/conda.sh
conda activate /storage/group/efc5279/default/conda_envs/bbmap
  
# Your analysis commands follow here.
```
  
**3. Create a new shared environment**

Check the existing environment list before creating one. Choose a descriptive tool-and-version name and coordinate with the lab so two people do not create the same path at once. Each environment has an owner who maintains it. For a new version or different dependency requirements, create a new environment with a distinct name; this keeps existing analyses reproducible.

The following is a runnable **example** for a new Python 3.11 environment. Run it only if the lab needs this environment, inside a compute allocation or batch job:

```bash
(
    set -e
    source /storage/icds/RISE/sw8-compat/mambaforge/etc/profile.d/conda.sh
    env_prefix=/storage/group/efc5279/default/conda_envs/lab-python-3.11

    if test -e "$env_prefix" || test -L "$env_prefix"; then
        echo "Environment path already exists; inspect it before proceeding."
        exit 1
    fi

    umask 0027
    conda create --yes --copy --override-channels -c conda-forge \
        --prefix "$env_prefix" python=3.11

    # Apply permissions only to the environment you just created.
    # find does not follow symlinks; chgrp -h changes the link itself.
    find "$env_prefix" -exec chgrp -h efc5279_collab {} +
    find "$env_prefix" -type d -exec chmod g+rx,g-w,o-rwx {} +
    find "$env_prefix" -type f -exec chmod g+rX,g-w,o-rwx {} +
    echo "Created $env_prefix; test it from another lab account."
)
```

For another tool, adapt the name, package specifications and required channels. Use `--copy` with this permission recipe so changes to permissions do not affect files linked to the package cache. Keep these permission commands scoped to the environment you own. 

After any maintenance, apply the same group/permission commands to newly installed files and run a small representative test. 
