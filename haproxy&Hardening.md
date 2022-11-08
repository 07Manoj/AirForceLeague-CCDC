**Getting started with HAproxy configuration**

The goal of this playbook and configuration is to solve for the following:

- Securing our webserver by limiting the direct access to the web server, we are using haproxy instead to re route the traffic to the server. We would be listening on port 80 and port 443 for http requests.
- We have seen that http sends username and passwords in the plaintext to the server which can be sniffed. To prevent this, we will be implementing https for which we would need a signed certificate from a certificate authority since https requires us to verify the server's identity which can be done by using a certificate.
- We will be signing our own certificate by creating a certifcate authority
- We saw earlier that we had access to the /admin page on the server which is never safe and hence we will be limiting the access to the /admin page using an access control list which is an additional measure haproxy offers.



```
defaults
timeout client 30s

global
    ssl-default-bind-options ssl-min-ver TLSv1.2

frontend fe1_http
    mode http
    bind *:80
    bind *:443 ssl crt /usr/local/etc/haproxy/airforce.ccdc.uml.edu.pem
    acl restricted_page path_beg /admin
    http-request redirect scheme https unless { ssl_fc }
    use_backend be1_http unless restricted_page


backend be1_http
    mode http
    balance roundrobin
    server website1 django-site:8000
    server website2 django-site1:8001

  ```

*Frontend:*  This is where haproxy listens on, binds to ports, does access control etc
*Backend* We will be defining our servers here and the haproxy listens on the frontend and redirects it to the backend.

**Mode:** This defines the mode of traffic that the proxy listens for and we are listening for http traffic since we are going to get http requests to our server.

**bind:** This defines the ports haporxy will be listening on. We are listening on ports 80 and 443 which are default ports for http and https. We can specify a ip address in the place of "*" and the haproxy will listen for the traffic to that particular ip on both the ports. We are listening for traffic coming to any ip on the above stated ports.

**bind *:443**  We have defined a Secure Socket Layer certificate that has been signed before. When using https, the communication is encrypted and hence you would need a certificate which would verify that it is really airforce.ccdc.uml.edu that we are using and not a insecure website.  We are esentially verifying our server's identity by giving it a certificate.

**Note:** Since this is a self - signed certificate, you will have to explicitly make your browser trustt the certificate by going into the browser settings. If this was signed by a certified certificate authority, you will no longer need to explicitly change the certificate trust settings 

*use_backend:* This specifies which backend to use and redirect the http requests to


 **Note:**  In the [Containers Playbook](containers_playbook.md) we have seen that we have deployed two django containers. The underlying reason for doing this is to solve for high availability and load balancing. We are using haproxy which is a load balancer and a proxy.  We are using a round robin algorithm which would balance between the two backend servers that we defined. This solves for high availability and we have two servers which will serve http requests instead of a single server.


*balance: roundrobin*  When a http request comes in, the haproxy is going to load balance the incoming requests between the two servers.

*server:*  We are specifying the name (which can be anything)  of the server and the django containers that we have created. 