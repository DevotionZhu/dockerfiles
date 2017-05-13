FROM ubuntu:16.04

MAINTAINER Lloyd Hughes <hughes.lloyd@gmail.com>

RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

#############################################################################
#Install miniconda
#############################################################################
RUN apt-get update --fix-missing && apt-get install -y wget bzip2 ca-certificates \
    libglib2.0-0 libxext6 libsm6 libxrender1 \
    git mercurial subversion

RUN echo 'export PATH=/opt/conda/bin:$PATH' > /etc/profile.d/conda.sh && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh

ENV PATH /opt/conda/bin:$PATH

##############################################################################
# Install tini
##############################################################################
RUN apt-get install -y curl grep sed dpkg && \
    TINI_VERSION=`curl https://github.com/krallin/tini/releases/latest | grep -o "/v.*\"" | sed 's:^..\(.*\).$:\1:'` && \
    curl -L "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini_${TINI_VERSION}.deb" > tini.deb && \
    dpkg -i tini.deb && \
    rm tini.deb && \
    apt-get clean

ENTRYPOINT [ "/usr/bin/tini", "--" ]

##############################################################################
# Patch and upgrade conda, pip and apt-get
##############################################################################
# Temporary patch to fix build issues https://github.com/conda/conda/issues/4309
RUN conda config --add channels conda-canary
RUN conda update conda -y --quiet

RUN pip install --upgrade pip
RUN apt-get update

##############################################################################
# Install Python libaries
##############################################################################
RUN conda install --name root ipykernel ipython numpy scipy matplotlib pandas scikit-learn scikit-image jupyter -y --quiet

##############################################################################
# Fix matplotlib dependancy issues (temporary patch)
##############################################################################
RUN conda install --name root pyqt=4.11 -y --quiet

##############################################################################
# Configure Jupyter
##############################################################################
RUN python -m ipykernel.kernelspec

ENV PASSWORD=dlTUM
COPY jupyter_notebook_config.py /root/.jupyter/
COPY run_jupyter.sh /root/

##############################################################################
# Install dependancies for Tensorflow and Caffe
##############################################################################
RUN apt-get update && apt-get upgrade -y --quiet && apt-get install --fix-missing -y\
		apt-utils \
        build-essential \
        cmake \
    	pkg-config \
    	python3 \
		python3-dev \
		python3-setuptools \
		python3-pip \
    	libprotobuf-dev \
        libleveldb-dev \
        libsnappy-dev \
        libhdf5-serial-dev \
        protobuf-compiler \
        libblas-dev \
		liblapack-dev\
        libatlas-base-dev \
        libgflags-dev \
        libgoogle-glog-dev \
        liblmdb-dev \
        libopencv-dev \
        gfortran

RUN apt-get install -y --no-install-recommends libboost-all-dev
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

##############################################################################
# Install Caffe
##############################################################################
RUN conda install --name root  -c menpo opencv3 -y --quiet

ENV CAFFE_ROOT=/opt/caffe
WORKDIR $CAFFE_ROOT

RUN git clone https://github.com/BVLC/caffe.git . && \
    cd python && for req in $(cat requirements.txt) pydot; do pip install $req; done && \
    pip install --upgrade python-dateutil && \
    cd .. && \
    mkdir build && cd build && \
    cmake -DCPU_ONLY=1 \
		-DBLAS=atlas \
    	-Dpython_version=3.5 .. && \
    make -j"$(nproc)" all && \
    make install && \
    make pycaffe && \
    make runtest

ENV PYCAFFE_ROOT $CAFFE_ROOT/python
ENV PYTHONPATH $PYCAFFE_ROOT:$PYTHONPATH
ENV PATH $CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig

RUN conda install --name root cython h5py nose pyyaml protobuf -y --quiet

##############################################################################
# Install Tensorflow
##############################################################################
RUN conda install -c conda-forge tensorflow
ENV TENSORFLOW_VERSION 0.12.1
RUN pip install --ignore-installed --upgrade\
    	https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.12.1-cp35-cp35m-linux_x86_64.whl
    	
RUN tensorboard --logdir ./ --host 0.0.0.0

##############################################################################
# Install other tools
##############################################################################
# Tools for sanity
RUN apt-get update && apt-get install -y --quiet vim
# RUN conda install protobuf

##############################################################################
# Cleanup
##############################################################################
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Tensorboard Jupyter
EXPOSE 6006 8888

WORKDIR "/notebooks"

CMD ["/root/run_jupyter.sh"]
