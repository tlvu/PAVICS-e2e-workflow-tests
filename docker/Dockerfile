FROM continuumio/miniconda3

RUN conda update conda

# to checkout other notebooks and to run pip install
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y git mercurial gcc && \
    apt-get clean

COPY environment.yml /environment.yml

# create env "birdy"
RUN conda env create -f /environment.yml

# needed for our specific jenkins
RUN groupadd --gid 1000 jenkins \
    && useradd --uid 1000 --gid jenkins --create-home jenkins

# alternate way to 'source activate birdy'
ENV PATH="/opt/conda/envs/birdy/bin:$PATH"

# our notebooks are hardcoded to lookup for kernel named 'birdy'
# this python is from the birdy env above
RUN python -m ipykernel install --name birdy

# install using same channel preferences as birdy original env to not downgrade
# anything accidentally
# this is for debug only, all dependencies should be specified in
# environment.yml above
# RUN conda install -c conda-forge -c cdat -c bokeh -c defaults nbdime

# build jupyterlab extensions installed by conda, see `jupyter labextension list`
RUN jupyter lab build

# for ipywidgets to work with jupyter lab (notebooks works out of the box)
RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager \
    && jupyter serverextension enable voila --sys-prefix \
    && jupyter labextension install @jupyter-widgets/jupyterlab-manager jupyter-leaflet \
    && jupyter labextension install @bokeh/jupyter_bokeh \
    && jupyter labextension install @pyviz/jupyterlab_pyviz

ADD https://raw.githubusercontent.com/jupyter/docker-stacks/master/base-notebook/start.sh /usr/local/bin/
ADD https://raw.githubusercontent.com/jupyter/docker-stacks/master/base-notebook/start-singleuser.sh /usr/local/bin/
ADD https://raw.githubusercontent.com/jupyter/docker-stacks/master/base-notebook/start-notebook.sh /usr/local/bin/
ADD https://raw.githubusercontent.com/jupyter/docker-stacks/master/base-notebook/fix-permissions /usr/local/bin/
ADD https://raw.githubusercontent.com/jupyter/docker-stacks/master/base-notebook/jupyter_notebook_config.py /etc/jupyter/
RUN chmod a+rx /usr/local/bin/start.sh /usr/local/bin/start-singleuser.sh /usr/local/bin/start-notebook.sh /usr/local/bin/fix-permissions; \
    chmod a+r /etc/jupyter/jupyter_notebook_config.py

# problem running start-notebook.sh when being root
# the jupyter/base-notebook image also do not default to root user so we do the same here
USER jenkins

# follow jupyter/base-notebook image so config in jupyterhub is simpler
CMD ["/usr/local/bin/start-notebook.sh"]
