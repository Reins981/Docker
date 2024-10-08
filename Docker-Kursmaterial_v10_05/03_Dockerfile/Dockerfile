ARG PYTHON_VERSION=latest
  
# Choose and name our temporary image.
FROM alpine as intermediate
# Add metadata identifying these images as our build containers (this will be useful later!)
LABEL stage=intermediate
  
# Take an SSH key as a build argument.
ARG SSH_KEY
  
# Install dependencies required to git clone.
RUN apk update && \
    apk add --update git && \
    apk add --update openssh
  
# 1. Create the SSH directory.
# 2. Populate the private key file.
# 3. Set the required permissions.
# 4. Add github to our list of known hosts for ssh.
RUN mkdir -p /root/.ssh/ && \
    echo "$SSH_KEY" > /root/.ssh/id_rsa && \
    chmod -R 600 /root/.ssh/ && \
    ssh-keyscan -t rsa git.elektrobitautomotive.com >> ~/.ssh/known_hosts
  
# Clone a repository (my website in this case)
RUN git clone --recursive git@git.elektrobitautomotive.com:InnerSource/eb-ita-testrunner.git
  
  
FROM python:${PYTHON_VERSION}
  
COPY --from=intermediate /eb-ita-testrunner /eb-ita-testrunner
  
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir pytest pytest-html pytest-cov psutil mock lxml bitstring serpent pyhamcrest==2.0.4 ifaddr getmac Pyro5 pyserial paramiko jinja2 libcst
  
RUN apt update && \
    apt install -y libpcap0.8 nano iproute2 net-tools iputils-ping
  
RUN mkdir /tmp/cif_go && \
    wget -qO- http://denue6pr290.ebgroup.elektrobit.com:8080/job/download_cif_golang_linux/lastSuccessfulBuild/artifact/conan_package.tgz | tar -xvz -C /tmp/cif_go && \
    cp /tmp/cif_go/include/* /eb-ita-testrunner/ebita_libs/cif_go/so && \
    cp /tmp/cif_go/lib/* /eb-ita-testrunner/ebita_libs/cif_go/so && \
    rm -R /tmp/cif_go
  
RUN wget -q -O /tmp.zip http://denue6pr290.ebgroup.elektrobit.com:8080/job/build_cif_golang/job/master/lastSuccessfulBuild/artifact/cif_swig_64.zip && \
    unzip /tmp.zip -d /eb-ita-testrunner/ebita_libs/cif_go/ && \
    rm /tmp.zip
  
RUN echo /eb-ita-testrunner/ebita_libs/cif_go/so > /etc/ld.so.conf.d/cif_go.conf && \
    ldconfig
  
CMD bash