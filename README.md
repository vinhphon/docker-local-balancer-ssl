
# Convenient and automated way to manage local development that require SSL/TLS and hostname resolution.

This stack is a local development setup using Docker Compose, which includes a DNS server (dnsmasq) for hostname resolution and a reverse proxy server (traefik or nginx) for load balancing and SSL/TLS termination. The SSL/TLS certificates are generated using mkcert and are signed by a rootCA certfile, which is installed on the reverse proxy container.

The hostname and certificate configuration for the reverse proxy server is dynamically generated using dockegen, which listens to container environment configurations to automatically generate the required configuration files. This makes it easy to add or remove containers and have their hostnames automatically resolved and SSL/TLS certificates generated and configured on the reverse proxy server.

## Features:

- Two reverse proxies (traefik/nginx)
- Automatic DNS hostname resolution for containers
- Automatic SSL configuration using a root CA certificate
- A private network that avoids port conflicts
- Container images that support both ARM64 and AMD64 platforms.

## Setup

1. Start the stack

    - Git clone/download the project, then go to type of reverse proxy folder you want to use (nginx/traefik)

    - Run it with compose command

        ```
        docker-compose up -d
        ```
    - MACOS, we cannot using private network, that you need to map port 80, 443 to host machine

        ```
        docker-compose -f docker-compose.yml -f compose.public.yml up -d
        ```
2. Setup valid HTTPS

    - MACOS: Add the rootCA.pem from folder rootCA to Keychain, allow trust all the key.
    
    - Linux:

        - Chrome/Edge: Go to Setting, search "Manage certificates", navigate to tab "AUTHORITIES" then import rootCA.pem, trust all and save.
        - Firefox: Go to Setting -> Search "View Certificates", click to open, navigate to Authories tab then import rootCA.pem and save

3. Setup auto dns

    - MacOS: Go to current network setting (Wifi or VPN or Ethernet) -> Change DNS to 127.0.0.1, ensue it is top priority.

    - Linux: Depend on your dns services that you are using, but the common that you need to add the dns server 10.10.10.53 to /etc/resolv.conf
        - In my case, using Network Manager as main tool, that I can edit DNS from UI like MacOs
        - If you are using systemd-resolver or dnsmasq as dns services, you need do some adjustment to make 10.10.10.53 as top priority dns server.

## Usage

Please ref docker-compose.nginx.sample.yml, docker-compose.traefik.sample.yml for example. But the common usage that you need pay atention is

 - Nginx:

    ```yml
        environment:
            - VIRTUAL_HOST=[your hostnames, seperated by comma]
            - VIRTUAL_PORT=[your expose port if different 80]
        networks:
            - default
            - net.autolbssl-stack
    networks:
        net.autolbssl-stack:
            external: true
    ```
- Traefik:

    ```yml
        labels:
            - traefik.enable=true
            - traefik.http.routers.[yourapp].tls=true
            - traefik.http.routers.[yourapp].rule=Host([your hostname swrap by ``, seperated by comma])
            - traefik.http.services.whoami.loadbalancer.server.port=[your expose port if different 80]
        networks:
            - default
            - net.autolbssl-stack
    networks:
        net.autolbssl-stack:
            external: true
    ```

## Limitation

- We cannot using .local or .dev as development domain


## Contributing

Contributions are always welcome! If you'd like to contribute, please open an issue or pull request on the GitHub repository.

## License

This project is licensed under the MIT License.