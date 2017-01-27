FROM continuumio/miniconda3

MAINTAINER Lloyd Hughes <hughes.lloyd@gmail.com>

# Temporary patch to fix build issues https://github.com/conda/conda/issues/4309
RUN conda config --add channels conda-canary
RUN conda update conda

RUN pip install --upgrade pip

RUN conda install --name root ipykernel ipython numpy scipy matplotlib pandas scikit-learn scikit-image jupyter -y --quiet
RUN conda install -c conda-forge tensorflow -y --quiet

# Temporary patch to fix matplotlib
RUN conda install pyqt=4.11 -y --quiet

RUN python -m ipykernel.kernelspec

ENV PASSWORD=dlTUM

COPY jupyter_notebook_config.py /root/.jupyter/
COPY run_jupyter.sh /root/

#RUN apt-get update && apt-get install -y \
#    libblas-dev \
#    liblapack-dev\
#    libatlas-base-dev \
#    gfortran \
#    && \
#    apt-get clean && \
#    rm -rf /var/lib/apt/lists/*

#ENV TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.11.0-cp35-cp35m-linux_x86_64.whl
#RUN pip install --upgrade -I setuptools
#RUN pip install --upgrade $TF_BINARY_URL

# Tools for sanity
RUN apt-get update && apt-get install -y vim

# Tensorboard Jupyter
EXPOSE 6006 8888

WORKDIR "/notebooks"

CMD ["/root/run_jupyter.sh"]
