**Add repository in ubuntu**

    sudo add-apt-repository ppa:vbernat/haproxy-2.6 -y

**Install HAproxy**

        sudo apt update
        sudo apt install -y haproxy=2.6.\*

**Check HAproxy**

        haproxy -v

        systemctl status haproxy

        systemctl enable haproxy

**Configure HAProxy**

        sudo nano /etc/haproxy/haproxy.cfg


_**For K8S**_

        global
          maxconn 100000
          nbthread 10
          cpu-map auto:1/1-10 0-9
          log /dev/log    local0 debug
          log /dev/log    local1 notice
          chroot /var/lib/haproxy
          stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
          stats timeout 30s
          user haproxy
          group haproxy
          daemon
          # Default SSL material locations
          ca-base /etc/ssl/certs
          crt-base /etc/ssl/private
        
          # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
          ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
          ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
          ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets
        
        defaults
          log     global
          mode    http
          option  httplog
          option log-separate-errors
          option log-health-checks
          option  dontlognull 
          option  forwardfor
          option  http-server-close
          timeout connect 5000
          timeout client  50000
          timeout server  50000
          errorfile 400 /etc/haproxy/errors/400.http
          errorfile 403 /etc/haproxy/errors/403.http
          errorfile 408 /etc/haproxy/errors/408.http
          errorfile 500 /etc/haproxy/errors/500.http
          errorfile 502 /etc/haproxy/errors/502.http
          errorfile 503 /etc/haproxy/errors/503.http
          errorfile 504 /etc/haproxy/errors/504.http
        
        #------------------------------------------------------
        listen kubernetes-apiserver-https
          bind *:6443
          mode tcp
          timeout client 3h
          timeout server 3h
          server master-node1 10.100.100.10:6443 check check-ssl verify none inter 10000
          server master-node2 10.100.100.20:6443 check check-ssl verify none inter 10000
          server master-node3 10.100.100.30:6443 check check-ssl verify none inter 10000
          balance roundrobin
        
        frontend http_frontend
          mode tcp
          maxconn 100000
          option forwardfor
          bind *:80
          http-request redirect scheme https unless { ssl_fc }
          http-request set-header X-Forwarded-For %[src]
          default_backend http_backend
        
        frontend https_frontend
          mode tcp
          maxconn 100000
          option forwardfor
          bind *:443
          http-request set-header X-Forwarded-Port %[dst_port]
          http-request add-header X-Forwarded-Proto https if { ssl_fc }  
          http-request set-header X-Forwarded-For %[src]
          default_backend https_backend
        
        backend http_backend
          mode http
          balance roundrobin
          server worker-node01 10.100.100.1:80 check port 80
          server worker-node02 10.100.100.2:80 check port 80
          server worker-node03 10.100.100.3:80 check port 80
        
        backend https_backend
          mode http
          balance roundrobin
          server worker-node01 10.100.100.1:443 check port 443
          server worker-node02 10.100.100.2:443 check port 443
          server worker-node03 10.100.100.3:443 check port 443
  
Show HAproxy logs

        nano -f /var/log/haproxy.log
