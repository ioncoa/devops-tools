# Install Exporters ( Debian )

## Create Installing script
    sudo su
    vim /opt/exporters.sh

    #################Add following script to sh file#######################
    sudo apt install git golang -y
    cd /opt
    git clone https://github.com/oliver006/redis_exporter.git
    cd redis_exporter
    go build .

    cd /opt
    wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
    tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
    mv node_exporter-1.7.0.linux-amd64 node_exporter && cd node_exporter

    groupadd -f node_exporter
    useradd -g node_exporter --no-create-home --shell /bin/false node_exporter
    groupadd -f redis_exporter
    useradd -g redis_exporter --no-create-home --shell /bin/false redis_exporter

    cat <<EOF >>/etc/systemd/system/node_exporter.service
    [Unit]
    Description=Node Exporter
    Documentation=https://prometheus.io/docs/guides/node-exporter/
    Wants=network-online.target
    After=network-online.target

    [Service]
    User=node_exporter
    Group=node_exporter
    Type=simple
    Restart=on-failure
    ExecStart=/opt/node_exporter/node_exporter \
    --web.listen-address=:9100

    [Install]
    WantedBy=multi-user.target
    EOF

    cat <<EOF >>/etc/systemd/system/redis_exporter.service
    [Unit]
    Description=Redis Exporter
    Wants=network-online.target
    After=network-online.target

    [Service]
    User=redis_exporter
    Group=redis_exporter
    Type=simple
    Restart=on-failure
    ExecStart=/opt/redis_exporter/redis_exporter \
    --web.listen-address=:9121 \
    --redis.password="xxxxxx"

    [Install]
    WantedBy=multi-user.target
    EOF

    chmod 664 /etc/systemd/system/node_exporter.service
    chmod 664 /etc/systemd/system/redis_exporter.service
    systemctl daemon-reload
    systemctl enable node_exporter
    systemctl enable redis_exporter
    systemctl start node_exporter
    systemctl start redis_exporter
    ####################END##############################


## Run the script
    chmod 664 /opt/exporters.sh
    bash -xe /opt/exporters.sh

## Check the system
    systemctl status node_exporter
    systemctl status redis_exporter