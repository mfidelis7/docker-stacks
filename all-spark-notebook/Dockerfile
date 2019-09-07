# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
ARG BASE_CONTAINER=jupyter/pyspark-notebook
FROM $BASE_CONTAINER

LABEL maintainer="Marco Fidelis <mfidelis@pobox.com>"

USER root

# RSpark config
ENV R_LIBS_USER $SPARK_HOME/R/lib
RUN fix-permissions $R_LIBS_USER

# R pre-requisites
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    fonts-dejavu \
    gfortran \
    gcc && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
		ca-certificates \
		curl \
		wget \
                vim \
                less \
                bzip2 \
                git \
                libgl1-mesa-glx \
        && apt-get clean \
        && rm -rf /var/lib/apt-get/lists/* /tmp/* /var/tmp/*

USER $NB_UID

# R packages
RUN conda install --quiet --yes \
    'r-base=3.6.1' \
    'r-ggplot2=3.2*' \
    'r-irkernel=1.0*' \
    'r-rcurl=1.95*' \
    'r-sparklyr=1.0*' \
    && \
    conda clean --all -f -y && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Apache Toree kernel
RUN pip install --no-cache-dir \
    https://dist.apache.org/repos/dist/release/incubator/toree/0.3.0-incubating/toree-pip/toree-0.3.0.tar.gz \
    && \
    jupyter toree install --sys-prefix && \
    rm -rf /home/$NB_USER/.local && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Spylon-kernel
RUN conda install --quiet --yes 'spylon-kernel=0.4*' && \
    conda clean --all -f -y && \
    python -m spylon_kernel install --sys-prefix && \
    rm -rf /home/$NB_USER/.local && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER
    
# Install miniconda, iampl and other useful python modules
RUN cd /tmp && wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh \
    && chmod +x Miniconda3-latest-Linux-x86_64.sh \
    && bash Miniconda3-latest-Linux-x86_64.sh -b -p /opt/miniconda \
    && rm Miniconda3-latest-Linux-x86_64.sh \
    && /opt/miniconda/bin/conda install python=3.6 jupyter ipython matplotlib pandas -y \
    && /opt/miniconda/bin/conda clean --all -y \
    && /opt/miniconda/bin/pip install amplpy

# AMPL location (default: freely available demo version)
ARG AMPL=http://ampl.com/demo/ampl.linux64.tgz

# If $AMPL is a local compressed file, it will be automatically
# uncompressed into /opt by the ADD command.
# The script find_ampl.sh tries to ensure that /opt/ampl
# will point to a directory containing the ampl binaries.

USER root

ADD ${AMPL} /opt
RUN chmod 777 /opt/ampl.linux64.tgz
COPY scripts /opt/scripts
RUN /opt/scripts/find_ampl.sh 

# Add ampl and conda/python to the path
ENV PATH /opt/miniconda/bin:/opt/ampl:$PATH
ENV LANG C.UTF-8

USER $NB_UID

# ipython/jupyter configurations
EXPOSE 8888

WORKDIR /root
#CMD ["/opt/scripts/start_notebook.sh"]
