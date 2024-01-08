FROM mcr.microsoft.com/devcontainers/universal

RUN sudo apt update

# Install Pulumi
RUN curl -fsSL https://get.pulumi.com/releases/sdk/pulumi-v3.100.0-linux-x64.tar.gz  -o /tmp/pulumi.tar.gz
RUN mkdir -p /opt/pulumi
RUN tar zxf /tmp/pulumi.tar.gz -C /opt
RUN chmod +x /opt/pulumi/pulumi
RUN ln -s /opt/pulumi/pulumi /usr/bin/pulumi

# Install LocalStack and AWS CLI (awslocal) 
RUN pip install localstack awscli-local[ver1] awscli
