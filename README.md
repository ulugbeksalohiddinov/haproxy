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
--------------------------------------------------------------------------------------
**Example:**

    172.26.45.54 - HAProxy
    172.26.45.50 - Master
    172.26.45.51 - Worker1
    172.26.45.52 - Worker2
    172.26.45.53 - Worker3

**1.HAProxy o'rnatish (Ubuntu)**

    sudo add-apt-repository ppa:vbernat/haproxy-2.8 -y
    sudo apt update
    sudo apt install -y haproxy=2.8.\*

Agar versiya yangilanga bo'lsa 2.8 dan yangi versiyani qo'yish kerak.

**2.Tekshirish**

    haproxy -v
    sudo systemctl status haproxy
    sudo systemctl enable haproxy

**3. .cert va .key fayillardan .pem fayl yasash**

    sudo bash -c 'cat "/home/user/mkb_uz ssl/mkb_uz_all.crt" > /etc/haproxy/certs/wildcard.pem'
    sudo bash -c 'echo "" >> /etc/haproxy/certs/wildcard.pem'
    sudo bash -c 'cat "/home/user/mkb_uz ssl/mkb_uz.key" >> /etc/haproxy/certs/wildcard.pem'
    sudo bash -c 'echo "" >> /etc/haproxy/certs/wildcard.pem'

    sudo chmod 600 /etc/haproxy/certs/wildcard.pem

**Tekshirish**

 Bloklarni sanash (4 yoki 5 bo'lishi kerak)
        
        grep -c "BEGIN" /etc/haproxy/certs/wildcard.pem

 Oxirgi qatorlar
        
        tail -3 /etc/haproxy/certs/wildcard.pem

**HAProxy config tekshirish**    

    sudo haproxy -c -f /etc/haproxy/haproxy.cfg

**4. Configuratsiyani to'g'irlash /etc/haproxy/haproxy.cfg**

1 ta master va 3 ta workerliy cluster. SSL haproyxda tekshiriladi. Backendlar 80 portda eshitadi.


    global
        log /dev/log local0
        log /dev/log local1 notice
        maxconn 50000
        user haproxy
        group haproxy
        daemon
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

    defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        option  forwardfor
        option  http-server-close
        timeout connect 5s
        timeout client  50s
        timeout server  50s
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

    #------------------------------------------------------
    # Kubernetes API Server
    #------------------------------------------------------
    listen kubernetes-apiserver-https
        bind *:6443
        mode tcp
        option tcplog
        timeout client 3h
        timeout server 3h
        balance roundrobin
        server master-node1 172.26.45.50:6443 check check-ssl verify none inter 10000

    #------------------------------------------------------
    # Frontend HTTP -> HTTPS redirect
    #------------------------------------------------------
    frontend http_frontend
        bind *:80
        mode http
        maxconn 100000
        http-request set-header X-Forwarded-For %[src]
        http-request redirect scheme https code 301 unless { ssl_fc }

    #------------------------------------------------------
    # Frontend HTTPS - SSL Termination
    #------------------------------------------------------
    frontend https_frontend
        bind *:443 ssl crt /etc/haproxy/certs/wildcard.pem alpn h2,http/1.1
        mode http
        maxconn 100000
        http-request set-header X-Forwarded-Port %[dst_port]
        http-request set-header X-Forwarded-For %[src]
        http-request add-header X-Forwarded-Proto https
        default_backend workers_backend

    #------------------------------------------------------
    # Backend Workers
    #------------------------------------------------------
    backend workers_backend
        mode http
        balance roundrobin
        option httpchk GET /
        http-check expect status 200,301,302,404
        server worker-node01 172.26.45.51:80 check port 80
        server worker-node02 172.26.45.52:80 check port 80
        server worker-node03 172.26.45.53:80 check port 80


**5.Restart**

    sudo haproxy -c -f /etc/haproxy/haproxy.cfg
    sudo systemctl restart haproxy
    sudo systemctl status haproxy

**6. Domen misol nginx.uz Bu domenni haproxy IPsiga qaratib qo'yish kerak.**

 HAProxy loglarini ko'rish

 1. Real-time log (eng qulay)

        sudo tail -f /var/log/haproxy.log


----
Kubectl bilan klient haproxy orqali ulanmoqchi bo'lsa 6443 portiga. Masterda Config.yaml ga SAN qo'shish
kerak.

    sudo nano /etc/rancher/rke2/config.yaml # Agar config file bo'lmasa yaratish kerak config.yaml

    tls-san:
      - 172.23.0.50
      - 172.23.0.54
      - 127.0.0.1

    sudo systemctl restart rke2-server #Tayyor bo'lishini kuting:

    sudo systemctl status rke2-server
kubectl get nodes

